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
自己的第一个系列博客，梳理巩固所思所得。如果能对你有所助益，是更吼的了。

以下是目录结构
<!-- more -->


## React中的类
  
ReactElement：节点类      
ReactComponent：外部组件类      
ReactCompositeComponent：内部组件类        
ReactNativeBaseComponent：RN中映射到Native的组件类     

## React的Virtual DOM——类实例间的关系

instantiateReactComponent: 通过element实例成ReactCompositeComponent或ReactNativeBaseComponent       
生命周期：ReactComponent提供实现，ReactNativeBaseComponent管理调用      
virtual dom结构        
render与逐层渲染      


## React中的一些辅助工具类

Transaction：在执行目标方法前后执行配置好的方法    
CallbackQueue：管理回调队列     
PooledClass：为了优化实例内存而做的类拓展       
traverseAllChildren：遍历this.props.children     


## React的更新流程（一）

ReactDefaultBatchingStrategy：更新的入口与出口 
ReactUpdates: 批处理更新      
ReactUpdateQueue：setState等内部实现   


## React的更新流程（二）

ReactReconciler：组件处理中心：挂载、更新、卸载       
shouldUpdateReactComponent：element diff   
shouldComponentUpdate: component diff      

## React的更新流程（三）
  
ReactChildReconciler：对children中有需要的项执行ReactReconciler     
ReactMultiChild：拓展ReactNativeBaseComponent，作为中间层调用ReactChildReconciler，children diff更新策略


