##Image类

###预载入

对于浏览器载入图像来说，只有在对图像发送一个 HTTP请求之后，它们才会被浏览器载入，对图像的 HTTP 请求要么使用 ＜img＞ 标记，要么通过方法调用实现。如果使用 JavaScript 脚本来处理在 mouseover 事件时交换图像，或者在一段时间之后自动更改图像，那么在从服务器获取图像时可能要等上几秒钟到几分钟的时间。如果使用一个慢速的 Internet 连接，或者要获取的图像非常大，或者其它一些情况，这种现象就特别明显；这样，延迟就造成你不能达到自己期望的效果。

一些浏览器采用一些措施来缓解这一问题，比如试图通过在本地缓存中存储图像，从而使随后对图像的调用能够立即被满足；但是在图像第一次被调用时依然会存在一些延迟。**预载入是在需要图像之前将其下载到缓存的一种方法。通过这一措施，当真正需要图像时，它就可以被立即从缓存中取出，从而能够立即显示**。


###Image()对象
创建一个Image对象：`var a=new Image()`;    定义Image对象的src:` a.src=”xxx.gif”`;    这样做就相当于给浏览器缓存了一张图片。

* 建立图像对象：图像对象名称=new Image([宽度],[高度])

* 图像对象的属性： border complete height hspace lowsrc name src vspace width

* 图像对象的事件：onabort onerror onkeydown onkeypress onkeyup onload

* 需要注意的是：src 属性一定要写到 onload 的后面，否则程序在 IE 中会出错。

<pre><code>
var img=new Image();  
    img.onload=function(){alert("img is loaded")};  
    img.onerror=function(){alert("error!")};  
    img.src="http://www.abaonet.com/img.gif";  
    function show(){alert("body is loaded");};  
    window.onload=show;  
</code></pre>

**在 FF 中**，img 对象的加载包含在 body 的加载过程中，既是 img加载完之后，body 才算是加载完毕，触发 window.onload 事件。

**在 IE 中**，img 对象的加载是不包含在 body 的加载过程之中的，body 加载完毕，window.onload 事件触发时，img对象可能还未加载结束，img.onload事件会在 window.onload 之后触发。

根据上面的问题，考虑到浏览器的兼容性和网页的加载时间，尽量不要在Image 对象里放置过多的图片，否则在 FF 下会影响网页的下载速度。**当然如果你在 window.onload 之后，执行预加载函数，就不会有 FF 中的问题了**。



###onLoad() 事件处理器 

像很多 JavaScript 的其它对象一样，Image() 对象也有一些事件处理器。其中最有用的一个肯定是 onLoad() 处理器，它在图像完全载入之后调用。这个事件处理器可以与一个自定义函数联系起来，以在图像完全载入之后执行一些特定的任务。


###complete ()属性
 可以通过Image对象的complete 属性来检测图像是否加载完成（每个Image对象都有一个complete属性，当图像处于装载过程中时，该属性值false,当发生了onload、onerror、onabort中任何一个事件后，则表示图像装载过程结束（不管成没成功），此时complete属性为true）

###注
ie 火狐等大众浏览器均支持 Image对象的onload事件。

ie8及以下、opera 不支持onerror事件


###eg:
```
<pre><code>
   modeImgCache = (function () {

    var Count =16;
    var Imgs = new Array(Count);
    var ImgLoaded =0;

    //加载单个图片
    this.downloadImage = function (i)
    {
        var imageIndex = i; //图片以1开始
        Imgs[i].src = "img/mode_bg_"+imageIndex+".png";
        Imgs[i].onLoad=validateImages(i);
    }

    //验证是否成功加载完成，如不成功则重新加载
    function validateImages(i){
        if (!Imgs[i].complete)
        {
            setTimeout('downloadImage('+i+')',200);
        }
        else if (typeof Imgs[i].naturalWidth != "undefined" && Imgs[i].naturalWidth == 0)
        {
            setTimeout('downloadImage('+i+')',200);
        }
        else
        {
            ImgLoaded++
        }
    }
    //预加载图片
    function preLoadImgs()
    {
        for(var i=1;i<=16;i++){
            Imgs[i]=new Image();
            try {
                downloadImage(i);
            }catch (e){
                console.log(e);
            }

        }
    };
    preLoadImgs();
    return Imgs;
})()
</code></pre>
```
