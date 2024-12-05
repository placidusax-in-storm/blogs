---
layout: post
title: io-ts 前置基础知识
tags:
  - 函数式编程
---
对io-ts的内部实现有所了解才能更好地使用其API, 而io-ts的实现上使用了函数式编程的两个概念 -- 代数数据类型, tag-less-final.

## 代数数据类型

### 代数数据类型是什么
[参考*函数式编程导言*](https://jituanlin.github.io/%E5%8E%9F%E5%88%9B/2020-07-27-%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BC%96%E7%A8%8B%E5%AF%BC%E8%A8%80/#%E4%BB%A3%E6%95%B0%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B)

### io-ts内部使用到的ADT

#### Semigroup/FreeSemigroup
对`Semigroup`的定义:
```ts
export interface Semigroup<A>{
    readonly concat: (x: A, y: A) => A
}
```
即, `Semigroup`语义上表示一个类型支持*合并操作*(`concat`).

`FreeSemigroup`本身不属ADT, 而是`type`, 即它是`Semigroup<A>`中的`A`, 而非`Semigroup<A`本身, 其定义如下:
```ts
export interface Of<A> {
    readonly _tag: 'Of'
    readonly value: A
}

export interface Concat<A> {
    readonly _tag: 'Concat'
    readonly left: FreeSemigroup<A>
    readonly right: FreeSemigroup<A>
}

export type FreeSemigroup<A> = Of<A> | Concat<A>
```
即, `FreeSemigroup`本身是一颗二叉树的结构.

`Semigroup<FreeSemigroup<A>>`构造函数如下:
```ts
export function getSemigroup<A = never>(): Semigroup<FreeSemigroup<A>> {
    return { concat }
}
```
通过将类型参数传入无参构造函数`getSemigroup`, 如`getSemigroup<string>`, 返回即为`Semigroup<FreeSemigroup<A>>`.

之所以需要借助一个无参构造函数而不是直接引用:
```ts
const FreeSemigroup = {
    concat
}
```
是因为需要借助该构造函数传入类型参数`A`.

io-ts内部使用`FreeSemigroup`表示validate时的报错信息:
```ts
export type DecodeError = FS.FreeSemigroup<DE.DecodeError<string>>
```

`DE.DecodeError`本身是一个树形结构, 其定义如下:
```ts
// DE.DecodeError
export type DecodeError<E> = Leaf<E> | Key<E> | Index<E> | Member<E> | Lazy<E> | Wrap<E>

export interface Leaf<E> {
    readonly _tag: 'Leaf'
    readonly actual: unknown
    readonly error: E
}

export interface Key<E> {
    readonly _tag: 'Key'
    readonly key: string
    readonly kind: Kind
    readonly errors: FS.FreeSemigroup<DecodeError<E>>
}
```
`DE.DecoderError`为union type, 其type member中除了`Leaf`外, 都有`errors: FS.FreeSemigroup<DecodeError<E>>`
递归属性, 从而构成了树形结构, 即`DecodeError`的树形结构(递归结果)是通过`errors`属性实现的.

而`FreeSemigroup`虽然为二叉树, 但其用途是对同层的`DE.DecodeError`进行一个`concat`收集, 
本质可以使用数组进行替代, 但二叉树的数据结构使得其性能更加优越(`append`和`preappend`都为O(1)复杂度).

但由于二叉树的缘故, `concat`后为了遍历同一层级的`DE.DecodeError`需要一个额外的算法:
```ts
const toForest = (e: DecodeError): ReadonlyArray<Tree<string>> => {
  const stack = []
  let focus = e
  const res = []
  while (true) {
    switch (focus._tag) {
      case 'Of':
        res.push(toTree(focus.value))
        if (stack.length === 0) {
          return res
        } else {
          focus = stack.pop()!
        }
        break
      case 'Concat':
        stack.push(focus.right)
        focus = focus.left
        break
    }
  }
}
```
这里的`toTree`将会接受参数`e: DecodeError`的下的直接子节点做为参数, 实现对其子节点的遍历.

其运行顺序为:
1. "从右往左", 将所有的`Of`节点压入`stack`(`Of`作为子节点, 其`value`储存了`DE.DecodeError`)
2. "从左往右", 遍历(`toTree`)所有的`Of`节点(在`DE.DecodeError`收集(`concat`)的时候, 是从左往右的)

故, `FreeSemigroup`提供的是对任意`type`进行`concat`的能力(此即`Free`命名的由来), 

值得注意的是, 上述`toForest`的实现是为了*stack safe*, 使用递归写法将使该算法相当简洁:
```ts
const toForest: (e: DecodeError) => ReadonlyArray<Tree<string>> = FS.fold(
  (value) => [toTree(value)],
  (left, right) => toForest(left).concat(toForest(right))
)
```

该实现运行步骤如下:

"从左往右"concat所有的`toTree(Of)`

#### Functor category
`Fuctor`的核心操作(operator)为`map`, 其定义如下:
```ts
export interface Functor<F> {
  readonly map: <A, B>(fa: HKT<F, A>, f: (a: A) => B) => HKT<F, B>
}
```
其语义为对一个"上下文中的值"进行计算, 然后将结果重新装入改上下文中.

例如`Promise.prototype.then`其实符合该语义 -- 对一个异步的中的值进行计算, 后
又返回一个异步中的结果.

#### Chain category
`Chain`是在`Functor`的基础上新增了`chain`操作, 其类型定义为:
```ts
export interface Chain<F>{
  readonly chain: <A, B>(fa: HKT<F, A>, f: (a: A) => HKT<F, B>) => HKT<F, B>
}
```
即, `chain`对一个上下文中的值进行计算, 计算的结果同样是一个上下文中的值(`F[B]`而不是`B`),
有实现`Chain`的instance决定如何进行两个上下文的合并.

`Promise.prototype.then`同样符合该语义 -- 对一个异步的中的值进行计算, 得出另外一个Promise, 后
将两个Promise进行合并.

#### Apply category
`Apply`是在`Functor`的基础上新增了`ap`操作, 其类型定义为:
```ts
export interface Apply<F> extends Functor<F> {
  readonly ap: <A, B>(fab: HKT<F, (a: A) => B>, fa: HKT<F, A>) => HKT<F, B>
}
```
即, `ap`将一个上下文中的*函数*, 作用到值上, 后返回一个上下文中的计算结果.
以下为对`Promise`版本的`Apply`的简单实现:
```ts
const ap = <A,B>(fab:Promise<(a:A)=>B>) => (fa:Promise<A>) => fab.then(fa => fa.then(a => fab(a)))
```

#### Applicative category
`Applicatvie`是在`Apply`的基础上扩展了`of`操作, 其类型定义为:
```ts
export interface Applicative<F> extends Apply<F> {
  readonly of: <A>(a: A) => HKT<F, A>
}
```
即, `of`操作将一个值"包裹"于上下文中后返回, `Promise.resolve`同样满足这个语义.

#### Traversable
`Traversable`核心操作为`traverse`与`sequence`(可由`traverse`派生), 其类型定义如下(scala语法):
```scala
trait Traverse[F[_]] {
  def traverse[G[_]: Applicative, A, B](fa: F[A])(f: A => G[B]): G[F[B]]
}
```
即, `traverse`将`F[A]`中的每一个`A`取出, 通过`A => G[B]`转化为`G[B]`(`G`实现了`Applicative`), 
然后通过`Applicative`提供的"操作"将所有的`G[B]`组合成`G[F[B]]`.

`sequence`可由`traverse`派生:
```ts
const sequence = (fa) => traverse(fa, a=>a)
```

即, `sequence`用于不需要进行遍历转化的场景.

`Promise`的`Promise.all`也实现了该(Sequence)语义, 其`traverse`实现示例如下:
```ts
const traverse = <A,B>(fa:A[], fab:(a:A)=> Promise<B>) =>{
    traversableOfArray.reduce(fa, applicativeOfPromise.of(traversableOfArray.empty), (facc,fx)=>{
        const ff = applicativeOfPromise.map(facc, acc=> x => traversableOfArray.concat(acc,x))
        return applicativeOfPromise.ap(ff,fx)
    })
}
```

#### either instance
`either`(`Monad`的实例), 用于表示*可能错误*上下文中的数据, 其定义如下:
```ts
export interface Left<E> {
  readonly _tag: 'Left'
  readonly left: E
}

export interface Right<A> {
  readonly _tag: 'Right'
  readonly right: A
}

export type Either<E, A> = Left<E> | Right<A>
```
即, `Left`表示错误分支, `Right`表示正确分支.

`map`操作可用于对正确分支的串联计算:
```ts
const eitherFirstName = either.map(name=>name.first)(either.right({first:'lin',last:'jit'}))
```
`mapLeft`操作可用于错误分支的串联计算:
```ts
// => either.left('can not get name')
const eitherFirstName = either.mapLeft(error=>error.message)(either.left(new Error('can not get name')))
```
此外还有`bimap`操作对两种可能的分支进行匹配串联计算等, 在此不进行赘述.

`Either`可以实现传统的`throw&catch`异常处理逻辑, 相对于后者, 具有以下优点:

###### 从类型上即可推断其潜在的错误, 及其错误详情的类型
```ts
    const withThrowCatch = (n:number):number =>{
        if(n !== 0){
            return n/10
        }
        throw 'zero can not be divider'
    }   
    
    const withEither =(n:number):Either<number,string> =>{
        if(n !== 0){
            return either.right(10/n)
        }
        return either.left('zero can not be divider')
    }
```
对比两个版本的实现, 使用`either`的版本我们可以从类型信息上知道其函数潜在运行错误的情况, 且从类型上强制其
返回值得消费者必须考虑处理该情况.

而`throw&catch`版本, 使用者必须观察其函数内部实现才能察觉该函数可能抛出异常, 且缺少类型上的约束, 
其返回值的使用者可能忘记处理抛出异常的执行分支.

##### `Either`的串联计算方式更加符合异常处理的流程, 且是引用透明的
大部分的异常的处理逻辑是串联的: 如果函数`fa`抛出异常, 那么依赖函数`fa`的函数`fb`不进行执行, 直接返回`fa`的异常, 其代码实现如下:
```ts
    const fa = () =>either.left('error message')
    const fb = () => {
       return either.map(n => n + 1)(fa())
    }
```
即, `either`处理异常的方式是串联的(链表结构): 上游逻辑的异常会导致下游逻辑直接返回, 不再继续执行.

这种方式使得错误处理的方式是*引用透明的*, 即对调用函数的调用总是可以替换为函数的返回值, 
如上述例子中, 我们可以将对`fa()`的调用替换为`either.left('error message')`, 其执行逻辑不变.

引用透明简化了代码执行的心智模型且便于测试, 这会在其他章节中进行阐述.

而`try catch`的异常处理方式是级联的(树形结构): 调用栈深层的异常会抛出到调用栈外层的函数进行处理,
这使得异常的逻辑处理取决于代码的组织结构(而并非仅仅取决于函数的直接调用位置), 且破坏了函数的引用透明.

#### task instance
`Task`是对`Promise`的thunk, 推迟(隔离)了`Promise`的副作用, 其类型签名如下:
```ts
    type Task<A> = () => Promise<A> 
```
详细的阐述请阅读其他章节.

**注: 不推荐在日常开发中使用`Task`进行抽象(除非用于兼容某些函数库(如io-ts)的使用).**

#### taskEither instance
`TaskEither`是在`Task`的基础上赋予了其错误处理的功能, 其类型定义如下:
```ts
    type TaskEither<A,E> = () => Promise<Either<A,E>> 
```

**注: 不推荐在日常开发中使用`TaskEither`进行抽象(除非用于兼容某些函数库(如io-ts)的使用).**

#### Kleisli
Kleisli范畴又名`ReaderT`, 表示`A => F[B]`的映射, 是对函数(`A => B`)的能力的扩展(`A => Id<B> <=> A => B`).

一般用`ReaderT`表示计算结果有上下文的计算过程, 如, `A => Option<B>`, `A => Either<E,B>`, `A => IO<B>` ... ...


### tag less final
`tag less final`指在编码过程中, 不对函数依赖的上下文(`IO`/`Option`/`Task`/`TaskEither`/...)进行
"硬编码", 而在调用时才进行动态注入, 使得函数更具通用性, 如:
```ts
export interface Kleisli<M extends URIS2, I, E, A> {
    readonly decode: (i: I) => Kind2<M, E, A>
}
```
这使得基于`Kleisli`, 我们可以派生出同步的`Decoder`, 也可以派生出异步的`TaskDecoder`:
```ts
import * as E from 'fp-ts/lib/Either'
import * as TE from 'fp-ts/lib/TaskEither'

export interface Decoder<I, A> extends K.Kleisli<E.URI, I, DecodeError, A> {}
export interface TaskDecoder<I, A> extends K.Kleisli<TE.URI, I, DecodeError, A> {}
```












