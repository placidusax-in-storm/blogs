一个表单页面, 由于写成 uncontrolled 组件, 导致组件中有两份数据, 一份是储存在子组件的 ref 里(展示在 input 框中), 一份储存在组件的 state 里, 等到提交的时候, 再去从 ref 里读 input 里的值, 并将其 Object.assign 到组件的的 state 里.

可以简化为, 将表单改造为 controlled 组件, 只维护一份 state, only source of truth.

这个便于去追踪组件的状态, console.log 一下组件的 state 便一目了然.

uncontrolled 的写法, 使得**在某一个时间点, 组件里的 state 是脏数据, 或者说过期数据**, 需要等到提交时 "从 ref 里读 input 里的值, 并将其 Object.assign 到组件的的 state 里”, 这个时候组件的数据才不是 "过期的数据”.

即缓存失效.