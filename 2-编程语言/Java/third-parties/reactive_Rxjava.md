# Rxjava

Rxjava编程思想 = **事件驱动 + 观察者模式 + 异步**

## 观察者模式

相比于传统观察者模式，RxJava中进行了改进，引入事件发射器，彻底将观察者和被观察者解耦。

![Rxjava 观察者模式](https://cdn.jsdelivr.net/gh/wshtx/myImageHosting/img/20221103161044.png)

RxJava观察者模式涉及元素：

- ObservableSource : 被观察者的顶层接口，用于创建被观察者和观察者之间的订阅关系
- Observable : 被观察者的抽象类，具体订阅逻辑由给子类实现
- ObservableCreate : 被观察者的具体实现类
  
---

- ObservableOnSubscribe : 中间接口，调用事件发射器，建立被观察者和事件发射器之间的**逻辑关系(并没有实际持有引用)**
- Emitter : 事件发射器的顶层接口，持有观察者引用(直接调用观察者)，将被观察者和观察者解耦
- CreateEmitter : 事件发射器的具体实现类

---

- Observer : 观察者的顶层接口，对各种事件进行处理
  
## 装饰器模式

![Rxjava 装饰器模式](https://cdn.jsdelivr.net/gh/wshtx/myImageHosting/img/20221103172149.png)

Rxjava 装饰器模式涉及元素：

- Component : 被装饰类的顶层接口或抽象类
  
---

- AbstractDecorator : 抽象装饰类，需要实现或继承被装饰类的抽象类或顶层接口
- ConcreteDecorator : 具体装饰类
