---
title: React的更新流程（二）
date: 2018-02-27 10:54:39
tags:
  - react
  - react-native
categories:
  - react
---
上节讲述了React的更新流程，更新流程是以完成所有dirtyComponents的更新结束的。        
dirtyComponent是internalInstance，任意组件的挂载、更新、卸载都是通过ReactReconciler统一调用的。
<!-- more -->
*******     
## ReactReconciler：组件处理中心：挂载、更新、卸载       

* mountComponent，挂载组件，并将ref处理加入callbackqueue
* unmountComponent，将ref清空，卸载组件
* receiveComponent，使用nextElement执行更新
* performUpdateIfNecessary，对dirtyComponent更新
    1. 若存在_pendingElement，走receiveComponent流程
    2. 反之，直接进入internalInstance的updateComponent
* getHostNode，在RN中一个原生组件有rootID与真实的Native View关联。任意组件的hostNode为其树结构中最顶级原生组件的rootID

*******     

## updateComponent: internalInstance的更新

1. ReactCompositeComponent，执行相关生命周期，并对其renderedComponent继续更新
2. ReactNativeBaseComponent，无生命周期，直接通过UIManager更新Native UI，并对rendedChildren继续更新

## shouldUpdateReactComponent：element diff   

比较同一处的prevElement和nextElement，是否存在更新的条件。若可以更新，在原有实例上更新；反之，卸载旧的，挂载新的

1. prevElement和nextElement都是空，可更新
2. prevElement和nextElement都是数字或字符串，可更新
3. prevElement和nextElement都是element类型，且element.type相同（类型相同），且key相同，可更新

*******     

## shouldComponentUpdate: component diff   

当在一个element上更新时，提供的自定义方法，判定是否需要更新

* 接受nextProps、nextState、nextContext三个参数
* PureComponent的shouldComponentUpdate浅比较nextProps、nextState
* 若forUpdate，则忽略shouldComponentUpdate的判断