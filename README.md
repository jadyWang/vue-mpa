vue+webpack 多页应用框架【可集成单页应用】

date: 2017-01-07 | author: zhoujiqiu

前言

了解了Vue一般都会去用Vue-cli入门，这是一个构建SPA的脚手架，查看其build的项目，可以看到它是将所有的模块都输出到一个build.js中，有时候会看到这个js文件特别大，有好几兆，然而当一个项目足够复杂时，SPA恐怕不再适合使用了，用户不可能访问你的网页的时候一下子下载一个几兆的文件，特别对于手机用户，可能用户只看了网站中的一篇文章，这也会导致网页加载慢，这是不可取的。所以应该将网站划分成若干模块。于是就有了本框架（一个用于构建多页面的脚手架）

为什么要搭建vue多页（mpa）框架

项目经常遇到的问题：单页应用spa打包文件太大的问题； 团队协助：单页开发对前端技能水平要求比较高，新手上手比较难,而且单页面在团队合作开发上比较低效； 业务场景需要：同一个项目多个H5应用；把多个应用做成一个单页体积会很大，而且难于维护； 因为各页之间相对独立，少了各种变量污染的担忧；

使用方法

# 安装
npm install

# 调试环境 serve with hot reload at http://localhost:8089/
npm run dev

# 生产环境 build for production with minification
npm run build
运行

# 多页demo地址
http://localhost:8089/multi/list/index.html

# 单页demo地址
http://localhost:8089/single/home/index.html
本地默认访问端口为8089，需要更改的童鞋请到项目根目录文件config.js修改。

目录结构

xwtoon-mpa
    |---build webpack配置文件
    |---dist 生产环境打包
    |---src  开发目录
        |---assets 静态资源
            |---common.css  公共css
            |---common.js  公共js
            |---font/  字体图标
            |---img/   公共图片
        |---components 组件库
              |---toast.vue  消息组件
              |---dialog.vue 弹框组件
        |---module 所有业务模块
              |---multi 多页应用模块
                  |--- list 列表页
                        |--- app.vue
                        |--- index.html
                        |--- index.js
                  |--- detail 详情页
                        |--- app.vue
                        |--- index.html
                        |--- index.js
              |---single 单页应用模块
                  |--- home 
                        |--- app.vue
                        |--- index.html
                        |--- index.js
                        |--- routers.js
                        |--- detail.vue
                        |--- list.vue

从目录结构上，各种组件、页面模块、公共资源等都按类新建了文件夹，方便我们储存文件。 我们所有的文件，最主要都是放在module文件夹里； 在module里下级文件夹，一个文件夹就是一个html，jsvue template 都统一放在当前文件夹里，当然你也可以继续放其他的资源，例如css、图片等，webpack会打包到当前页面里。 如果项目不需要这个页面了，可以直接把这个文件夹直接删除掉，干净项目，干活也开心。 像以前我们传统开发项目，所有的图片都习惯放在images里，当项目有改动时，有些图片等资源用不上了，但还占着坑位，虽然现在的硬件容量大到惊人，但我们应该还是要养到一个良好的习惯。 当前页面的开发在app.vue里，打开后你就会看到很熟悉的<template>、<script>、<style scoped>了。 注意：请保持module下的各模块的目录结构一致；

webpack主要配置说明

1、在webpack中配置项目公用的JS文件，并提取打包到指定目录； 由于是多页应用，必然会有多个入口，本架构会将module下的所有js文件作为入口文件；所以公共文件的提取需要在入口文件处单独处理，配置如下：

# 入口文件
var entries = getEntry('./src/module/**/**/*.js');

# 指定公共JS资源，并打包到指定的目录
entries['static/common/vendors'] = ['vue','vue-resource','n-zepto','./src/assets/common.js'];
2、配置js(html)输出路径 如下文件输出路径请根据项目结构需要做相应修改。

# 输出路径
glob.sync(globPath).forEach(function (entry) {
  basename = path.basename(entry, path.extname(entry));
  //过滤非入口文件的JS：routers.js和vuex/*.js
  if(basename.indexOf('routers') !== -1 || entry.indexOf('vuex/') !== -1) return; 
  // 原路径：entry = ‘src/module/multi/list/index.js’
  tmp = entry.split('/').splice(-3);
  // 分解后：tmp = [multi,list,index.js]
  pathname =  tmp.slice(0, 2).join('/') + '/' + basename; 
  // 正确输出js和html的路径: pathname = 'multi/list/index'
  entries[pathname] = entry;
});
3、配置静态资源输出 将静态资源输出到static文件夹

# css
{
  test: /\.scss$/,
  loader: ExtractTextPlugin.extract('style-loader', 'css-loader!sass-loader')
},
# 图片
{
  test: /\.(png|jpe?g|gif)(\?.*)?$/,
  loader: 'url',
  query: {
    limit: 8192,
    name: 'static/images/[name].[hash:7].[ext]'
  }
},
# 图标字体
{
  test: /\.((ttf|eot|woff|svg)(\?t=[0-9]\.[0-9]\.[0-9]))|(ttf|eot|woff|svg)\??.*$/,
  loader: 'url-loader?name=static/font/[name].[ext]'
}
集成单页（spa）应用架构

由于该mpa架构页面跳转会刷新页面，导致在某些业务场景下，数据传递和vue组件之间的通信会很不方便，所以在框架中集成了单页面应用架构；

# 单页面应用结构
  |--- home  某个应用模块
      |--- index.html  入口html
      |--- index.js    入口js,import vue-router
      |--- app.vue     router-view 路由容器
      |--- routers.js  路由配置，通过resolve参数实现异步加载；每个路由会生成自己业务的js
      |--- detail.vue     路由1:详情模块
      |--- list.vue    路由2:列表模块
# 单页面路由打包，将不是入口文件的vue-router单独打包；统一放在static/chunks目录
output: {
    path: config.build.assetsRoot,
    publicPath: config.build.assetsPublicPath,
    filename: '[name].js',
    chunkFilename: 'static/chunks/[name].js?[chunkhash]'
}
