# 🚀 手写 Loader

> npm i
> npm run build

## 什么是 Loader

loader 就是一个 node 模块,在 webpack 的定义中，**loader 导出一个函数**,
调用时间：loader 会在转换源模块（resource）的时候调用该函数；

- 输入：传入 this 上下文给 Loader API 来使用它们
- 返回：一般情况下可以通过 return 返回一个值，也就是转化后的值。如果需要返回多个参数，则须调用 this.callback(err, values...) 来返回。在异步 loader 中你可以通过抛错来处理异常情况。Webpack 建议我们返回 1 至 2 个参数，第一个参数是转化后的 source，可以是 string 或 buffer。第二个参数可选，是用来当作 SourceMap 的对象。
- loader 的功能：把源模块转换成通用模块
- 处理后结果：这个函数接受的唯一参数是一个包含源文件内容的字符串。我们暂且称它为「source」

## 1. 配置 webpack config 文件

单个 loader

```js
let webpackConfig = {
  //...
  module: {
    rules: [
      {
        test: /\.js$/,
        use: [
          {
            //这里写 loader 的路径
            loader: path.resolve(__dirname, 'loaders/a-loader.js'),
            options: {
              /* ... */
            }
          }
        ]
      }
    ]
  }
}
```

多个 loader

```js
let webpackConfig = {
  //...
  module: {
    rules: [
      {
        test: /\.js$/,
        use: [
          {
            //这里写 loader 名即可
            loader: 'a-loader',
            options: {
              /* ... */
            }
          },
          {
            loader: 'b-loader',
            options: {
              /* ... */
            }
          }
        ]
      }
    ]
  },
  resolveLoader: {
    // 告诉 webpack 该去那个目录下找 loader 模块
    modules: ['node_modules', path.resolve(__dirname, 'loaders')]
  }
}
```

> webpack 规定 use 数组中 loader 的执行顺序是从最后一个到第一个，
> loader 是一个模块导出函数，在正则匹配成功的时候调用，webpack 把文件数组传入进来，在 this 上下文可以访问 loader API

## 在 this 上下文可以访问 loader API

> loader 是一个模块导出函数，在正则匹配成功的时候调用，webpack 把文件数组传入进来，在 this 上下文可以访问 loader API

```js
this.context: 当前处理文件的所在目录，假如当前 Loader 处理的文件是 /src/main.js，则 this.context 就等于 /src。
this.resource: 当前处理文件的完整请求路径，包括 querystring，例如 /src/main.js?name=1。
this.resourcePath: 当前处理文件的路径，例如 /src/main.js。
this.resourceQuery: 当前处理文件的 querystring。
this.target: 等于 Webpack 配置中的 Target
this.loadModule: 但 Loader 在处理一个文件时，如果依赖其它文件的处理结果才能得出当前文件的结果时， 就可以通过 - - - this.loadModule(request: string, callback: function(err, source, sourceMap, module)) 去获得 request 对应文件的处理结果。
this.resolve: 像 require 语句一样获得指定文件的完整路径，使用方法为 resolve(context: string, request: string, callback: function(err, result: string))。
this.addDependency: 给当前处理文件添加其依赖的文件，以便再其依赖的文件发生变化时，会重新调用 Loader 处理该文件。使用方法为 addDependency(file: string)。
this.addContextDependency: 和 addDependency 类似，但 addContextDependency 是把整个目录加入到当前正在处理文件的依赖中。使用方法为 addContextDependency(directory: string)。
this.clearDependencies: 清除当前正在处理文件的所有依赖，使用方法为 clearDependencies()。
this.emitFile: 输出一个文件，使用方法为 emitFile(name: string, content: Buffer|string, sourceMap: {...})。
this.async: 返回一个回调函数，用于异步执行。
```

### 开发 Loader 原则

1. 单一职责
   一个 loader 只做一件事，这样不仅可以让 loader 的维护变得简单，还能让 loader 以不同的串联方式组合出符合场景需求的搭配。

2. 链式组合
   事实上串联组合中的 loader 并不一定要返回 JS 代码。只要下游的 loader 能有效处理上游 loader 的输出，那么上游的 loader 可以返回任意类型的模块。
3. 模块化
   保证 loader 是模块化的。loader 生成模块需要遵循和普通模块一样的设计原则。
4. 无状态
   在多次模块的转化之间，我们不应该在 loader 中保留状态。每个 loader 运行时应该确保与其他编译好的模块保持独立，同样也应该与前几个 loader 对相同模块的编译结果保持独立。
5. 使用 Loader 实用工具
   loader-utils 包
   schema-utils 包用 schema-utils 提供的工具，获取用于校验 options 的 JSON Schema 常量，从而校验 loader options

```js
import { getOptions } from 'loader-utils'
import { validateOptions } from 'schema-utils'

const schema = {
  type: object,
  properties: {
    test: {
      type: string
    }
  }
}

export default function(source) {
  const options = getOptions(this)

  validateOptions(schema, options, 'Example Loader')

  // 在这里写转换 source 的逻辑 ...
  return `export default ${JSON.stringify(source)}`
}
```
