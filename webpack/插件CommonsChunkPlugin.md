### 概念:

是一个可选的用于建立一个独立文件(又称作 chunk)的功能，这个文件包括多个入口 chunk 的公共模块。通过将公共模块拆出来，最终合成的文件能够在最开始的时候加载一次，便存起来到缓存中供后续使用。这个带来速度上的提升，因为浏览器会迅速将公共的代码从缓存中取出来，而不是每次访问一个新页面时，再去加载一个更大的文件。

### 配置

```
{
  name: string,
  names: string[],
  // 这是 common chunk 的名称。已经存在的 chunk 可以通过传入一个已存在的 chunk 名称而被选择。
  // 如果一个字符串数组被传入，这相当于插件针对每个 chunk 名被多次调用
  // 如果该选项被忽略，同时 `options.async` 或者 `options.children` 被设置，所有的 chunk 都会被使用，否则 `options.filename` 会用于作为 chunk 名。

  filename: string,
  // common chunk 的文件名模板。可以包含与 `output.filename` 相同的占位符。
  // 如果被忽略，原本的文件名不会被修改(通常是 `output.filename` 或者 `output.chunkFilename`)

  minChunks: number|Infinity|function(module, count) -> boolean,
  // 在传入  公共chunk(commons chunk) 之前所需要包含的最少数量的 chunks 。
  // 数量必须大于等于2，或者少于等于 chunks的数量
  // 传入 `Infinity` 会马上生成 公共chunk，但里面没有模块。
  // 你可以传入一个 `function` ，以添加定制的逻辑（默认是 chunk 的数量）

  chunks: string[],
  // 通过 chunk name 去选择 chunks 的来源。chunk 必须是  公共chunk 的子模块。
  // 如果被忽略，所有的，所有的 入口chunk (entry chunk) 都会被选择。


  children: boolean,
  // 如果设置为 `true`，所有  公共chunk 的子模块都会被选择

  async: boolean|string,
  // 如果设置为 `true`，一个异步的  公共chunk 会作为 `options.name` 的子模块，和 `options.chunks` 的兄弟模块被创建。
  // 它会与 `options.chunks` 并行被加载。可以通过提供想要的字符串，而不是 `true` 来对输出的文件进行更换名称。

  minSize: number,
  // 在 公共chunk 被创建立之前，所有 公共模块 (common module) 的最少大小。
}

```

#### 公共chunk用于入口chunk(entry chunk)

- 生成一个额外的 chunk 包含入口chunk 的公共模块。

```
new webpack.optimize.CommonsChunkPlugin({
  name: "commons",
  // ( 公共chunk(commnons chunk) 的名称)

  filename: "commons.js",
  // ( 公共chunk 的文件名)
  // minChunks: 3,
  // (模块必须被3个 入口chunk 共享)
  // chunks: ["pageA", "pageB"],
  // (只使用这些 入口chunk)
})

```
你必须在 入口chunk 之前加载生成的这个 公共chunk:

```
<script src="commons.js" charset="utf-8"></script>
<script src="entry.bundle.js" charset="utf-8"></script>
```

#### 第三方库 chunk

将你的代码拆分成公共代码和应用代码。

```
 entry: {
    app: './src/main.js',
    vendors: ['vue', 'vue-router']
  }
// app: 应用程序入口
// vendors: 公共库入口
```

主要说下vendors的作用:

```
vendors: 主要是指一些经常需要被别的模块引用的第三方库。
以vue-route为例，被很多模块引用,我们加载这些模块时候,就要去加载一次vue-router,那么我们就可以把vue-route抽取出来打包成一个vendors bundle.
这样就只需要打包一次.这个过程是由CommonsChunkPlugin插件从「应用程序 bundle」中提取 vendor 引用(vendor reference) 到 vendor bundle.
使用 CommonsChunkPlugin 将公共库(vendor)和应用程序代码分离开来，并创建一个显式的 vendor chunk 以防止它频繁更改。 当使用 CommonsChunkPlugin 时，运行时代码会被移动到最后一个公共入口(entry)。
```
见我项目中的配置:

```
 // split vendor js into its own file
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks: function (module, count) {
        // any required modules inside node_modules are extracted to vendor
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
        //
      }
    }),
    // extract webpack runtime and module manifest to its own file in order to
    // prevent vendor hash from being updated whenever app bundle is updated.
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      chunks: ['vendor']
    })
    // 主要涉及缓存:为了能够长期缓存 webpack 生成的静态资源:
    // 将 webpack mainfest 提取到一个单独的文件中去。
```

#### 关于给 minChunks 配置传入函数

给 minChunks 传入一个函数。这个函数会被 CommonsChunkPlugin 插件回调，并且调用函数时会传入 module 和 count 参数。

module 参数代表每个 chunks 里的模块，这些 chunks 是你通过 name/names 参数传入的.module有两个重要的属性.

- module.context: 存储文件的目录,例如: '/my_project/node_modules/xxx-dependency'
- module.resource: 正在处理的文件的名称。例如： '/my_project/node_modules/xxx-dependency/index.js'

count 参数表示 module 被使用的 chunk 数量。

这里又涉及一个对静态资源长期缓存的问题,为了去避免 公共chunk改变,我们需要[chunk-manifest-webpack-plugin插件](https://github.com/soundcloud/chunk-manifest-webpack-plugin).

demo配置:


```
var path = require("path");
var webpack = require("webpack");
var ChunkManifestPlugin = require("chunk-manifest-webpack-plugin");
var WebpackChunkHash = require("webpack-chunk-hash");

module.exports = {
  entry: {
    vendor: "./src/vendor.js", // vendor reference file(s)
    main: "./src/index.js" // application code
  },
  output: {
    path: path.join(__dirname, "build"),
    filename: "[name].[chunkhash].js", // 根据文件内容生成哈希值

    chunkFilename: "[name].[chunkhash].js"
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: ["vendor", "manifest"], // vendor libs + extracted manifest
      minChunks: Infinity,
    }),
    new webpack.HashedModuleIdsPlugin(),
    new WebpackChunkHash(),
    new ChunkManifestPlugin({
      filename: "chunk-manifest.json",
      manifestVariable: "webpackManifest",
      inlineManifest: true
    })
  ]
};

```

[chunkhash] webpack 允许你根据文件内容生成哈希值.也就是说如果文件内容在两次构建之间没有变化，就生成相同的文件名.

不要在开发环境下使用 [chunkhash]，因为这会增加编译时间。将开发和生产模式的配置分开，并在开发模式中使用 [name].js 的文件名， 在生产模式中使用 [name].[chunkhash].js 文件名。


#### 将公共模块打包进父 chunk

使用代码拆分功能，一个 chunk 的多个子 chunk 会有一些公共的依赖。为了防止重复依赖打包，可以将这些公共模块移入父 chunk。这会减少总体的大小，但会对首次加载时间产生不良影响。如果预期到用户需要下载许多兄弟 chunks（例如，入口 trunk 的子 chunk），那这对改善加载时间将非常有用。

```
new webpack.optimize.CommonsChunkPlugin({
  // names: ["app", "subPageA"]
  // (选择 chunks，或者忽略该项设置以选择全部 chunks)

  children: true,
  // (选择所有被选 chunks 的子 chunks)

  // minChunks: 3,
  // (在提取之前需要至少三个子 chunk 共享这个模块)
})

```

#### 异步公共chunk

与上面的类似，但是并非将公共模块移动到父 chunk（增加首屏初始加载时间），而是使用新的异步加载的额外公共chunk。当下载额外的 chunk 时，它将自动并行下载。

```
new webpack.optimize.CommonsChunkPlugin({
  // names: ["app", "subPageA"]
 // (选择 chunks，或者忽略该项设置以选择全部 chunks)

  children: true,
  // (选择所有被选 chunks 的子 chunks)

  async: true,
  // (创建一个异步 公共chunk)

  // minChunks: 3,
  // (在提取之前需要至少三个子 chunk 共享这个模块)
})

```
