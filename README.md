# artTemplate-3.0

新一代 javascript 模板引擎

##	目录

*	[特性](#特性)
*	[快速上手](#快速上手)
*	[模板语法](#模板语法)
*	[演示](#演示)
*	[下载](#下载)
*	[方法](#方法)
*	[NodeJS](#nodejs)
*	[使用预编译](#使用预编译)
*	[更新日志](#更新日志)
*	[授权协议](#授权协议)

##	特性

*	性能卓越，执行速度通常是 Mustache 与 tmpl 的 20 多倍（[性能测试](http://aui.github.com/artTemplate/test/test-speed.html)）
*	支持运行时调试，可精确定位异常模板所在语句（[演示](http://aui.github.io/artTemplate/demo/debug.html)）
*	对 NodeJS Express 友好支持
*	安全，默认对输出进行转义、在沙箱中运行编译后的代码（Node版本可以安全执行用户上传的模板）
*	支持``include``语句，可在浏览器端实现按路径加载模板
*	支持预编译，可将模板转换成为非常精简的 js 文件
*	支持所有流行的浏览器

## 快速上手


### 编写模板

使用一个``type="text/html"``的``script``标签存放模板：
	
	<script id="test" type="text/html">
	<h1>{{title}}</h1>
	<ul>
	    {{each list as value i}}
	        <li>索引 {{i + 1}} ：{{value}}</li>
	    {{/each}}
	</ul>
	</script>

### 渲染模板
	
	var data = {
		title: '标签',
		list: ['文艺', '博客', '摄影', '电影', '民谣', '旅行', '吉他']
	};
	var html = template('test', data);
	document.getElementById('content').innerHTML = html;


[演示](http://aui.github.com/artTemplate/demo/basic.html)

##	模板语法

有两个版本的模板语法可以选择。

###	简洁语法

推荐使用，语法简单实用，利于读写。

	{{if admin}}
		{{include 'admin_content'}}
		
		{{each list}}
			<div>{{$index}}. {{$value.user}}</div>
		{{/each}}
	{{/if}}
	
[查看语法文档](https://github.com/aui/artTemplate/wiki/syntax:simple)

###	原生语法
	
	<%if (admin){%>
		<%include('admin_content')%>
	
		<%for (var i=0;i<list.length;i++) {%>
			<div><%=i%>. <%=list[i].user%></div>
		<%}%>
	<%}%>

[查看语法文档](https://github.com/aui/artTemplate/wiki/syntax:native)

##	演示

*	[基本例子](http://aui.github.io/artTemplate/demo/basic.html)
*	[不转义HTML](http://aui.github.io/artTemplate/demo/no-escape.html)
*	[在javascript中存放模板](http://aui.github.io/artTemplate/demo/compile.html)
*	[嵌入子模板(include)](http://aui.github.io/artTemplate/demo/include.html)
*	[访问外部公用函数(辅助方法)](http://aui.github.io/artTemplate/demo/helper.html)
*	[错误调试](http://aui.github.io/artTemplate/demo/debug.html)
*	[print方法](http://aui.github.io/artTemplate/demo/print.html)

##	下载

* [template.js](https://raw.github.com/aui/artTemplate/master/dist/template.js) *(简洁语法版, 2.7kb)* 
* [template-native.js](https://raw.github.com/aui/artTemplate/master/dist/template-native.js) *(原生语法版, 2.3kb)*

## 方法

###	template(id, data)

根据 id 渲染模板。内部会根据``document.getElementById(id)``查找模板。

如果没有 data 参数，那么将返回一渲染函数。

###	template.``compile``(source, options)

将返回一个渲染函数。[演示](http://aui.github.com/artTemplate/demo/compile.html)

###	template.``render``(source, options)

将返回渲染结果。

###	template.``helper``(name, callback)

添加辅助方法。

模板无法读写外部对象，只能通过定义辅助方法的方式声明公用方法。例如扩展一个UBB替换方法：

```
template.helper('$ubb2html', function (content) {
	// 转义 HTML 字符
	content = template.helpers.$escape(content);
	// 解析 UBB 字符
    return content
    .replace(/\[b\]([^\[]*?)\[\/b\]/igm, '<b>$1</b>')
    .replace(/\[i\]([^\[]*?)\[\/i\]/igm, '<i>$1</i>')
    .replace(/\[u\]([^\[]*?)\[\/u\]/igm, '<u>$1</u>')
    .replace(/\[url=([^\]]*)\]([^\[]*?)\[\/url\]/igm, '<a href="$1">$2</a>')
    .replace(/\[img\]([^\[]*?)\[\/img\]/igm, '<img src="$1" />');
});
```
	
在模板中的使用方式：

	{{$ubb2html content}}
	
注意：引擎不会对辅助方法输出的 HTML 字符进行转义。
	
[演示](http://aui.github.com/artTemplate/demo/helper.html)

###	template.``config``(name, value)

更改引擎的默认配置。

    openTag: '<%'     // 逻辑语法开始标签
    closeTag: '%>'    // 逻辑语法结束标签
    escape: true      // 是否编码输出变量的 HTML 字符
    cache: true       // 是否开启缓存（依赖 options 的 filename 字段）
    compress: false   // 是否压缩输出
    parser: null      // 自定义语法格式器
	
##	使用预编译

可突破浏览器限制，让前端模板拥有后端模板一样的同步“文件”加载能力：

一、**按文件与目录组织模板**

```
template('tpl/home/main', data)
```

二、**模板支持引入子模板**


	{{include '../public/header'}}

###	基于预编译：

*	可将模板转换成为非常精简的 js 文件
*	使用同步模板加载接口
*	支持多种 js 模块输出：AMD、CMD、CommonJS
*	支持作为 GruntJS 插件构建
*	前端模板可共享给 NodeJS 执行
*	自动压缩打包模板

artTemplate 的预编译工具是一个子项目，请前往：[TmodJS](http://github.com/aui/tmodjs/)

##	NodeJS

###	安装

	$ npm install art-template -g
	
###	使用

	var template = require('art-template');
	var data = {list: ["aui", "test"]};
	
	var html = template(__dirname + '/index/main', data);

###	配置

NodeJS 版本新增了如下默认配置：

	// 指定模板目录
	template.config('path', '');
	
	// 指定模板后缀名
	template.config('extname', '.html');
	
	// 指定模板编码
	template.config('encoding', 'utf-8');
	
配置``path``指定模板目录可以缩短模板的路径，并且能够避免``include``语句越级访问任意路径引发安全隐患，例如：
	
	template.config('path', __dirname);
	var html = template('index/main', data)
	
###	NodeJS + Express

	app.register('.html', require('art-template'));
	
> 若使用 js 原生语法作为模板语法，请改用 ``require('art-template/node/template-native.js')``

##	升级参考

为了适配 NodeJS express，artTemplate v3.0.0 接口有调整。

###	接口变更

1.	默认使用简洁语法
2. ``template.render()``方法的第一个参数不再是 id，而是模板字符串
3. 使用新的配置接口``template.config()``并且字段名有修改
4. ``template.compile()``方法不支持 id 参数
5. helper 方法不再强制原文输出，是否编码取决于模板语句
6. ``template.helpers`` 中的``$string``、``$escape``、``$each``已迁移到``template.utils``中

###	升级方法

1. 如果想继续使用 js 原生语法作为模板语言，请使用 [template-native.js](https://raw.github.com/aui/artTemplate/master/dist/template-native.js)
2. 查找项目```template.render```替换为```template```
3. 使用``template.config(name, value)``来替换以前的配置

## 更新日志

### v3.0.0

1. 提供 NodeJS 专属版本，支持使用路径加载模板，并且模板的``include``语句也支持相对路径
2. 适配 [express](http://expressjs.com) 框架
3. 内置``print``语句支持传入多个参数
4. 支持全局缓存配置
5. 简洁语法版支持管道风格的 helper 调用，例如：``{{time | dateFormat:'yyyy年 MM月 dd日 hh:mm:ss'}}``

当前版本接口有调整，请阅读 [升级参考](#升级参考)

> artTemplate 预编译工具 [TmodJS](https://github.com/aui/tmodjs) 已更新

###	v2.0.4

1.	修复低版本安卓浏览器编译后可能产生语法错误的问题（因为此版本浏览器 js 引擎存在 BUG）

###	v2.0.3

1.	优化辅助方法性能
2.	NodeJS 用户可以通过 npm 获取 artTemplate：``$ npm install art-template -g``
3.	不转义输出语句推荐使用``<%=#value%>``（兼容 v2.0.3 版本之前使用的``<%==value%>``），而简版语法则可以使用``{{#value}}``
4.	提供简版语法的合并版本 dist/[template-simple.js](https://raw.github.com/aui/artTemplate/master/dist/template-simple.js)

### v2.0.2

1.	优化自定义语法扩展，减少体积
2.	[重要]为了最大化兼容第三方库，自定义语法扩展默认界定符修改为``{{``与``}}``。
3.	修复合并工具的BUG [#25](https://github.com/aui/artTemplate/issues/25)
4.	公开了内部缓存，可以通过``template.cache``访问到编译后的函数
5.	公开了辅助方法缓存，可以通过``template.helpers``访问到
6.	优化了调试信息

### v2.0.1

1.	修复模板变量静态分析的[BUG](https://github.com/aui/artTemplate/pull/22)

### v2.0 release

1.	~~编译工具更名为 atc，成为 artTemplate 的子项目单独维护：<https://github.com/cdc-im/atc>~~

### v2.0 beta5

1. 修复编译工具可能存在重复依赖的问题。感谢 @warmhug
2. 修复预编译``include``内部实现可能产生上下文不一致的问题。感谢 @warmhug
3. 编译工具支持使用拖拽文件进行单独编译

### v2.0 beta4

1. 修复编译工具在压缩模板可能导致 HTML 意外截断的问题。感谢 @warmhug
2. 完善编译工具对``include``支持支持，可以支持不同目录之间模板嵌套
3. 修复编译工具没能正确处理自定义语法插件的辅助方法

### v2.0 beta1

1.	对非 String、Number 类型的数据不输出，而 Function 类型求值后输出。
2.	默认对 html 进行转义输出，原文输出可使用``<%==value%>``（备注：v2.0.3 推荐使用``<%=#value%>``），也可以关闭默认的转义功能``template.defaults.escape = false``。
3.	增加批处理工具支持把模板编译成不依赖模板引擎的 js 文件，可通过 RequireJS、SeaJS 等模块加载器进行异步加载。

## 授权协议

Released under the MIT, BSD, and GPL Licenses

============

[所有演示例子](http://aui.github.com/artTemplate/demo/index.html) | [引擎原理](http://cdc.tencent.com/?p=5723)

© tencent.com
