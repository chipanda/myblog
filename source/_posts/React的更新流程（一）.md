---
title: React的更新流程（一）
date: 2018-02-26 15:51:08
tags:
---
React的更新流程（批处理更新）是围绕待更新组件（React称为dirtyComponent）来实现，dirtyComponent即ReactCompositeComponent实例。

批处理更新过程如下：
1. 开始一次批处理更新
2. 收集dirtyComponents
3. 更新dirtyComponent（更改UI的操作在这里）
4. 执行回调（生命周期函数、setState传入的回调方法）
5. 若在3、4间产生了新的dirtyComponent，重复3、4步，直至dirtyComponents清空，完成了一次完整的批处理更新
<!-- more -->
*******     
## ReactDefaultBatchingStrategy

全局变量，管理一次更新流程的开始、结束。功能如下：

* isBatchingUpdates，标识当前是否处于更新流程中
* batchedUpdates方法，接受一个callback参数
    * 若当前处于一次更新流程中，直接执行
    * 反之，isBatchingUpdates = true，然后通过ReactDefaultBatchingStrategyTransaction来执行callback
* ReactDefaultBatchingStrategyTransaction有两个wrapper
    * wrapper1：
        * initialize为空
        * 调用ReactUpdates.flushBatchedUpdates
    * wrapper2:
        * initialize为空
        * isBatchingUpdates = false

*******

## ReactUpdates

1. 更新流程的开始结束由ReactDefaultBatchingStrategy控制，但更新流程的发起方是ReactUpdates。

2. 在当前页面处于未更新的稳定状态时，更新流程开始于两种情况：
    1. RN原生组件的绑定事件触发（ReactUpdates.batchedUpdates）
    2. 定时器、请求等异步操作的回调方法更改了state或props（ReactUpdates.enqueueUpdate）

3. batchUpdates方法
    
    接受一个callback，内部实际执行了ReactDefaultBatchingStrategy.batchedUpdates\(callback\)

4. enqueueUpdate方法

    接受一个component          
    * 若处于更新流程中，将其加入dirtyComponents中
    * 若不处于更新流程中，发起一次更新流程，将其加入dirtyComponents中

5. flushBatchedUpdates

    * 开始时记录当前dirtyComponents个数，更新dirtyComponents
    * 在更新完成后，若更新过程中产生了新的dirtyComponents（通过前后个数对比判断，或本次transaction执行完成后，再判断dirtyComponents非空），再次执行一次flushBatchedUpdates，直至dirtyComponent被清空
    * 然后执行所有注册的回调
*******

## ReactUpdateQueue

1. 更新流程的开始结束由ReactDefaultBatchingStrategy控制，但更新流程的发起方是ReactUpdates。

2. 在当前页面处于未更新的稳定状态时，更新流程开始于两种情况：
    1. RN原生组件的绑定事件触发（ReactUpdates.batchedUpdates）
    2. 定时器、请求等异步操作的回调方法更改了state或props（ReactUpdates.enqueueUpdate）

1. setState、forceUpdate方法本质是通过ReactUpdatesQueue将component加入dirtyComponents

2. 若有callback，会将component两次加入dirtyComponents，有两种情况：       
    1. 两次在同一个更新流程中，即setState时已经处于更新流程中
    2. 两次在不同的更新流程中，即setState未处于更新流程中，第一次加入dirtyComponents本身执行了一次更新流程，后一次加入又执行了一次

3. 还记得setState方法后直接获取的state不一定是最新的吗。
    1. 当setState执行时已经处于更新流程中，setState内部执行只增加了dirtyComponent而未更新dirtyComponent，所以setState执行后state是旧的
    2. 当setState执行时未处于更新流程中，setState内部执行时不仅加自己入了dirtyComponent同时也完成了更新流程，所以setState后直接获取的state是新的