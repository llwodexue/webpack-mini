## make 前流程回顾

**步骤**

1. 实例化 `compiler` 对象（它会贯穿整个 webpack 工作过程）
2. 由 `compiler` 调用 run 方法

**compiler 实例化操作**

1. `compiler` 继承 `tapable` 因此它具备钩子的操作能力（监听事件、触发事件，webpack 是一个事件流）
2. 在实例化了 `compiler` 对象之后就往它的身上挂载很多属性，其中 `NodeEnvironmentPlugin` 这个操作就让它具备了文件读写的能力（我们模拟时采用的是 node 自带的 fs）
3. 具备了 fs 操作能力之后有将 `plugins` 中的插件都挂载到了 `compiler` 对象身上
4. 将内部默认的插件与 `compiler` 建立管理，其中 `EntryOptionPlugin` 处理了入口模块的 id
5. 在实例化 `compiler` 的时候只是监听了 `make` 钩子（`SingleEntryPlugin`）
   - 在 `SingleEntryPlugin` 模块的 `apply` 方法中有二个钩子监听
   - 其中 `compilation` 钩子就是让 `compilation` 具备了 `normalModuleFactory` 工厂创建一个普通模块的能力
   - 因为它就是利用一个自己创建的模块来加载需要被打包的模块
   - 其中 `make` 钩子在 `compiler.run` 的时候会被触发，走到这里就意味着某个模块执行打包之前的所有准备工作就完成了
   - `addEntry` 方法调用

**run 方法执行**（当前想看的是什么时候触发了 make 钩子）

1. `run` 方法就是一堆钩子按着顺序触发（`beforeRun`、`run`、`compile`）
2. `compile` 方法执行
   - 准备参数（其中 `newCompilationParams` 是我们后续创建工厂模块的）
   - 触发钩子 `beforeCompile`
   - 将第一步的参数传给一个函数，开始创建一个 `compilation`（`newCompilation`）
   - 在调用 `newCompilation` 的内部
     - 调用了 `createCompilation`
     - 触发了 `this.compilation` 钩子和 `compilation` 钩子的监听
3. 当创建了 `compilation` 对象之后就触发了 `make` 钩子
4. 当我们触发 `make` 钩子监听的时候，将 `compilation` 对象传递过去

**总结**

1. 实例化 `compiler`
2. 调用 `compile` 方法
3. `newCompilation`
4. 实例化了一个 `compilation` 对象（它和 `compiler` 是有关系的）
5. 触发 `make` 监听
6. 调用 `addEntry` 方法（这个时候就带着 `context`、`name`、`entry` 一堆东西）就奔着编译去了