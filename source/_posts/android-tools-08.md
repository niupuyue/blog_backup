---
title: android奇技淫巧 08 Android 焦点问题
date: 2019-07-07 22:27:10
tags:
  - android
---

前段时间在做公司的项目的时候，有一个布局文件的焦点一直被另外一个空间所获取到，在开发的过程中要做很多操作。虽然最后解决了，但是问题总是不太理想，所以，趁着这个时间把android关于焦点的问题复习一下。
<!--more-->
# 获取焦点的前提

1. view.isFocuseable返回true，或者在触摸模式中view.isfocusableInTouchMode也要返回true
2. 控件必须可见
3. 控件相关的父控件，包括祖父控件等，viewGroup.getDescendantFocusability()不能使ViewGroup.FOCUS_BLOCK_DESCENDANTS

# View 

## 获取焦点

调用view.requestFocus()系列方法

## 进入view.requestFocusNoSearch

在该方法中会对控件的当前状态进行判断, 如果不符合获取焦点的前提则直接返回false告知调用方, 控件不会获取焦点

只要符合前提就会继续执行, 最终必定返回true, 不论当前控件的焦点状态是否有改变

## 符合前提则进入 View#handleFocusGainInternal

如果控件已经持有焦点, 则不会做任何事情, 直接结束流程

如果没有焦点,

- 改变焦点标志位, 此时View#isFocused就会返回true了

- 通过ViewParent#requestChildFocus通知父控件即将获取焦点

- 通知其他部件焦点状态发生变化(略, 本文不关心)

- 触发OnGlobalFocusChangeListener的回调

- 触发OnFocusChangeListener回调

- 重绘, 结束流程

## 清除焦点

调用View#clearFocus主动放弃焦点

如果控件本身没有焦点, 则什么都不会发生

如果控件持有焦点

- 改变焦点标志位

- 通过ViewParent#clearChildFocus通知父控件, 当前控件放弃焦点

- 触发OnFocusChangeListener回调

- 调用当前控件的根控件(rootView)的requestFocus方法

- 如果步骤4中没有找到新的焦点控件, 则触发OnGlobalFocusChangeListener的回调, 注: 如果找到新的焦点控件, 那么新的控件获取焦点的过程中就会回调OnGlobalFocusChangeListener, 所以这里只有没找到才进行步骤5

注: 由上流程可以知道, 如果根控件查找控件的时候找到的控件还是这个控件, 那么OnFocusChangeListener就会被调用两次, 先失去焦点, 然后又获取到焦点

# ViewGroup

## 焦点分发策略 DescendantFocusability

- FOCUS_BLOCK_DESCENDANTS: 拦截焦点, 直接自己尝试获取焦点

- FOCUS_BEFORE_DESCENDANTS: 首先自己尝试获取焦点, 如果自己不能获取焦点, 则尝试让子控件获取焦点

- FOCUS_AFTER_DESCENDANTS: 首先尝试把焦点给子控件, 如果所有子控件都不要, 则自己尝试获取焦点

## 获取焦点

根据焦点分发策略决定下面两个方法的调用顺序

## 通过view#requestFocus()获取焦点

把ViewGroup看作View, 直接走View获取焦点的流程来获取焦点

## 进入 onRequestFocusInDescendants

可以传入方向来改变遍历的顺序, 默认是从0递增

遍历子控件, 调用子控件的View#requestFocus来尝试把焦点给可见的子控件, 某个子控件成功获取到焦点后, 停止遍历

注: 重写该方法可以改变ViewGroup分发焦点给子控件的行为, 例如遍历顺序

## 清除焦点

如果焦点控件不是它的子控件, 那么直接把当前的ViewGroup看作View走View#clearFocus流程, 反之则调用焦点控件的View#clearFocus.

注: 区别在于重新分发焦点时的选择范围.

## viewParent

ViewParent是一个接口, 表示了一个父控件应该具备的功能, ViewGroup实现了该接口.

与焦点相关的接口有4个

## clearChildFocus

当子控件主动放弃焦点的时候会通过这个方法通知父控件.

在ViewGroup的默认实现中, 会置空当前焦点控件, 表示该父控件下没有子控件获取焦点, 接着把这个事件通知给上级父控件.

注1: 这个方法名有点让人误解, 应该把这个方法看作一个回调, 表明了一个状态, 在这个方法中并没有做清除焦点的操作, 实际的清除动作是在View#clearFocus中完成的, 这个方法也是在这个流程中被调用的. 而且是在子控件已经放弃焦点后调用.
注2: 区分主动放弃和因为其他控件获取了焦点而被动丢失焦点的情况

## requestChildFocus

当子控件获取了焦点后, 通过这个方法通知父控件. 同clearChildFocus类似, 应该把这个方法看作是一个回调.

在ViewGroup的默认实现中, 因为同时只会有一个焦点, 因此在这里应该把旧焦点清除掉, 大致流程如下

如果焦点分发策略为FOCUS_BLOCK_DESCENDANTS则什么也不干

如果父控件自身有焦点, 通过View#unFocus清除焦点

如果父控件当前已经有焦点控件, 并且和新的控件不一致, 那么通过View#unFocus清除旧焦点控件的焦点

向上传递这个事件

## 内部清除焦点 View#unFocus

这个方法和View#clearFocus相同点在于都会执行View#clearFocusInternal方法, 区别在于unFocus只会执行clearFocus中, 上文清除焦点中提到的1, 3步骤, 因此不会通知父控件, 不会触犯requestChildFocus回调, 因为这个方法是在子控件被动失去焦点时调用的, 所以也不会触发焦点分发.

因此新旧焦点切换的大致流程是

- 新焦点控件获取焦点

- 新焦点控件通知父控件

- 父控件清除旧焦点控件的焦点

- 旧焦点控件回调OnFocusChangeListener

- 触发OnGlobalFocusChangeListener的回调

- 新焦点控件回调OnFocusChangeListener

## focusableViewAvailable

通知父控件, 子控件的状态发生改变, 从不能获取焦点, 变成可能可以获取焦点.

有两种情况会被调用

- 子控件从unFocusable变为focusable

- 子控件从不可见变为可见, 即使它不是focusable也会调用, 因此它的子控件可能可以获取焦点.

而ViewGroup中的默认实现只是在符合条件的情况下把这个事件向上传递给自己的父控件.

- focusSearch(View, int)

查找指定方向中最近的, 想要获取焦点的控件.

这个方法直接决定了焦点的移动规则, 非常重要.

在ViewGroup的默认实现中, 会一直向上传递, 直到根控件, 接着调用FocusFinder#findNextFocus方法查找合适的控件. 稍后再分析这个方法.

> View中有一个同名的方法focusSearch(int), 该方法直接调用了父控件的focusSearch(View, int)来查找下一个焦点控件

## findNextFocus

查找步骤大致如下

### 手动指定

如果有通过android:nextFocusDown等手动指定控件, 则返回对应方向的控件

### 动态计算

- 获取所有可以获取焦点的控件的集合

- 计算相对当前焦点控件的坐标

- 根据方向选择合适的控件


