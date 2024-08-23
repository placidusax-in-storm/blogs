Module Federation提出的解决方案是将前端之间的依赖放在运行时注入
bazel(或者大仓)提出的解决方案是将依赖在构建时就进行注入

Module Federation提出的设想是 "各个APP之间独立构建, 独立部署",
但其实依赖关系已经形成, 一个被其他APP引用的APP(remote module)其设计和迭代就需要考虑引用方的使用方式
版本管理和接口协议才是维护的痛点

Module Federation没有对此提供完善的解决方案, 但bazel(或者大仓)已经对此提供了一套解决的方法论"单一版本, 统一管理"

所以这是两个截然不同的方案, 既然我们已经选择了大仓, 那么应该不会走Module Federation的方案