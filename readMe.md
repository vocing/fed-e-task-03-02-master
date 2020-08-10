Vue.js 源码剖析-响应式原理、虚拟 DOM、模板编译和组件化

一、简答题

1、请简述 Vue 首次渲染的过程。

答: 1.如果是包含编译器的, 重写Vue.prototype.$mount, 增加模板编译方法，在入参对象中没有render属性却有template属性的时候, 将template编译成render函数并设为入参对象render属性的值.
    2.Vue初始化, 初始化实例成员、静态成员
    3.调用Vue.$mount, 判断编译render函数、执行mountComponent函数
    4.执行mountComponent函数,
      a.判断是否有render选项，如果没有但是传入了模板，并且当前是开发环境的话会发送警告
      b.触发beforeMount, 设置updateComponent方法,装入vm._update方法(将虚拟DOM转换成真实DOM, 如果是开发环境同时会启用性能监控)
      c.设置Watcher监听, 并调用get方法, 执行传入的updateComponent() --> 执行vm._update()

2、请简述 Vue 响应式原理。

答: 1.调用 initState() --> initData() --> observe(), 依次从父函数中调用相应方法, 直到observe方法
    2.判断value是否是对象，如果不是对象直接返回、判断value对象是否有__ob__，如果有直接返回、创建observer对象， 并返回该对象
    3.给value对象定义不可枚举的__ob__属性，记录当前的observer对象， 并判断是否为数组, 数组使用observeArray方法，对象使用walk方法
    4.为数组及数组每项添加defineProperty双向绑定(且数组的部分方法被改写, 在调用方法后触发watcher的发布方法), 为对象的每个属性添加defineProperty双向绑定
    5.添加watcher监听, 执行依赖收集, 监听到调用发布方法后执行数据变更

3、请简述虚拟 DOM 中 Key 的作用和好处。

答: 1.key的设置, 在patch过程中, 判断新旧节点数组中是否含有相同节点, 从而提取相同节点
    2.这样可以减少一些不必要的重复渲染, 优化了界面渲染效率

4、请简述 Vue 中模板编译的过程。

答: 本过程由外至内入参, 取出内方法生成的数据
    1.调用compileToFunctions函数中取出 render、staticRenderFns: const { render, staticRenderFns } = compileToFunctions(...args)
    2.调用createCompiler函数生成compileToFunctions函数: const { compile, compileToFunctions } = createCompiler(baseOptions)
    3.调用createCompilerCreator函数生成createCompiler函数： const createCompiler = createCompilerCreator(function baseCompile (){...})
    4.createCompilerCreator函数, 入参baseCompile函数, 返回一个createCompiler函数, baseCompile在createCompiler函数中调用: (baseCompile) => () => {...; baseCompile(...args)}
    5.baseCompile函数, 调用parse生成ast抽象语法树, 调用generate函数, 将template模板编译为render函数
    函数生成过程: createCompiler = createCompilerCreator(baseCompile)
    :{
      compile(){ ...; baseCompile(...args)},
      compileToFunctions = createCompileToFunctionFn(compile) : {
        render,
        staticRenderFns
      }
    }
    总: 1.compile方法调用baseCompile函数, 执行generate函数将template模板编译为render字符串, 返回{ast、render、staticRenderFns}
        2.compileToFunctions 由createCompileToFunctionFn生成, 内部调用compile方法, 且还会将render字符串生成为render函数, 然后返回{render、staticRenderFns}并记入缓存便于下次取用

