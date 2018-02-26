---
title: React中的一些辅助工具类
date: 2018-02-26 10:57:54
tags:
---

本节介绍几个工具类，React中更新任务的执行与管理依赖它们辅助完成。
<!-- more -->

## Transaction：在执行目标方法前后执行配置好的方法    

为方法funcA添加包装层wrappers，每个wrapper有initialize、close两个方法。在funA执行前后分别一次执行wrappers的initialize、close。若wrappers为[wrapper1, wrapper2]，执行的顺序为wrapper1.initialize->wrapper2.initialize->funcA->wrapper1.close->wrapper2.close。

使用方法
* 定义一个transaction方法，其prototype混入Transaction的Mixin
* 实现getTransactionWrappers方法，指定wrappers
* 通过transaction方法实例的perform来执行指定方法funcA，以达到在执行funA前后执行wrapper的目的

perform执行有以下特点
* 执行过程中的异常会被捕获，并继续执行原流程中的下一步
* 每个wrapper的initialize的返回值作为其close执行时的参数

## CallbackQueue：管理回调队列     

顾名思义，管理一个回调队列，并在需要时执行。

使用方法
* 获得一个CallbackQueue实例
* enqueue\(callback，context\)，在队列里压入新的回调方法和其执行作用域
* notifyAll\(\)，依次执行回调队列中的回调方法

## PooledClass：为了优化实例内存而做的类拓展      

为优化频繁创建的对象实例而设计，当对象使用后不销毁而是初始化其参数，等待之后需要新实例时重置空闲对象的参数以供使用，避免了重新创建新对象。

使用方法
* 通过PooledClass.addPoolingTo改造KClass，KClass.prototype需实现destructor方法，用来在实例释放时初始化参数
* 通过KClass.getPooled()获取实例
* 通过KClass.release(instance)释放实例（内部调用了destructor）

## traverseAllChildren：遍历this.props.children     

遍历数组，包含数组中为数组类型的元素。用于处理某个节点的子节点（RN中为ReactNativeBaseComponent的子节点）

功能说明
* traverseAllChildren(children, callback, traverseContext)，返回值为节点个数
* 遍历到某个元素时，对其执行callback，traverseContext作为参数传入callback，可将需要的结果放入traverseContext中
* 当children为一个element时，则只对其执行callback，返回1
* 每个节点的都有自己的name，作为参数传入callback，name的命名规则为：`父节点名称:自己的key（若不存在则使用index）`
* 第一层节点的命名为：`.自己的key`