# KVO的底层原理

## 概述

KVO是一套事件通知机制，他的功能是让一个对象去监听另一个对象特定属性的改变，并在属性值发生改变的时候派发一个事件，让其观察者接收，一般继承`NSObject`的类都默认支持KVO。

KVO和`NSNotificationCenter`都是iOS中观察者模式的一种实现，区别在于相对观察者和订阅者之间的关系，KVO是一对一的，而`NSNotificationCenter`是一对多。KVO对其被监听对象无代码侵入性，在不修改其内部实现情况下，就可实现属性监听。

KVO可以监听单个属性的变化，也可以监听集合对象的变化，例如`NSArray`和`NSSet`。

## 基本使用

KVO的基本使用分三个步骤：

1. 注册观察者，管着可接受到keyPath属性的变化事件。
2. 在观察中实现事件回调方法，`observeValueForKeyPath:ofObject:change:context:`，当keyPath属性发生改变时，KVO会回调此方法来通知观察者。
3. 当观察者不需要再继续监听keyPath的时候，需要在观察者被释放之前，调用`removeObsever:forKePath`来移除监听。

## 实现原理

KVO是通过`isa-swizzling`技术来实现的。当我们监听类的某属性时，在运行时会动态的创建一个名为`NSKeyNotifiying_ClassName`的中间类，并将被观察者类实例对象的isa指针指向这个中间类，并将class方法重写，返回被观察者的原类。

在中间类中，根据keyPath值来重写对象属性的setter方法，在修改值的前后分别调用`willChangeValueForKey:`和`didChangeValueForKey:`，这两个方法调用后，都会回调`observeValueForKeyPath: ofObject:change:context:`。