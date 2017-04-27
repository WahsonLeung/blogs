# Route

标签（空格分隔）： 前端技术

---
[TOC]

---
### 1、URL结构（[链接][1]）

<img src="https://staticapps.org/articles/images/url-anatomy.svg">

```json
在浏览器中输入url，在控制台可以获取location对象的值：
window.location = 
{
    "hash":"#traffic-conditions",
    "search":"?type=daily",
    "pathname":"/weather/90014",
    "port":"",
    "hostname":"www.example.com",
    "host":"www.example.com",
    "protocol":"http:",
    "origin":"http://www.example.com",
    "href":"http://www.example.com/weather/90014?type=daily#traffic-conditions",
    "ancestorOrigins":{}
}

```
查看[Location API][2]

### 2、Polymer 自带路由组件的原理
> polymer主要通过以下组件实现路由：`<app-location>` `<app-route>` `<iron-location>` `<iron-query-params>`
```
<!- Polymer提供的地址解析控件 ->
<app-location route="{{route}}" use-hash-as-path></app-location>

<!- app-location内置了iron-location，iron-location主要是负责解析url ->
<iron-location
    path="{{__path}}"
    query="{{__query}}"
    hash="{{__hash}}"
    url-space-regex={{urlSpaceRegex}}>
</iron-location>
<!-
    path = window.location.pathname
    query = window.location.search.slice(1)
    hash = window.location.hash.slice(1)
->

<iron-query-params
    params-string="{{__query}}"
    params-object="{{queryParams}}">
</iron-query-params>

<!-
    iron-query-params负责把 "param1=abc&param2=def" 
    解析成对象 queryParams = { param1: "abc", param2: "def"} 
->
```
> 原理很简单，iron-location中监听了window.location的变化，然后把URL解析成path、query、hash
> 
```
//上例中，path、query、hash的值如下
path="/weather/90014"
query="type=daily"
hash="traffic-conditions"
```
> 接着< app-location>组装``route``对象，并向上notify。
```json
route = 
{
    "prefix":"",
    "path":"traffic-conditions",
    "__queryParams":{
        "type":"daily"
    },
}
注意这里path的值是hash的值，因为在app-location中，当使用use-hash-as-path时，path使用hash的值。
```
> 然后<app-route>根据`route`对象的值，和pattern匹配出要跳转的页面。
```
<app-route route="{{route}}" pattern="/:page" data="{{data}}" tail="{{tail}}"></app-route>
```
------
> 但是单纯使用polymer的路由方案并不能完全满足现时的需求。
### 3、< ks-route> 需求
> 
- 路由控件使用要方便
- 通过给定uri可以跳转到任意页面，并且uri中可以传递参数
- 页面间跳转，能够很方便传递参数
- 能够区分页面是第一次进入还是返回
- 需要有权限控制，如未登录时，通过uri访问订单页面需要重定向到登录页面

### 4、< ks-route> 方案
> 
1、采取所有页面都是一级页面的方案，不管多少级的页面，直接通过页面名称完成跳转。i.e. ``
AppRoute.show('center-index',param); 
AppRoute.show('order-payment',param);
``
2、使用Polymer的路由组件``<app-route>``、``<app-location>``、``<iron-pages>``，在这三个组件的基础上进行封装实现``<ks-route>``、``<ks-location>``
>
3、浏览器直接定位资源方式：页面后直接带上参数
http://127.0.0.1:8088/main.html#/test-order-page**?orderNo=orderNo12**

**[查看代码][3]**
#### 路由数据流动
1、from 一个页面（page1） to 另一个页面（page2）
<ks-route> 
```sequence
    page1->>ks_route:调用: show(page2)
    ks_route->>app_route: 更新app_route绑定的rt
    app_route-->>ks_route:data
    ks_route->>iron_pages:更新显示的页面
    Note over ks_route:进行页面权限检查，然后调用页面定义的display方法
    ks_route->>ks_location: 更新location绑定，更新 url
```
``注意：app-route和ks-location的数据绑定和更新都在show()方法里触发``

2、页面返回
```sequence
    page2->>ks_route:调用back()
    Note over ks_route:call:history.back()
```
调用history.back()后，url会发生改变：
```sequence
    Note over ks_location:url changed
    
    alt 页面改变
        ks_location->>ks_route:fire route-change event
    else 页面参数改变
        ks_location->>ks_route:fire query-param-change event
    end
    ks_route->>iron_pages:更新显示的页面
    Note over ks_route:进行页面权限检查，然后调用页面定义的display方法
```

3、url 直接进入页面
``与调用history.back()后的过程相同。``
#### 使用方法
```html
//页面声明
<ks-route id="appRoute" is-root>
    <section id="main-page">
</ks-route>
<script>
    const AppRoute = document.querySelector("#appRoute");
</script>
```

```javascript
//页面跳转
AppRoute.show(`${pageName}`,param);
//页面返回
AppRoute.back();
```

```javascript
//main-page.html
Polymer({
    is:'main-page',
    //check authority before display
    preCheck(resolve, reject) {
        //do something here
        
        //if preCheck passed, then return resolve()
        //else return reject()
        //you can also pass a closure as an argument to reject method
        //i.e. reject(() => console.log("preCheck not passed");)
        return resolve();
    },

//this method will be called with an argument({param:{isReturn: false, yourParamKey: yourParamValue}}) when this page is displayed
    display({param:{isReturn, orderNo}}={}){
        if(isReturn) console.log("page return occured");
        if (!!orderNo) this.set("orderNo", orderNo);
    },
    
    back() {
        return AppRoute.back();
    },
    
    //display order-page
    showOrderPage(){
        AppRoute.show("order-page",{orderNo:123});
    }
});
```
### 5、不足
> 
- 不容易定制页面的切换效果
- ...

### 6、其他
> 
- 页面html和js中混杂了路由跳转的东西，需要处理，工作量不少
- ...

### 7、问题
> 
- 权限控制
- 页面重定向

### 8、重构总结
编程式点击a标签方法：
在一般浏览器下：
```javascript
<!-- html -->
<a id="aid" href="http://xxx.com"></a>

//js
$('#aid').click();
```
在开发webapp时，当不得不使用a标签时，就要小心点击穿透问题。下面的方法可以有效防止点击穿透：
```javascript
<!-- html -->
<a id="aid" on-tap="redirectHandler"></a>

//js
const redirectHandler = (e) => {
    $('#aid').attr('href','http://xxx.com').append('<span></span>').find('span:eq(0)').click().remove();
    $('#aid').removeAttr('href');
}
```
但是，这种方法在polymer 1.7之前是行之有效的，在升级到1.7后，发现这种方法死活不生效，最后通过把remove()后的代码添加到setTimeout里面即可。**原因未明**。

```javascript
const redirectHandler = (e) => {
    $('#aid').attr('href','http://xxx.com').append('<span></span>').find('span:eq(0)').click().remove();
    setTimeout(() => {
        $('#aid').removeAttr('href');
    },0);
}
```

**使用window.onpopstate要注意：一般来说只有进行浏览器动作时（如点击返回按钮或调用history.back()）才会触发，但是在chrome和safari中，当网页加载时也会触发。**


  [1]: https://staticapps.org/articles/routing-urls-in-static-apps/
  [2]: http://www.w3schools.com/jsref/obj_location.aspcapps.org/articles/routing-urls-in-static-apps/
  [3]: http://git.oa.isuwang.com/isuwang-com/isuwang-app/tree/F_WECHAT_V1.0/app/source/elements/demo