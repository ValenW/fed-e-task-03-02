# 王文茏｜Part 3｜模块二

## 简答题

### 第一题

1. `new Vue`创建vue实例, 调用vue的构造函数
2. 构造函数中调用`_init`方法
3. `_init`方法初始化Vue, 最后调用`vm.$mount`将vue挂载到`vm.options.el`上

#### $mount

1. 没有render函数, 则调用compileToFunction将template编译为render函数
2. 调用mountComponent函数
   1. 定义updateComponent函数: `vm._update(vm._render(), ...)`
   2. 将updateComponent作为callback, 创建render watcher
3. 调用`watcher.get`, 即执行`updateComponent`函数, 将vue挂载到页面上

### 第二题

#### 收集依赖

##### initState阶段, 初始化属性getter

1. 每个响应式数据都绑定有一个Observer, 创建时会将数据每个属性都转换为getter/setter
2. 每个属性都绑定有一个Dep对象, 用于收集依赖(Watcher)
3. getter中会调用dep.depend, 为当前render watcher和dep建立联系
   1. 将当前渲染组件的Watcher对象加入到dep.subs中
   2. 将dep加入到watcher.newDeps中

##### mount阶段, 调用getter收集依赖

1. 每个组件都绑定有一个render watcher, 用于监听其依赖的属性的变化
2. 组件渲染时会初始化Watcher, 并调用watcher.get()
3. get方法首先会赋值Dep.target为当前watcher
4. 之后会渲染组件, 就需要访问组件依赖的属性, 调用其getter, 从而为watcher和dep建立联系

#### 通知更新

1. 属性的setter中会调用`dep.notify`, 按照id顺序调用`watcher.update`
2. 如果是同步watcher, 直接执行`watcher.run`, 否则将watcher加入到一个队列中, 等待执行更新
3. 最终都会执行`watcher.run`, 通过`watcher.get()`更新视图或数据

### 第三题

在更新子节点的过程中, 我们需要遍历和对比新旧队列中两个vnode是否是同一节点, 来判断是更新旧dom还是直接插入新节点.

key的作用就是为了标识两个vnode对应的是同一个节点, 这样就能更快地找到队列中的相同节点/没有对应旧节点, 从而提高diff算法的效率.

如果没有设置key, 则只能通过tag来判断两个vnode是否是同一个节点, 这样在patch的时候会做更多的更新操作, 降低效率.

### 第四题

1. 将模板编译为抽象语法树(AST)
   1. 将template解析为AST对象, 借鉴了simplehtmlparser
   2. 基本原理是通过正则匹配各种标签, 在遇到开始/结束/字符等节点时, 调用对应的处理函数生成AST对象
2. 遍历标记AST中的静态根节点, 用于优化AST
   1. 静态节点就是内容永远不发生变化的节点, 如纯文本节点
   2. 如果元素只有一个文本子节点, 则不将其标记为静态根节点, 因为Vue认为这种优化得不偿失
3. 调用`generate`将AST转换为字符串形式的js代码
   1. genElement, 将普通AST节点根据不同类型, 如static, v-if, slot, 扑通标签等, 都编译为对应的render函数
   2. 对于static节点, genStatic会调用genElement获得render函数并缓存起来, 达到优化的目的
4. 将字符串js代码转换为render函数并返回 `{ render, staticRender}`
   1. 使用的方法是`new Function(code)`





