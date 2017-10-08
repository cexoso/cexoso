---
title: react-motion源码阅读
date: 2017-05-21 20:19:02
tags: react-motion react动画
reward: true
---

> 一直对动画有兴趣，可惜自己想要的效果现在没有办法实现。只能多学习相关的库了。这也是读react-motion的初衷。

# 概念 spring（弹簧）
在motion中，会给定一个初始值和一个终止值。motion会按桢计算出两个值之间的过滤。并且每个桢都会调用setState来触发render。而决定初始值到终止值之前是怎么过滤的就取决于`spring`


```javascript
import presets from './presets';

const defaultConfig = {
  ...presets.noWobble,
  precision: 0.01,
};

export default function spring(val: number, config) {
  return {...defaultConfig, ...config, val};
}

```
一个`spring`最终是由`val`(终止值)、`stiffness`(刚度)、`damping`(阻尼)、和 `precision`(精度)来表示。

想像一根弹簧`val`就是弹簧的原始长度。`stiffness`的在高中物理中叫`k`(劲度系数)，有一个公式`F = k * deltaS`，`deltaS`是弹簧形变的长度。`damping`阻尼被定义为速度与力的一个系数。在motion中被定义中`F = b * V`其中V为速度,b为阻尼系数,F为力。

motion中有4组预定义的系数

```javascript
export default {
  noWobble: {stiffness: 170, damping: 26}, // the default, if nothing provided
  gentle: {stiffness: 120, damping: 14},
  wobbly: {stiffness: 180, damping: 12},
  stiff: {stiffness: 210, damping: 20},
};
```
分别是不摇晃的,温和的，摇晃的和僵硬的。其实的数值应该是根据经验试出来的。(在motion中所有的公式都是定性不定量的，即没有米的概念也没有牛的概念)

# 驱动react渲染的motion.js
```javascript
/* @flow */
import mapToZero from './mapToZero';
import stripStyle from './stripStyle';
import stepper from './stepper';
import defaultNow from 'performance-now';
import defaultRaf from 'raf';
import shouldStopAnimation from './shouldStopAnimation';
import React from 'react';
import PropTypes from 'prop-types';

import type {ReactElement, PlainStyle, Style, Velocity, MotionProps} from './Types';

const msPerFrame = 1000 / 60;

type MotionState = {
  currentStyle: PlainStyle,
  currentVelocity: Velocity,
  lastIdealStyle: PlainStyle,
  lastIdealVelocity: Velocity,
};

export default class Motion extends React.Component<MotionProps, MotionState> {
  static propTypes = {
    // TOOD: warn against putting a config in here
    defaultStyle: PropTypes.objectOf(PropTypes.number),
    style: PropTypes.objectOf(PropTypes.oneOfType([
      PropTypes.number,
      PropTypes.object,
    ])).isRequired,
    children: PropTypes.func.isRequired,
    onRest: PropTypes.func,
  };

  constructor(props: MotionProps) {
    super(props);
    this.state = this.defaultState();
  }

  wasAnimating: boolean = false;
  animationID: ?number = null;
  prevTime: number = 0;
  accumulatedTime: number = 0;

  defaultState(): MotionState {
    const {defaultStyle, style} = this.props;
    const currentStyle = defaultStyle || stripStyle(style);
    const currentVelocity = mapToZero(currentStyle);
    return {
      currentStyle,
      currentVelocity,
      lastIdealStyle: currentStyle,
      lastIdealVelocity: currentVelocity,
    };
  }

  // it's possible that currentStyle's value is stale: if props is immediately
  // changed from 0 to 400 to spring(0) again, the async currentStyle is still
  // at 0 (didn't have time to tick and interpolate even once). If we naively
  // compare currentStyle with destVal it'll be 0 === 0 (no animation, stop).
  // In reality currentStyle should be 400
  unreadPropStyle: ?Style = null;
  // after checking for unreadPropStyle != null, we manually go set the
  // non-interpolating values (those that are a number, without a spring
  // config)
  clearUnreadPropStyle = (destStyle: Style): void => {
    let dirty = false;
    let {currentStyle, currentVelocity, lastIdealStyle, lastIdealVelocity} = this.state;

    for (let key in destStyle) {
      if (!Object.prototype.hasOwnProperty.call(destStyle, key)) {
        continue;
      }

      const styleValue = destStyle[key];
      if (typeof styleValue === 'number') {
        if (!dirty) {
          dirty = true;
          currentStyle = {...currentStyle};
          currentVelocity = {...currentVelocity};
          lastIdealStyle = {...lastIdealStyle};
          lastIdealVelocity = {...lastIdealVelocity};
        }

        currentStyle[key] = styleValue;
        currentVelocity[key] = 0;
        lastIdealStyle[key] = styleValue;
        lastIdealVelocity[key] = 0;
      }
    }

    if (dirty) {
      this.setState({currentStyle, currentVelocity, lastIdealStyle, lastIdealVelocity});
    }
  };

  startAnimationIfNecessary = (): void => {
    // TODO: when config is {a: 10} and dest is {a: 10} do we raf once and
    // call cb? No, otherwise accidental parent rerender causes cb trigger
    this.animationID = defaultRaf((timestamp) => {
      // check if we need to animate in the first place
      const propsStyle: Style = this.props.style;
      if (shouldStopAnimation(
        this.state.currentStyle,
        propsStyle,
        this.state.currentVelocity,
      )) {
        if (this.wasAnimating && this.props.onRest) {
          this.props.onRest();
        }

        // no need to cancel animationID here; shouldn't have any in flight
        this.animationID = null;
        this.wasAnimating = false;
        this.accumulatedTime = 0;
        return;
      }

      this.wasAnimating = true;

      const currentTime = timestamp || defaultNow();
      const timeDelta = currentTime - this.prevTime;
      this.prevTime = currentTime;
      this.accumulatedTime = this.accumulatedTime + timeDelta;
      // more than 10 frames? prolly switched browser tab. Restart
      if (this.accumulatedTime > msPerFrame * 10) {
        this.accumulatedTime = 0;
      }

      if (this.accumulatedTime === 0) {
        // no need to cancel animationID here; shouldn't have any in flight
        this.animationID = null;
        this.startAnimationIfNecessary();
        return;
      }

      let currentFrameCompletion =
        (this.accumulatedTime - Math.floor(this.accumulatedTime / msPerFrame) * msPerFrame) / msPerFrame;
      const framesToCatchUp = Math.floor(this.accumulatedTime / msPerFrame);

      let newLastIdealStyle: PlainStyle = {};
      let newLastIdealVelocity: Velocity = {};
      let newCurrentStyle: PlainStyle = {};
      let newCurrentVelocity: Velocity = {};

      for (let key in propsStyle) {
        if (!Object.prototype.hasOwnProperty.call(propsStyle, key)) {
          continue;
        }

        const styleValue = propsStyle[key];
        if (typeof styleValue === 'number') {
          newCurrentStyle[key] = styleValue;
          newCurrentVelocity[key] = 0;
          newLastIdealStyle[key] = styleValue;
          newLastIdealVelocity[key] = 0;
        } else {
          let newLastIdealStyleValue = this.state.lastIdealStyle[key];
          let newLastIdealVelocityValue = this.state.lastIdealVelocity[key];
          for (let i = 0; i < framesToCatchUp; i++) {
            [newLastIdealStyleValue, newLastIdealVelocityValue] = stepper(
              msPerFrame / 1000,
              newLastIdealStyleValue,
              newLastIdealVelocityValue,
              styleValue.val,
              styleValue.stiffness,
              styleValue.damping,
              styleValue.precision,
            );
          }
          const [nextIdealX, nextIdealV] = stepper(
            msPerFrame / 1000,
            newLastIdealStyleValue,
            newLastIdealVelocityValue,
            styleValue.val,
            styleValue.stiffness,
            styleValue.damping,
            styleValue.precision,
          );

          newCurrentStyle[key] =
            newLastIdealStyleValue +
            (nextIdealX - newLastIdealStyleValue) * currentFrameCompletion;
          newCurrentVelocity[key] =
            newLastIdealVelocityValue +
            (nextIdealV - newLastIdealVelocityValue) * currentFrameCompletion;
          newLastIdealStyle[key] = newLastIdealStyleValue;
          newLastIdealVelocity[key] = newLastIdealVelocityValue;
        }
      }

      this.animationID = null;
      // the amount we're looped over above
      this.accumulatedTime -= framesToCatchUp * msPerFrame;

      this.setState({
        currentStyle: newCurrentStyle,
        currentVelocity: newCurrentVelocity,
        lastIdealStyle: newLastIdealStyle,
        lastIdealVelocity: newLastIdealVelocity,
      });

      this.unreadPropStyle = null;

      this.startAnimationIfNecessary();
    });
  };

  componentDidMount() {
    this.prevTime = defaultNow();
    this.startAnimationIfNecessary();
  }

  componentWillReceiveProps(props: MotionProps) {
    if (this.unreadPropStyle != null) {
      // previous props haven't had the chance to be set yet; set them here
      this.clearUnreadPropStyle(this.unreadPropStyle);
    }

    this.unreadPropStyle = props.style;
    if (this.animationID == null) {
      this.prevTime = defaultNow();
      this.startAnimationIfNecessary();
    }
  }

  componentWillUnmount() {
    if (this.animationID != null) {
      defaultRaf.cancel(this.animationID);
      this.animationID = null;
    }
  }

  render(): ReactElement {
    const renderedChildren = this.props.children(this.state.currentStyle);
    return renderedChildren && React.Children.only(renderedChildren);
  }
}

```
motion是一个react组件。其中一个核心的函数就是`startAnimationIfNecessary`，其各种生命周期函数都是在围绕着这个函数的调用。

startAnimationIfNecessary每次都只是会计算一桢的值供render使用。其中使用了raf(一个`requestAnimationFrame`兼容库)，注意到
```javascript
const framesToCatchUp = Math.floor(this.accumulatedTime / msPerFrame);
if (this.accumulatedTime > msPerFrame * 10) {
    this.accumulatedTime = 0;
}

if (this.accumulatedTime === 0) {
// no need to cancel animationID here; shouldn't have any in flight
    this.animationID = null;
    this.startAnimationIfNecessary();
    return;
}
...
for (let i = 0; i < framesToCatchUp; i++) {
    [newLastIdealStyleValue, newLastIdealVelocityValue] = stepper(
        msPerFrame / 1000,
        newLastIdealStyleValue,
        newLastIdealVelocityValue,
        styleValue.val,
        styleValue.stiffness,
        styleValue.damping,
        styleValue.precision,
    );
}
...
this.accumulatedTime -= framesToCatchUp * msPerFrame
```
startAnimationIfNecessary每次都会消费掉新产生的时间片，除非超过10桢没响应。

motion中的state存着4个值。currentStyle当前样式，currentVelocity当前的速率。lastIdealStyle上一个样式,lastIdealVelocity上一个速度。

在startAnimationIfNecessary主要是获取时间来迭代计算state中的值。其中用到一个变量函数stepper。

#stepper.js

```javascript
/* @flow */

// stepper is used a lot. Saves allocation to return the same array wrapper.
// This is fine and danger-free against mutations because the callsite
// immediately destructures it and gets the numbers inside without passing the
// array reference around.
let reusedTuple: [number, number] = [0, 0];
export default function stepper(
  secondPerFrame: number,
  x: number,
  v: number,
  destX: number,
  k: number,
  b: number,
  precision: number): [number, number] {
  // Spring stiffness, in kg / s^2

  // for animations, destX is really spring length (spring at rest). initial
  // position is considered as the stretched/compressed position of a spring
  const Fspring = -k * (x - destX);

  // Damping, in kg / s
  const Fdamper = -b * v;

  // usually we put mass here, but for animation purposes, specifying mass is a
  // bit redundant. you could simply adjust k and b accordingly
  // let a = (Fspring + Fdamper) / mass;
  const a = Fspring + Fdamper;

  const newV = v + a * secondPerFrame;
  const newX = x + newV * secondPerFrame;

  if (Math.abs(newV) < precision && Math.abs(newX - destX) < precision) {
    reusedTuple[0] = destX;
    reusedTuple[1] = 0;
    return reusedTuple;
  }

  reusedTuple[0] = newX;
  reusedTuple[1] = newV;
  return reusedTuple;
}

```

Fspring由劲度系数产生的力，Fdamper由阻尼产生的力。a为加速度，此处直接定义为两个力的和（假定质量为1了呗）。接着用积分的方法(假定了时间无穷小，其实一个时间片为1/60秒),算出新的速度和新的位置。并且如果速度和位置差在精度内就将速度置为0来结束运动。最后返回带有新位置和速度的tuple。

*react-motion中还有其它的方法不在本次阅读范围内*