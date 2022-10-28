# 一文看懂Htmx

## 引言

作为一个前端开发者，使用React这类前端框架也有一段时间了。在工作中使用某些工具时间久了，对相关新技术的就麻木了，不会主动思考诸如“为什么代码是这样写的？”，“这是最自然高效的开发方式吗？”这类的问题。

偶然在JavaScript Weekly上看到了一篇对比`React`和`Htmx`的[文章](https://htmx.org/essays/a-real-world-react-to-htmx-port/)，生词出现了，什么是Htmx？它为什么要与React对比？它与JSX有什么关系？在本文中，我基于一些资料的学习试图回答上述问题。

![htmx](../images/htmx.png)

## HATEOAS
简单来说，Htmx提供了一种通过**HTML**（而不是**JavaScript**）访问各种浏览器特性的方式。

这里的`X`应该是extension，是“扩展”的意义。JSX对JavaScript的功能进行了扩展（虽然只是语法级的），Htmx对HTML的功能进行了扩展。

那么到底扩展了什么？回答这个问题，需要先了解[**HATEOAS**](https://htmx.org/essays/hateoas/)的概念。这个词是`Hypermedia as the Engine of Application State`的缩写，即**超媒体驱动应用状态**。

我们都熟悉超链接（Hyperlink）和超文本（Hypertext）的概念，[超媒体（Hypermedia）](https://en.wikipedia.org/wiki/Hypermedia)是对这些概念的扩展，其实就是我们在HTML页面上常见的各类多媒体资源的统称，包括视频、音频、图片、文本、链接等等。

我们可以回想下，浏览器端的Web应用程序是如何管理**应用状态**的。比如一个后台用户管理单页面应用（SPA)，添加、编辑、删除、排序这类功能是如何在页面上变化和呈现的？一般我们会通过**JavaScript**来实现。

监听控件的UI事件后，需要JavaScript添加响应代码并定义后续操作。需要请求或者提交数据时，也需要JavaScript发起REST请求，处理返回的数据，最后渲染到DOM。这里一个隐含的要求是：请求者需要知道**如何使用返回的数据以切换应用状态**。

比如一个Server返回的用户数据的JSON对象如下：
```json
{
    "account": {
        "id": 12345,
        "name": "jtzcode",
        "status": "active",
        "permissions": ["view", "edit"],
        "action": "/users/12345/save"
    }
}
```

这份数据到达后，我们需要通过JavaScript或者CSS等技术将用户的信息呈现到UI，也就是**光有这些数据，我们仍然不知道怎样呈现UI**，根本原因是**数据中没有呈现的逻辑**。在上面例子中，我们不知道UI怎样通过`status`的值来呈现，也不知道如何使用`action`字段里的链接。

那我们能不能直接让server返回下一个状态的HTML结构呢？这就是HATEOAS的理念。每次应用程序状态的变化，不是靠数据驱动，而是**页面结构和元素（超媒体）直接驱动**。比如上面的例子可能得到类似这样的响应结果：
```html
<div>
    <form method="post" action="/users/12345/save">
        <label>Name: </label><span>jtzcode</span>
        <img src="/users/avarta/12345.png"/>
        <button>Edit</button>
        <button type="submit">Save</button>
    </form>
</div>
```
这个响应结果可以直接嵌入当前HTML页面的某个位置，让应用程序进入下一个状态，即直接通过**超媒体**驱动，省去了大量JavaScript的操作。

## 开始上手
这里有个官方提供的例子，可以直观感受一下用法：
```html
<button hx-post="/clicked"
    hx-trigger="click"
    hx-target="#parent-div"
    hx-swap="outerHTML"
>
    Click Me!
</button>
```
Htmx更符合**声明式**的编程语法，有点**所见即所得**的意思。如上面的声明告诉Htmx：
> 当用户`单击`按钮，向`/click`的路由发送`POST`请求，并把返回的内容设置到`#parent-div`表示的容器中。

看起来这种用法的确可以减少前端开发量，代码简单明确，也确实有商业项目从React迁移到Htmx实现的[案例](https://htmx.org/essays/a-real-world-react-to-htmx-port/)。不过我觉得在真正在自己的项目里使用前，至少还应该考虑：

1. 应用规模。上述成功案例只是一个中小规模的应用，对于大型项目，Htmx能否满足所有业务需求仍需评估。
2. API的实现。由于返回结果变为HTML格式，那么API端可能有更多的渲染工作要做。这方面的技术积累自己是否胜任需要考虑。
3. 重构成本。如果是已有的项目迁移到htmx，要根据项目规模、团队结构、业务逻辑评估迁移的ROI。

## 延伸话题

你可能说Web API返回的数据是什么格式和结构有好坏之分吗（JSON vs Hypermedia）？其实对REST API的设计有一个[理查森成熟度模型]（https://en.wikipedia.org/wiki/Richardson_Maturity_Model），这个模型的初衷是找到REST约束与其他类型Web服务的联系。它将设计分层四个层级：**The Swamp of POX，Resources，HTTP Verbs，Hypermedia Controls**，本文的HATEOAS即来源于最后一个层级。

另外，采用Htmx的开发方法也符合[行为局部性原则](https://htmx.org/essays/locality-of-behaviour/)（Locality of Behavior，LoB），即开发者应该尽量让一段代码的功能看起来直观（不要让功能定义各处分散）。比如：
```html
<button hx-get="/get">Click Me</button>
```
和
```javascript
$("#btn").on("click", function(){
    fetch(...)
});
<button id="btn">Click Me</button>
```
第一段Htmx代码直观地看出点击按钮的行为（发起`/get`请求），而后一段代码声明按钮控件在一个文件，定义单击事件行为的代码却在另一个文件，就不满足行为局部性约束。

## Takeaways
综上，我们可以概括使用Htmx开发的要点：
1. 使用声明式的语法（而不是JavaScript脚本）来实现前端的交互。
2. 使用超媒体（HTML元素）格式（而不是JSON）与服务器交换数据。

现在就开启你的Htmx探索之旅吧。

## 参考阅读
- https://htmx.org/essays/a-real-world-react-to-htmx-port/
- https://htmx.org/essays/hateoas/
- https://en.wikipedia.org/wiki/Hypermedia
- https://htmx.org/essays/locality-of-behaviour/
- https://htmx.org/essays/hypermedia-driven-applications/
- https://en.wikipedia.org/wiki/Richardson_Maturity_Model
