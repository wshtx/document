# Reactive Mutiny

- [Reactive Mutiny](#reactive-mutiny)
  - [Mutiny Guides](#mutiny-guides)
    - [Mutiny operators](#mutiny-operators)
      - [Create items](#create-items)
      - [Filter items](#filter-items)
      - [Transform/Map items](#transformmap-items)
    - [Blocking and Non-blocking](#blocking-and-non-blocking)
    - [Stream operations](#stream-operations)
      - [`Uni` stream operations](#uni-stream-operations)
      - [`Multi` stream operations](#multi-stream-operations)
      - [Join several Unis](#join-several-unis)
    - [Handling failures](#handling-failures)
      - [Retrying on failures](#retrying-on-failures)
    - [Other Guides](#other-guides)
  - [Reference](#reference)

>[B站Rxjava视频课程](https://www.bilibili.com/video/BV1H54y1j7uN?p=21&vd_source=ba265019a7cefebef2404097db580673)  
>[Events list](https://smallrye.io/smallrye-mutiny/1.7.0/tutorials/observing-events/)

Mutiny可以类比Rxjava进行学习，都是reactive programming库. ***reactive programming = 响应式数据流  + 事件驱动  +  观察者模式***
在Rxjava中的变种观察者模式中，显示区分出被观察者(Observable)，观察者(Observer)，事件发射器(Emitter)，中间关系类(ObservableOnSubscription)四种角色. Mutiny提供的`Uni/Multi`类型就是被观察者角色. 整个响应式事件流的构建和订阅过程如下图：

![Rxjava](https://cdn.jsdelivr.net/gh/wshtx/myImageHosting/img/20221104113452.png)

## Mutiny Guides

![同步模型](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/blocking-threads.png)
![Mutiny 异步模型](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/reactive-thread.png)

### Mutiny operators

#### Create items

- [Create `Uni` pipelines](https://smallrye.io/smallrye-mutiny/1.7.0/tutorials/creating-uni-pipelines/#creating-unis-using-an-emitter-advanced)  
- [Create `Multi` pipelines](https://smallrye.io/smallrye-mutiny/1.7.0/tutorials/creating-multi-pipelines/)

&nbsp;&nbsp;&nbsp;&nbsp;Note:

- ***Multis are lazy by nature. To trigger the computation, you must subscribe.if you don’t subscribe, nothing is going to happen.***
- Note create `Uni/Multi` will produce the returned Cancellable: this object allows canceling the stream if need be.

#### Filter items

[Conditional filter items](https://smallrye.io/smallrye-mutiny/1.7.0/guides/filtering-items/)  
[Skip/Take items](https://smallrye.io/smallrye-mutiny/1.7.0/guides/take-skip-items/)  
[De-duplication items](https://smallrye.io/smallrye-mutiny/1.7.0/guides/eliminate-duplicates-and-repetitions/)

#### Transform/Map items

[Transfrom items](https://smallrye.io/smallrye-mutiny/1.7.0/tutorials/transforming-items/)  
[Transfrom items asynchronous](https://smallrye.io/smallrye-mutiny/1.7.0/tutorials/transforming-items-asynchronously/)  
[Map/FlatMap/Concatmap](https://smallrye.io/smallrye-mutiny/1.7.0/guides/rx/)  

### Blocking and Non-blocking

- [How to run your blocking code in a pure reactive application?](https://smallrye.io/smallrye-mutiny/1.7.0/guides/imperative-to-reactive/)
- [What is the difference between emitOn and runSubscriptionOn?](https://smallrye.io/smallrye-mutiny/1.7.0/guides/emit-on-vs-run-subscription-on/)
- [How to do blocking in a reactive application?](https://smallrye.io/smallrye-mutiny/1.7.0/guides/reactive-to-imperative/)

### Stream operations

#### `Uni` stream operations

- [How to join several `Unis`?](https://smallrye.io/smallrye-mutiny/1.7.0/guides/joining-unis/)

#### `Multi` stream operations

- [Merge/Concatenate stream and what is the difference between them?](https://smallrye.io/smallrye-mutiny/1.7.0/guides/merging-and-concatenating-streams/)
- [Combine `Unis` or `Multis` from different stream](https://smallrye.io/smallrye-mutiny/1.7.0/guides/combining-items/)
- [Cold stream and hot stream](https://smallrye.io/smallrye-mutiny/1.7.0/guides/hot-streams/)
- [Replay/Reuse the `Multi` items](https://smallrye.io/smallrye-mutiny/1.7.0/guides/replaying-multis/)
- [Control the demand number](https://smallrye.io/smallrye-mutiny/1.7.0/guides/controlling-demand/)

#### Join several Unis

>[Joining several unis](https://smallrye.io/smallrye-mutiny/1.7.0/guides/joining-unis/)

### Handling failures

>[Handling failures](https://smallrye.io/smallrye-mutiny/1.7.0/tutorials/handling-failures/)

&nbsp;&nbsp;&nbsp;&nbsp;Failures are terminal events sent by the observed stream, indicating that something bad happened. **After a failure, no more items are being received**.
When such an event is received, you can:

- propagate the failure downstream (default), or
- transform the failure into another failure, or
- recover from it by switching to another stream, passing a fallback item, or completing, or
- retrying (covered in the next guide)
  
&nbsp;&nbsp;&nbsp;&nbsp;If you don’t handle the failure event, it is propagated downstream until a stage handles the failure or reaches the final subscriber.

&nbsp;&nbsp;&nbsp;&nbsp;**Note**:on Multi, a failure cancels the subscription, meaning you will not receive any more items. The retry operator lets you re-subscribe and continue the reception.

#### Retrying on failures

- [Retrying on failures](https://smallrye.io/smallrye-mutiny/1.7.0/tutorials/retrying/)
- [Mutiny - How does retry... retries?](https://quarkus.io/blog/uni-retry/)

### Other Guides

- [How to handle null](https://smallrye.io/smallrye-mutiny/1.7.0/guides/handling-null/)
- [How to handle  timeouts](https://smallrye.io/smallrye-mutiny/1.7.0/guides/handling-timeouts/)
- [How to delay events](https://smallrye.io/smallrye-mutiny/1.7.0/guides/delaying-events/)
- [How to use paginated APIs?](https://smallrye.io/smallrye-mutiny/1.7.0/guides/pagination/)
- [How to use polling?](https://smallrye.io/smallrye-mutiny/1.7.0/guides/polling/)
- [How to write unit/integration tests using assertion?](https://smallrye.io/smallrye-mutiny/1.7.0/guides/testing/)
- [How to custom operators?](https://smallrye.io/smallrye-mutiny/1.7.0/guides/custom-operators/)
- [How to use `Context` in `Uni/Multi`?](https://smallrye.io/smallrye-mutiny/1.7.0/guides/context-passing/)

## Reference

- [Why is asynchronous important?](https://smallrye.io/smallrye-mutiny/1.7.0/reference/why-is-asynchronous-important/)
- [What is Reactive programming?](https://smallrye.io/smallrye-mutiny/1.7.0/reference/what-is-reactive-programming/)
- [What makes Mutiny different?](https://smallrye.io/smallrye-mutiny/1.7.0/reference/what-makes-mutiny-different/#events)
