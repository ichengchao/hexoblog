---
title: Jeff Bezos Mandate
date: 2018-01-20 23:34:01
tags:
 - api
 - tech
 - amazon

---

![](http://www.chengchao.name/resource-container/image/jeff-bezos.jpg)

很早之前看过Bezos在Amazon早期的时候发过一个内部技术要求,看完后非常有感触,觉得这几点说的非常好.今天突然想起来就找来了原文,做个备份.

### Mandate

>one day Jeff Bezos issued a mandate, sometime back around 2002 (give or take a year):

- All teams will henceforth expose their data and functionality through service interfaces.
- Teams must communicate with each other through these interfaces.
- There will be no other form of inter-process communication allowed: no direct linking, no direct reads of another team’s data store, no shared-memory model, no back-doors whatsoever. The only communication allowed is via service interface calls over the network.
- It doesn’t matter what technology they use.
- All service interfaces, without exception, must be designed from the ground up to be externalizable. That is to say, the team must plan and design to be able to expose the interface to developers in the outside world. No exceptions.


The mandate closed with:
>Anyone who doesn’t do this will be fired.  Thank you; have a nice day!
>


### 中文翻译
- 从今天起，所有的团队都要以服务接口的方式，提供数据和各种功能。
- 团队之间必须通过接口来通信。
- 不允许任何其他形式的互操作：不允许直连，不允许直接读其他团队的数据，不允许共享内存，不允许任何形式的后门。唯一许可的通信方式，就是通过网络调用服务。
- 具体的实现技术没有限制。
- 所有的服务接口，必须从一开始就以可以公开作为设计导向，没有例外。这就是说，在设计接口的时候，就默认这个接口可以对外部开发人员开放，没有例外。

>不遵守上面规定，就开除。
