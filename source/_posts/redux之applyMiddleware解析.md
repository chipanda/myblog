---
title: redux之applyMiddleware解析
date: 2018-03-01 15:04:22
tags:
  - react
  - redux
categories:
  - react
---

redux允许添加中间件的方式封装dispatch，源码实现很简单（很精彩），但涉及到多层闭包的使用和函数式编程在初次理解时还是要费些功夫的。

## applyMiddleware源码

    export default function applyMiddleware(...middlewares) {
        return createStore => (...args) => {
            const store = createStore(...args)
            let dispatch = () => {
            throw new Error(
                `Dispatching while constructing your middleware is not allowed. ` +
                `Other middleware would not be applied to this dispatch.`
            )
            }
            let chain = []

            const middlewareAPI = {
            getState: store.getState,
            dispatch: (...args) => dispatch(...args)
            }
            chain = middlewares.map(middleware => middleware(middlewareAPI))
            dispatch = compose(...chain)(store.dispatch)

            return {
            ...store,
            dispatch
            }
        }
    }

## compose源码

    export default function compose(...funcs) {
        if (funcs.length === 0) {
            return arg => arg
        }

        if (funcs.length === 1) {
            return funcs[0]
        }

        return funcs.reduce((a, b) => (...args) => a(b(...args)))
    }

## middleware的结构

applyMiddleware处理middleware的数组，而每个middleware都是一个函数，结构如下：

    function (store) {
        return function (next) {
            return function (action) {
                //中间件内容
                    next(action);
                //中间件内容
            }
        }
    }
es6 写法为     
    
    store => next => action => { next(action); } 

可见一个middleware可拆解为3层，若有middleware为A，我们称

    1. first_A代表 store => next => action => { next(action); }
    2. second_A代表 first_A的执行结果，即next => action => { next(action); }
    3. third_A代表 second_A的执行结果，即action => { next(action); }

## 将middlewares组合成最终的dispatch

1. `chain = middlewares.map(middleware => middleware(middlewareAPI))`通过注入middlewareAPI（即store）获得了second_middleware的数组
2. 若假设chain为[a, b, c]，compose(chain)的执行结果为      
    
        function (...args) {
            return a(b(c(...args)));
        }
    而compose(...chain)(store.dispatch)即为

            a(b(c(store.dispatch)))

    那a、b、c又是什么呢，参见前文它们是与中间件A、B、C对应的second_A、second_B、second_C。每个second_middleware都接受一个next函数，并返回一个处理action的函数。所以`a(b(c(store.dispatch)))`的执行结果为：

        function thirdA(action) {
            // thirdA内容
                (function thirdB(action) {
                    // thirdB内容
                        (function thirdC(action) {
                            // thirdC内容
                                store.dispatch(action);
                            // thirdC内容
                        })(action);
                    // thirdB内容
                })(action);
            // thirdA内容
        }

## 反向理解融合各middleware的原理

1. 已知中间件A，B，C，每个中间件有处理action的能力，可得`action => {}`
2. 若将A，B，C串联起来，当A处理完交给B，则需要为每个middleware指定下一个处理action的方法next，next由闭包注入获得，即`next => action => { next(action) }`
3. 中间件内部需要能对store读取，继续处理为`store => next => action => { next(action) }`的最终中间件格式

## 总结

1. applyMiddleware充分利用了js的闭包特性，为处理action的中间件方法提供了具备store、next的作用域
2. 基于函数式编程的设计理念，使中间件的结构允许其依次执行