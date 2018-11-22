# npm-react-demo
> 如何写一个react公用组件的npm包并上传下载可安装。


## 目录

- [生成NPM包](#生成NPM包)
- [压缩代码](#压缩代码)
- [发布NPM包](#发布NPM包)
- [拓展](#拓展)
- [注意事项](#注意事项)
- [关于](#关于)


## 生成NPM包

**生成package.json**

此说明只是讲怎么生成package.json，可直接用本项目中的package.json，替换你的信息即可。

打开cmd控制台，假如我们的项目要建立在D:/demo下
```js
d:
cd demo
npm init
```

回车之后按照要求输入即可。如我们的项目叫npm-react-demo
```js
package name:(name) npm-react-demo -----包的名字，输入之后回车，直接回车表示选中默认值"npm"
version:(1.0.0) 0.0.1beta-1        -----版本，默认1.0.0，以后每次发布版本要比先前的大，所以建议写最小单元
description:a demo for npm         -----描述，随便自己写
entry point:(index.js) ./index.js  -----入口文件，index.js
test command:                      -----测试指令，一般不填写
git repository:                    -----源码github地址，建议填写
keywords:                          -----关键词，用于npm网站的搜索分类索引
author: vivy(lizhiziyan@qq.com)    -----作者
license:(ISC)MIT                   -----许可证，填MIT吧
```
回车之后会看到配置，确认即生成package.json。

**常规配置**

上一步我们已经生成了package.json，接下来我们加入如下配置：

```
src                  // 源文件
|
lib                  // 压缩后的文件
├ .babelrc           // Babel设置
├ .editorconfig      // 编辑器配置
├ .gitignore         // 提交设置，比如node_modules这些不需要提交的写在这里
├ .LICENSE           // 许可证模板，可直接复制过去改改版本信息即可
└ README.md          // 说明文档，就是你看到的我现在这些说明就写在这里
```

**babel的使用**

书写代码时可以用ES5+，但是最后编译出来的必须要是ES5，具体的说明请看我另外项目https://github.com/liziyan/dt-antd#%E4%BB%A3%E7%A0%81%E8%AF%B4%E6%98%8E 这里有babel的说明，这个项目既然只是一个底层框架，是可以直接下载在里面写代码的，故而可以不更改我的配置。

如果你是按照npm安装来的package.json，则要执行安装babel的操作

```js
yarn add babel-cli
yarn add babel-loader
yarn add babel-preset-es2015
yarn add babel-preset-react
yarn add babel-preset-stage-0
```

**来个小例子试试**

我们在src里新建一个MyComponent.jsx文件，因为用到了react，所以要`yarn add react`安装react。

```js
import React from 'react';

const MyComponent = props=> {
  return <div className="my-component">
    props:
    <pre>{JSON.stringify(props, null, 2)}</pre>
  </div>
}

export default MyComponent;
```
我们的入口文件是index.js，虽然MyComponent.jsx写在src里，但index.js要指向/lib/，所以index.js要暴露组件入口地址：

```js
export { default as MyComponent } from './lib/MyComponent';
```
同时在src里新建index.js写入引用组件（原因将在下面`复制代码`里说明）
```js
export { default as MyComponent } from './MyComponent';
```

这样我们简单的例子就完成了。


### 压缩代码

我们的初衷是上传到npm的只有压缩后的`lib`代码，源码及依赖包都不上传(这样做可以让引入组件的项目打包后的js文件更小)。
所以要分两步走：
1.压缩组件
2.复制代码

**压缩组件**

在package.json里写入compile命令

```js
"scripts": {
  "compile": "rimraf lib && babel src --copy-files --source-maps --extensions .es6,.es,.jsx,.js --out-dir lib && node copy-files.js"  
},
```

`rimraf lib`表示删除现有的lib文件夹，要执行rimraf命令，就需要`yarn add rimraf`安装rimraf依赖
`babel src --copy-files --source-maps --extensions .es6,.es,.jsx,.js`对src文件夹里面的文件进行压缩
`--out-dir lib`新建lib文件夹，并将压缩后的文件放到里面

**复制代码**

一个完整的npm包，要包含`package.json`,`README.md`,`LICENSE`,`入口文件`。
而这所需的四个文件，有三个刚才我们已经完成过，直接拷贝到里面即可。
`入口文件`还记得上面的小例子中，在src文件里添加了index.js引入组件吗？这样直接压缩即可，不需要再去修改文件并复制。
上面的命令中`node copy-files.js`就是执行复制的文件，`copy-files.js`直接看代码，很浅显。


执行命令成功后，就可以看到项目中多了一个lib文件夹，里面含有我们要发布的所有文件。这时候就可以去发布npm包啦！


## 发布NPM包

1.到https://www.npmjs.com/ 注册一个账号
2.cmd到文件目录下执行代码

```js
npm adduser
Username: <你的用户名>
Password: <你的的登录密码>
Email: <你的注册邮箱>
```
3.发布版本

```js
yarn compile     <压缩代码>
npm publish lib  <发布压缩后的lib文件>
```

当当当，发布成功。接下来就可以执行`yarn add npm-react-demo`去安装使用了~

```js
import {MyComponent} from 'npm-react-demo';

<MyComponent />
```

## 拓展

我的例子里，有一个使用`antd`的例子，具体可看代码，相信聪明的你能举一反三的。


### 注意事项

这里有几个坑需要注意：

1.每次publish，package.json里的version都要比前一次大，不然会报403错误

2.组件里使用的依赖，比如`moment`，引入的项目中如果没有安装这个，则会在npm-react-demo里多一个`node_modules`，所以尽量避免用项目里没有的依赖。

3.如果npm使用的是淘宝镜像，可能会在adduser时登录不成功，解决方法：
```js
npm i nrm -g
nrm use npm
```
然后再次登录，登录之后输入`npm whoami`出现你的用户名则表示登录成功


## 关于

**本文使用的 babel 版本**

`./node_modules/.bin/babel --version` 6.4.5 (babel-core 6.4.5)

**LICENSE**

MIT

