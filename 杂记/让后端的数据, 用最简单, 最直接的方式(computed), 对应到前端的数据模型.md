## Bad

```jsx
create(){
    const response = await getData();
    const model = formatData(response);
    this.model = model;
}
```

## Good

```jsx
create(){
    this.response = await getData();
}

computed(){
    return formatData(this.response);
}
```

应避免 "命令式 地定义 派生状态", 比如在生命周期函数(created)中, 去手动地进行数据绑定:

```jsx
create(){
    const response = await getData();
    this.model =  formatData(response);
}

```

这里的`model` 是 `response` 的派生状态, 这里用命令式的方式, 手动在事件出发的时候, 绑定这个派生状态. 这样会使得这个派生状态的派生关系不够明显, 因为这个派生关系是依赖于一个 事件 (created) 的. 更推荐的方式是, 直接用 computed 去做这个映射, 因为 computed 将映射关系清晰地写在一个函数中, 并排除了事件的干扰, 它清晰地说明了数据之间的派生, 依赖关系:

```
computed: {
    model(){
        return formatData(this.response)
    }
}

```

但这样也存在性能损耗 — 把一个后端数据, 直接塞入到整个 state 中. 我_没有_详细地测试这种性能损耗有多大, 但粗浅地估计还好? 我 _猜测_:

1. vue2 中 _可能不会_ 在 `this.response = response` 的时候, 遍历整个 `response`, 将其所有属性定义为 Reactive. 而是将 `response` 的 reference 作为监听对象, 监听 一个 object 的索引, 损耗是不高的.
2. vue3 我没用过, 但 vue3 使用 Proxy 作为响应式数据的基础, 哪怕是做 deep watch, 也不需要预先递归地将所有属性进行预处理, 只需要通过 Proxy 动态地拦截, 性能损耗可能也还好 (这里瞎猜成分比较大).