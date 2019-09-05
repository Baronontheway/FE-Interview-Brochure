# HTML篇
## Q1：浏览器解析渲染页面过程

`腾讯一面`

A1：HTML解析构建DOM->CSS解析构建CSSOM树->根据DOM树和CSSOM树构建render树->根据render树进行布局渲染render layer->根据计算的布局信息进行绘制
### Q1.1：布局渲染的过程发生的回流和重绘区别&减少回流次数的方法
A1.1：区别：
- 回流指当前窗口发生改变，发生滚动操作，或者元素的位置大小相关属性被更新时会触发布局过程，发生在render树
- 重绘指当前视觉样式属性被更新时触发的绘制过程，发生在渲染层render layer

减少回流次数的方法：
- 1）避免一条一条的修改DOM样式，而是修改className或者style.classText
- 2）对元素进行一个复杂的操作，可以先隐藏它，操作完成后在显示
- 3）在需要经常获取那些引起浏览器回流的属性值时，要缓存到变量中
- 4）不使用table布局，一个小的改动可能就会引起整个table重新布局
- 5）在内存中多次操作节点，完成后在添加到文档中

# CSS篇

# JavaScript篇

`腾讯二面`

## Q1：跨域的方式有哪些？jsonp的原理以及服务端如何处理

> 分析:谈到跨域，首先想到的就是我们为什么需要跨域？要实现跨域，就要知道跨域的方式有哪些？你最常用那种方式跨域？原理是什么？浏览器端如何做？服务端又该如何处理？接下来逐一回答
### Q1.1：为什么需要跨域？同源策略拦截客户端请求还是服务器响应？
A1.1：之所以需要跨域，是因为浏览器同源策略的约束，面对不同源的请求，我们无法完成，这时候就需要用到跨域。同源策略拦截的是跨源请求，原因：CORS缺少`Access-Control-Allow-Origin`头
### Q1.2：跨域的方式有哪些
A1.2：JSONP、proxy代理、CORS、XDR
### Q1.3：JSONP的原理是什么，缺点是什么，浏览器端如何做？
A1.3：JSONP主要是因为`script`标签的`src`属性不被同源策略所约束，同时在没有阻塞的情况下资源加载到页面后会立即执行

缺点：
- JSONP只支持get请求而不支持post请求，如果想传给后台一个json格式的数据,此时问题就来了,浏览器会报一个http状态码415错误，请求格式不正确
- JSONP本质是一种代码注入，存在安全问题
### Q1.4：JSONP服务器端如何处理？
A1.4：实际项目中JSONP通常用来获取`json`格式的数据，这时候前后端通常约定一个参数`callback`，该参数的值，就是处理返回数据的函数名称
```js
var f = function(data){
    alert(data.name)
}

var _script = document.createElement('script');
_script.type = 'text/javascript'
_script.src = 'http://localhost:8888/jsonp?callback=f'
document.head.appendCild(_script);
```

```js
//node处理
var query = _scr.query;
var params = qs.parse(query);
var f = '';

f = params.callback;

res.writeHead(200,{'Content-Type','text/javascript'})
res.write(f + "({name:'hello world'})")
res.end
```

```php
//php处理(注意输出格式)
$data = array(
    'rand' => $_GET['random'],
    'msg' => 'Success'
);

echo $_GET['callback'].'('.json_encode($data).')';
```
### 1.5：（扩展）cors
cors是一种现代浏览器支持跨域资源请求的一种方式，当使用XMLHttpRequest发送请求时，浏览器发现不符合同源策略，会给请求加一个请求头`Origin`,后台进行一系列处理，如果确定接受请求则在返回结果中加入一个响应头：`Access-Control-Allow-Origin`;浏览器判断该响应头是否包含`Origin`的值,如果有则浏览器会处理响应,我们就可以拿到响应数据，如果不包含浏览器会直接驳回，这时我们就无法拿到响应数据
```js
//node处理
if(req.headers.origin){
    res.wirteHead(200,{
        'Content-type':'text/html;charset=UTF-8',
        'Access-Control-Allow-Origin':'*', // *通配，或者安全考虑替换为指定域名
    });
    res.write('cors');
    res.end();
}
```