A adapter(glue) between UI and data model.

Why this gap are exist:
Data model lack of attributes which used to 'summary' it 's state

UI are compose by two parts: 
1. entity with state, attributes
2. structure of entity.

State are propagating

前端在使用后端的数据时, 往往需要做一层适配, 因为后端的业务逻辑的建模, 往往不是针对前端展示做的.

但有一种 "适配", 是应该避免的.

那就是模型 (model) 的状态, 总结性的属性: process.isRunning.

这种状态往往具有传递性, 例如, 一个 entity(记为 entityA) 的状态, 往往取决于它聚合的其他 entity(记为 entityB) 的状态.

这个时候, entityA 和 entityB 有应该有一个属性, 表示他们的状态.

而不是前端去做聚合.

**前端不应该对业务模型进行建模.**

就如同, 数组有一个 length 属性, 但其实没有这个属性, 我们 **可以** 通过一个元素一个元素去数, 最终得到这个属性, 但 **不应该** 这么做.
