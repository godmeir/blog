##TouchEven

#### 1、TouchEvent里的targetTouches、touches、changedTouches的区别的具体体现是？

* touches:当前屏幕上所有触摸点的集合列表
* targetTouches: 绑定事件的那个结点上的触摸点的集合列表
* changedTouches: 触发事件时改变的触摸点的集合


举例来说，比如div1, div2只有div2绑定了touchstart事件，第一次放下一个手指在div2上，触发了touchstart事件，这个时候，三个集合的内容是一样的，都包含这个手指的touch，然后，再放下两个手指一个在div1上，一个在div2上，这个时候又会触发事件，但changedTouches里面只包含第二个第三个手指的信息，因为第一个没有发生变化，而targetTouches包含的是在第一个手指和第三个在div2上的手指集合，touches包含屏幕上所有手指的信息，也就是三个手指。
![img](https://segmentfault.com/img/bVvmpn)
####  2、 javascript坐标

clientX 设置或获取鼠标指针位置相对于窗口客户区域的 x 坐标，其中客户区域不包括窗口自身的控件和滚动条。  
  
clientY 设置或获取鼠标指针位置相对于窗口客户区域的 y 坐标，其中客户区域不包括窗口自身的控件和滚动条。  
  
offsetX 设置或获取鼠标指针位置相对于触发事件的对象的 x 坐标。  
  
offsetY 设置或获取鼠标指针位置相对于触发事件的对象的 y 坐标。  
  
screenX 设置或获取获取鼠标指针位置相对于用户屏幕的 x 坐标。  
  
screenY 设置或获取鼠标指针位置相对于用户屏幕的 y 坐标。  
  
x 设置或获取鼠标指针位置相对于父文档的 x 像素坐标。  
  
y 设置或获取鼠标指针位置相对于父文档的 y 像素坐标。  




#### 3、获取点击元素的位置
#####  3.1、获取屏幕上的一个元素的位置

```javascript
绝对位置坐标：
$("#elem").offset().top
$("#elem").offset().left
相对父元素的位置坐标：
$("#elem").position().top
$("#elem").position().left
```
##### 3.2 jquery获取点击控件的绝对位置

#### 4 各种设备上事件的异同
#####  4.1 ipad手指触摸事件与pc鼠标事件
#####  4.2 判断是否为移动端设备的js代码

#### 5 事件的分发
1.事件的三个阶段

捕获
目标
冒泡
捕获(IE8及以下版本不支持)，目标，冒泡 捕获阶段给事件截获提供了可行性。

当点击目标元素的时候都是这三步，唯一的区别是你控制事件触发在哪个阶段。

所以我们可以看到要想截获事件必须要在目标元素事件发生之前，也就是捕获阶段。


![冒泡](http://oankigr4l.bkt.clouddn.com/%E5%86%92%E6%B3%A1%E5%9B%BE%E7%89%87.png)


```
var browser={  
    versions:function(){   
           var u = navigator.userAgent, app = navigator.appVersion;   
           return {//移动终端浏览器版本信息   
                trident: u.indexOf('Trident') > -1, //IE内核  
                presto: u.indexOf('Presto') > -1, //opera内核  
                webKit: u.indexOf('AppleWebKit') > -1, //苹果、谷歌内核  
                gecko: u.indexOf('Gecko') > -1 && u.indexOf('KHTML') == -1, //火狐内核  
                mobile: !!u.match(/AppleWebKit.*Mobile.*/), //是否为移动终端  
                ios: !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/), //ios终端  
                android: u.indexOf('Android') > -1 || u.indexOf('Linux') > -1, //android终端或者uc浏览器  
                iPhone: u.indexOf('iPhone') > -1 , //是否为iPhone或者QQHD浏览器  
                iPad: u.indexOf('iPad') > -1, //是否iPad    
                webApp: u.indexOf('Safari') == -1, //是否web应该程序，没有头部与底部  
                weixin: u.indexOf('MicroMessenger') > -1, //是否微信   
                qq: u.match(/\sQQ/i) == " qq" //是否QQ  
            };  
         }(),  
         language:(navigator.browserLanguage || navigator.language).toLowerCase()  
}   
  
  if(browser.versions.mobile || browser.versions.ios || browser.versions.android ||   
    browser.versions.iPhone || browser.versions.iPad){        
        window.location = "http://m.zhaizhainv.com";      
  }  

```

http://blog.sina.com.cn/s/blog_780a94270101kdgo.html
http://blog.csdn.net/chelen_jak/article/details/45365335

详解addEventListener的三个参数之useCapture

[参考链接1](https://segmentfault.com/q/1010000002870710/a-1020000004869367)
[参考链接2](http://blog.csdn.net/zhao19890429/article/details/13771405)
[客户端类型判断](http://blog.csdn.net/kongjiea/article/details/17612899)
[事件分发](http://www.cnblogs.com/zqzjs/p/5916591.html)