---
title: React的Virtual DOM——类实例间的关系
date: 2018-02-24 20:16:52
tags:
---
本节讲述React中的Virtual DOM是如何逐层解析并绘制为真实页面的。
<!-- more -->
*******
## instantiateReactComponent: 通过element实例成ReactCompositeComponent或ReactNativeBaseComponent        

1. instantiateReactComponent将节点实例化为具备挂载、更新、卸载能力的实例
2. 存在5种：
    1. ReactEmptyComponent.create（依赖注入，在RN中是一个空的View。空节点是当render返回为null时渲染的，children里的null会被过滤掉）
    2. ReactHostComponent.createInternalComponent（依赖注入，RN中不存在该类型的实例）
    3. ReactHostComponent.createInstanceForText（字符串，依赖注入，ReactNativeTextComponent，类似于ReactNativeBaseComponent）
    4. ReactCompositeComponentWrapper（ReactCompositeComponent） 
    5. element.type（ReactNativeBaseComponent）

*******

## 生命周期：ReactComponent提供实现，ReactCompositeComponent管理调用       

1. 在开发过程中我们会给有需要的组件实现它的生命周期函数
2. 一个组件在render中是被识别为element的，根据element创建的ReactCompositeComponent的实例（internalInstance）的renderedComponent即组件的实例（publicInstance）
3. 一个ReactComponent节点的挂载、更新、卸载会调用internalInstance的mountComponent、updateComponent、unmountComponent。在此期间触发其对应的publicInstance实现的各生命周期函数

*******

## virtual dom结构            

1. React的virtual dom树即由element组成的节点树，这些节点包含自定义组件、原生组件
2. 每个自定义组件都可以解析为原生组件组成的节点树
3. 原生组件可以绘制成Native中真实View，真实的View由ReactNativeBaseComponent通过调用UIManager绘制
4. ReactCompositeComponent的渲染本质是处理数据，并调用向下层节点的渲染，当遇到ReactNativeBaseComponent时，才真正绘制
5. 当ReactNativeBaseComponent绘制完，继续处理其children节点，循环2-4，最终整棵树处理完成实现了virtual dom在Native中的绘制

*******

## render与逐层渲染          

### 首次挂载流程   
1. 顶层element -> 实例化为internalInstance（ReactCompositeComponent） -> internalInstance.mountComponent -> render获得下一级element（重复实例化并mountComponent，直至遇到ReactNativeComponent） -> UIManager.createView(Native中真实创建View，返回关联的tag) -> 遍历children， 对child（element）实例化为internalInstance，重复上述挂载过程 -> UIManager.setChildren(将子节点渲染的tag与父tag关联)    

### 更新流程
1. 当触发更新时，由一个节点开始，向下逐层解析，中间经过diff策略（element diff、component diff、children diff）   
2. element diff： 比较前后该位置的element，若其可以更新，则向下继续更新；若不可更新，卸载纠节点，挂载新节点
3. component diff: shouldComponentUpdate
4. children diff: 移动需保留的旧节点并更新，删除废弃的旧节点并卸载，插入新建的节点并挂载