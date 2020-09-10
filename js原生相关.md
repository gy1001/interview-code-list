

##  Bug-List
[TOC]
### vue 单页面微信分享

> 表现形式： 安卓端分享成功，ios端分享设置失败

> 原因分析： 
> * 1. Ios 每次路由切换时，url是不会变的，发起签名请求的url参数必须是当前页面的url也就是最初进入页面的url
> * 2. Android 每次切换路由时，url是会变的，发起签名请求的url参数必须是当前页面的url(不是最初进入的页面)
> * 3. 官方文档原文：所有需要使用JS-SDK的页面必须先注入配置信息，否则将无法调用（同一个url仅需调用一次，对于变化url的SPA的web app可在每次url变化时进行调用,目前Android微信客户端不支持pushState的H5新特性，所以使用pushState来实现web app的页面会导致签名失败，此问题会在Android6.2中修复）。
> * 4. 总得说，就是Vue单页面应用在ios环境下，微信只会记录第一次你进入的页面url，尽管你的路由发生了跳转，但是并没有触发微信更新当前的url函数，所以当你获得到的url和你当前实际的url不一致就会导致签名失败，

> 解决方法
> * IOS 需要修正url路径
>  1. 进行刷新页面，location.href 跳转到需要子自定义分享的页面，注意会导致之前的keep-alive失效，因为可以算作重新打开一个页面
>  2. 注册时候ios单独处理, 获取配置信息时候的url传递为刚进入页面时候的url

### mand-mobile中 swiper 组件bug
> 复现方式：swiper-item 进行padding 或者margin 样式时候 , 弱网方式下 初始化宽度或者高度(竖滑) 会错误

> 解决方法: this.$nextTick中重新进行初始化（不推荐）

> 上述问题出现原因：使用了px to rem转换，根元素也设置了fontSize，在尺寸转换后但是根节点fontSize还没赋值上就会出现获取的clientHeight 或者 clientWidth异常错误
> 根元素设置代码移动到head中提前执行即可

### 关于display:flex碰上white-space nowrap 影响布局的问题
> w3c school 原文：By default, flex items won’t shrink below their minimum content size (the length of the longest word or fixed-size element).

> 解决方案：文本父级div设置min-width:0即可,因为设置了white-space:nowrap，所以content没法收缩了，设置0后就有了固定尺寸就可以收缩了
> 代码如下
```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>flex 布局</title>
  </head>
  <style>
    .outer{
      display: flex;
      border: solid 1px red;
    } 
    .box{
      width: 100px;
      height: 100px;
      background-color: aquamarine;
    }
    .right{
      flex: 2;
      white-space: nowrap;
      min-width: 0;
      overflow: hidden;
      text-overflow: ellipsis;
    }
  </style>
  <body>
    <div class="outer">
      <div class="box">
      </div>
      <div class="right">
        测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试
        测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试
        测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试
        测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试
        测试测试测试测试测试测试测试测试测试测试测试测试试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测
        试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试
      </div>
    </div>
  </body>
  </html>
```

### JS文件下载,页面跳转出现空白页
> 代码：直接使用a链接进行跳转 或者 使用点击事件location.href=""进行跳转，ios手机是执行打开预览，大部分安卓手机是打开下载（在微信里也是提示下载），部分手机在微信里打开会跳转打开系统浏览器提示下载，然后返回微信是空白页，再次返回直接关闭页面

1. 方案：用 JS 添加a标签元素并触发点击事件，然后移除该元素
> tips： 自测在某测试机上不好使，一如既往的会出现空白页面
```javascript
  var downLoad=function(src){
  var a = document.createElement('a');
  a.id = 'expertFile'
  a.href =src;
  document.body.append(a); 
  a.click();
  document.getElementById('expertFile').remove();
}
downLoad()
```
2. 方案：通过添加 iframe 元素,打开连接，然后移除该 iframe
> 亲测在出现空白页面手机，完美下载并不出现空白页
```javascript 
  function  download (src) {
    var iframe = document.createElement("iframe");
    document.body.appendChild(iframe);
    iframe.src = src
    iframe.remove()
  }
```
参考连接: [跳转](https://blog.csdn.net/weixin_39417767/article/details/84874516)

### video标签相关
#### 隐藏全屏按钮
1. 使用CSS
```css
  /*video默认全屏按钮*/
  video::-webkit-media-controls-fullscreen-button{ 
    display: none 
  }
  /*video默认aduio音量按钮*/
    video::-webkit-media-controls-mute-button { 
      display: none
    }

    /*video默认setting按钮*/
    video::-internal-media-controls-overflow-button{ 
      display: none
    }

```
#### 删掉下载/全屏等按钮
1. 属性更改： 添加属性
```html
  controlsList='nofullscreen nodownload noremote footbar'
```
2. CSS更改(部分手机无效)
```css
  video::-internal-media-controls-download-button {
      display:none;
  }
```

参考链接：
  1. [跳转](https://zhuanlan.zhihu.com/p/37219669)
  2. [跳转](https://blog.csdn.net/jiajia199470/article/details/85276267)
  3. [跳转](https://blog.csdn.net/jw19950424/article/details/80323087)

#### video在微信浏览器中的标签
1. 安卓机
    * 部分安卓机视频会悬浮，需要设置属性
      webkit-playsinline="true"
      x-webkit-airplay="true"
      playsinline="true"
      x5-video-player-type="h5"
      x5-video-player-fullscreen="true"
2. IOS
    * 手机播放会自动全屏播放
#### video在webView中的表现
1. 安卓机
      * App中不能全屏播放，否则会卡死，需要App对webview做一些属性支持
2. IOS
      * App中可以播放

参考链接：
  1. [如何禁止 iPhone Safari 视频自动全屏？](https://www.zhihu.com/question/21094425)
  2. [HTML5 inline video on iPhone vs iPad/Browser](https://stackoverflow.com/questions/3699552/html5-inline-video-on-iphone-vs-ipad-browser)
  3. [video在安卓下不能全屏播放](https://ask.dcloud.net.cn/question/12689)

### 移动端查看pdf文件
  1. 使用 vue-pdf
      * bug: 涉及到第三方签章时候，会导致签章不显示，慎用
  2. 使用 pdfh5
      * 正常显示并完美解决上述签章不显示问题

参考链接： 
1. [Vue关于pdf展示问题——第三方电子签章不能正常展示](https://blog.csdn.net/AnnaF/article/details/107231856)
2. [pdfh5](https://www.gjtool.cn/archives/pdfh5)
3. [vue-pdf](https://www.npmjs.com/package/vue-pdf)


<span id="jump">MarkDown页面内跳转测试</span>