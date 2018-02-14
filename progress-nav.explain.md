## progress-nav

* 进度导航 http://lab.hakim.se/progress-nav/

[github source](https://github.com/hakimel/css/tree/master/progress-nav)

[codepen.io - 复制例子](https://codepen.io/china-boy/pen/NyvYrR)

---

- [index.html {explain}](./hakimel-css/progress-nav/index.html)

- [script.js {explain}](./hakimel-css/progress-nav/script.js)

- [style.css {explain}](./hakimel-css/progress-nav/style.css)

---

## index-html

我们先看html 文件

### html-head

``` html
<!DOCTYPE html> 
<!-- 编译声明 -->
<html lang="en">
<!-- 国家语言 -->
	<head>

		<meta charset="UTF-8">
<!-- 字符 -->
		<title>Progress Nav</title>
<!-- 标签页显示 -->
        <meta name="description" content="">
<!-- 描述 -->
		<meta name="author" content="Hakim El Hattab">
<!-- 作者 -->
		<meta name="viewport" content="width=700, user-scalable=no" />
<!-- 设备视窗 -->
        <link href="https://fonts.googleapis.com/css?family=Frank+Ruhl+Libre|Roboto" rel="stylesheet">
<!-- 字体加载 -->
        <link href="normalize.css" rel="stylesheet" type="text/css" />
<!-- normalize.css 重定义 css 属性, 给予浏览器一个标准的开始 -->
        <link href="style.css" rel="stylesheet" type="text/css" />
<!-- 导入自己定义的 css -->

	</head>
```

### html-body-nav

``` html
	<body>

		<nav class="toc">
			<ul>
				<li><a href="#intro">Intro</a></li>
				<li>
					<a href="#dev">Developer Mode</a>
					<ul>
						<li><a href="#dev-edit-html">Edit HTML</a></li>
						<li><a href="#dev-element-classes">Element Classes</a></li>
						<li><a href="#dev-slide-classes">Slide Classes</a></li>
						<li><a href="#dev-export-html">Export HTML</a></li>
					</ul>
				</li>
				<li>
					<a href="#css">CSS Editor</a>
					<ul>
						<li><a href="#css-fonts">Custom Fonts</a></li>
						<li><a href="#css-developer-mode">Developer Mode</a></li>
						<li><a href="#css-examples">Examples</a></li>
					</ul>
				</li>
			</ul>
			<svg class="toc-marker" width="200" height="200" xmlns="http://www.w3.org/2000/svg">
				<path stroke="#444" stroke-width="3" fill="transparent" stroke-dasharray="0, 0, 0, 1000" stroke-linecap="round" stroke-linejoin="round" transform="translate(-0.5, -0.5)" />
			</svg>
		</nav>
```

### html-script

代码 224

``` html
<script src="script.js"></script>
```

---
## script-js

代码 1

> 浏览器加载后运行

``` js
window.onload = function() {
    // ...
}
```

代码 3-12

> 获取便签列表元素+path-svg元素-->[html-nav-element](#html-body-nav)

``` js
    var toc = document.querySelector( '.toc' ); // 便签列表元素
	var tocPath = document.querySelector( '.toc-marker path' ); // path-svg元素
	var tocItems;

	// Factor of screen size that the element must cross
	// before it's considered visible
	var TOP_MARGIN = 0.1,
		BOTTOM_MARGIN = 0.2;

	var pathLength;
```

代码 14-15

> 浏览器-滚动+浏览窗口重载 触发函数

``` js
	window.addEventListener( 'resize', drawPath, false ); //窗口重载
	window.addEventListener( 'scroll', sync, false ); //滚动
```

代码 17-78

> 整理-便签列表数据

``` js
drawPath(); // 先运行一次

	function drawPath() {

		tocItems = [].slice.call( toc.querySelectorAll( 'li' ) );

		// Cache element references and measurements
		tocItems = tocItems.map( function( item ) {
			var anchor = item.querySelector( 'a' );
			var target = document.getElementById( anchor.getAttribute( 'href' ).slice( 1 ) );

			return {
				listItem: item,
				anchor: anchor,
				target: target
			};
		} );

		// Remove missing targets
		tocItems = tocItems.filter( function( item ) {
			return !!item.target;
		} );

		var path = [];
		var pathIndent;

		tocItems.forEach( function( item, i ) {

			var x = item.anchor.offsetLeft - 5,
				y = item.anchor.offsetTop,
				height = item.anchor.offsetHeight;

			if( i === 0 ) {
				path.push( 'M', x, y, 'L', x, y + height );
				item.pathStart = 0;
			}
			else {
				// Draw an additional line when there's a change in
				// indent levels
				if( pathIndent !== x ) path.push( 'L', pathIndent, y );

				path.push( 'L', x, y );

				// Set the current path so that we can measure it
				tocPath.setAttribute( 'd', path.join( ' ' ) );
				item.pathStart = tocPath.getTotalLength() || 0;

				path.push( 'L', x, y + height );
			}

			pathIndent = x;

			tocPath.setAttribute( 'd', path.join( ' ' ) );
			item.pathEnd = tocPath.getTotalLength();

		} );

		pathLength = tocPath.getTotalLength();

		sync();

	}
```

代码 80-120

> 随着滚动 帮 便签列表加减-class + svg-加减长度

``` js
	function sync() {

		var windowHeight = window.innerHeight;

		var pathStart = pathLength,
			pathEnd = 0;

		var visibleItems = 0;

		tocItems.forEach( function( item ) {

			var targetBounds = item.target.getBoundingClientRect();

			if( targetBounds.bottom > windowHeight * TOP_MARGIN && targetBounds.top < windowHeight * ( 1 - BOTTOM_MARGIN ) ) {
				pathStart = Math.min( item.pathStart, pathStart );
				pathEnd = Math.max( item.pathEnd, pathEnd );

				visibleItems += 1;

				item.listItem.classList.add( 'visible' );
			}
			else {
				item.listItem.classList.remove( 'visible' );
			}

		} );

		// Specify the visible path or hide the path altogether
		// if there are no visible items
		if( visibleItems > 0 && pathStart < pathEnd ) {
			tocPath.setAttribute( 'stroke-dashoffset', '1' );
			tocPath.setAttribute( 'stroke-dasharray', '1, '+ pathStart +', '+ ( pathEnd - pathStart ) +', ' + pathLength );
			tocPath.setAttribute( 'opacity', 1 );
		}
		else {
			tocPath.setAttribute( 'opacity', 0 );
		}

	}

};
```

- add( 'visible' ) + remove( 'visible' )

加减class [css-visible](#css-visible-a)

---


## style-css

``` css
* {
  box-sizing: border-box; }

body {
  width: 100vw;
  height: 100vh;
  overflow: auto; }

body {
  padding: 2em 2em 2em 17em;
  font-size: 16px;
  line-height: 1.6;
  font-family: 'Roboto', sans-serif; }

.toc {
  position: fixed;
  left: 3em;
  top: 5em;
  padding: 1em;
  width: 14em;
  line-height: 2; }
  .toc ul {
    list-style: none;
    padding: 0;
    margin: 0; }
  .toc ul ul {
    padding-left: 2em; }
  .toc li a {
    display: inline-block;
    color: #aaa;
    text-decoration: none;
    -webkit-transition: all 0.3s cubic-bezier(0.23, 1, 0.32, 1);
            transition: all 0.3s cubic-bezier(0.23, 1, 0.32, 1); }

```

### css-visible-a

```css
  .toc li.visible > a {
    color: #111;
    -webkit-transform: translate(5px);
        -ms-transform: translate(5px);
            transform: translate(5px); }

```

``` css
.toc-marker {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -1; }
  .toc-marker path {
    -webkit-transition: all 0.3s ease;
            transition: all 0.3s ease; }

.contents {
  padding: 1em;
  max-width: 800px;
  font-size: 1.2em;
  font-family: 'Frank Ruhl Libre', sans-serif; }
  .contents img {
    max-width: 100%; }
  .contents .code-block {
    white-space: pre;
    overflow: auto;
    max-width: 100%; }
    .contents .code-block code {
      display: block;
      background-color: #f9f9f9;
      padding: 10px; }
  .contents .code-inline {
    background-color: #f9f9f9;
    padding: 4px; }
  .contents h2
h3 {
    padding-top: 1em; }
  .contents h2 {
    margin-top: 1.2em; }

@media screen and (max-width: 1200px) {
  body {
    font-size: 14px; } }

```
