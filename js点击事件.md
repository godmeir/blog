###html点击事件
* touchstart:触摸开始的时候触发
* touchmove:手指滑动的时候触发
* touchend:触摸结束的时候触发

每个触发事件都有三个触摸列表，每个触摸列表里都包含了对应的一系列触摸点，用来实现多点触控

* touchs:当前位于屏幕上的所有手指的列表
* targetTouches:位于当前dom元素上的手指的列表
* changedTouches:涉及当前事件手指的列表

每个触摸点包含如下触摸信息：

* identifier:一个数值，唯一标识触摸会话中的当前手指。一般为重零开始的流水号
* target:DOM元素，是动作所针对的目标
* pageX/pageY/clientX/clientY/screenX/screenY:一个数值，动作在屏幕上发生的位置（page包含滚动距离，client不包含滚动距离，screen则是以屏幕为基准）
* radiusX/radiusY/rotationAngle:画出大约相当于手指形状的椭圆形的两个半径和旋转角度，初步测试浏览器不支持

###eg:
<pre><code>
var obj = document.getElementByIdx_x('id');
obj.addEventListener('touchmove', function(event) {
     // 如果这个元素的位置内只有一个手指的话
    if (event.targetTouches.length == 1) {
　　　　 event.preventDefault();// 阻止浏览器默认事件，重要 
        var touch = event.targetTouches[0];
        // 把元素放在手指所在的位置
        obj.style.left = touch.pageX-50 + 'px';
        obj.style.top = touch.pageY-50 + 'px';
        }
}, false);
</code></pre>