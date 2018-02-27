---
title: '## React的更新流程（三）'
date: 2018-02-27 10:54:54
tags:
---

前一节讲述了单个节点的更新，本节介绍Virtual DOM树中的多个子节点是如何更新的，即ReactNativeBaseComponent的renderedChildren的更新。
<!-- more -->
*******     

## ReactChildReconciler

辅助ReactNativeBaseComponent处理其children的挂载、更新、卸载，按规则调用相应的ReactReconciler方法

* instantiateChildren，使用travelAllChildren遍历children，生成child（element）的internalInstance       
* updateChildren，对比prevChildren和nextChildren，对旧的internalInstance更新或卸载，实例化新插入的element        
* unmountChildren，遍历children，逐个卸载   

*******     

## ReactMultiChild

拓展ReactNativeBaseComponent，作为中间层调用ReactChildReconciler，包含了重要的children diff更新策略。
任意嵌套结构的children都可以展开为一级，如`[A,B,[C,D,[E,F]],G]`可以展开为`[A,B,C,D,E,F,G]`

* mountChildren，遍历children，实例化每个element为internalInstance并挂载
* updateChildren，首先通过ReactChildReconciler处理children的实例操作，然后计算得出重新排序的操作队列（children different）并处理         
    1. 通过插入、移动、删除三种操作来更新旧children，把旧children改造为新children
    2. 按序遍历新children，并产生相应的update（插入、移动或删除），这样保证了处理到一个child时其前面的child已经排序完成（位置是可靠的）        
    3. 新节点插入在前一个节点之后     
    4. 保留的旧节点分为两类：需要移动的，维持原位置       
        * 使用lastIndex记录处理过的旧节点的最大旧的mountIndex，初始为0
        * 若当前节点的旧mountIndex < lastIndex（前面的旧节点是排好序的）,所以需移动该节点到前一个处理过的节点之后
        * 反之，保留它的位置就可保证当前节点的位置是正确的，则不创建update，维持原位置     
    5. 删除废弃的旧节点        
* unmountChildren，通过ReactChildReconciler卸载children