---
title: react源码分析（前言）
date: 2018-02-12 11:12:24
categories:
- react
tags:
- react
- react-native
---

看了一段时间React源码，主要是React Native实现的Virtual Dom树渲染更新部分。React版本是15.2.1，React Native版本是0.30.0。
想着写成自己的第一个系列博客，梳理巩固所思所得，将来忘了也能有个快速回顾的资料。如果能对他人有所助益，是更吼的了。

以下是目录结构


## React中的类
  
ReactElement：节点类      
ReactComponent：外部组件类      
ReactCompositeComponent：内部组件类        
ReactNativeBaseComponent：RN中映射到Native的组件类     

## React中类实例之间的关系

instantiateReactComponent: 通过element实例成ReactCompositeComponent或ReactNativeBaseComponent       
生命周期：ReactComponent提供实现，ReactNativeBaseComponent管理调用      
virtual dom结构        
render与逐层渲染      


## React中的一些工具辅助类

Transaction：在执行目标方法前后执行配置好的方法    
CallbackQueue：管理回调队列     
PooledClass：为了优化实例内存而做的类拓展       
traverseAllChildren：遍历this.props.children     


## React的更新流程（一）

shouldUpdateReactComponent：element diff   
shouldComponentUpdate: component diff     
ReactUpdateQueue：setState等内部实现       
ReactDefaultBatchingStrategy：所有更新的入口与出口   
ReactNativeReconcileTransaction：处理更新后需执行的回调     
ReactUpdatesFlushTransaction：清空待更新组件


## React的更新流程（二）

batchedUpdates：保证callback在一次updatequeue中执行        
enqueueUpdate：压入一个需更新的internalInstance     
flushBatchedUpdates: 发起一次清空待更新组件      
例子：setState后取到的值是新值还是旧值      


## React的更新流程（三）
  
ReactReconciler：组件处理中心：挂载、更新、卸载       
ReactChildReconciler：对children中有需要的项执行ReactReconciler     
ReactMultiChild：拓展ReactNativeBaseComponent，作为中间层调用ReactChildReconciler，children diff更新策略



<!-- more -->

