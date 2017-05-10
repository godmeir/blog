###【OpenCV入门教程之六】 创建Trackbar & 图像对比度、亮度值调整
这篇文章中我们一起学习了如何在OpenCV中用createTrackbar函数创建和使用轨迹条，以及图像对比度、亮度值的动态调整。
文章首先详细讲解了OpenCV2.0中的新版创建轨迹条的函数createTrackbar，并给上一个详细注释的示例。
然后讲解图像的对比度、亮度值调整的细节，最后放出了一个利用createTrackbar函数创建轨迹条来辅助进行图像对比度、亮度值调整的程序源码。

####一、OpenCV中轨迹条（Trackbar）的创建和使用
* <1>创建轨迹条——createTrackbar函数详解<code>C++: int createTrackbar(conststring& trackbarname, conststring& winname,  
 int* value, int count, TrackbarCallback onChange=0,void* userdata=0);  </code>

第一个参数，const string&类型的trackbarname，表示轨迹条的名字，用来代表我们创建的轨迹条。

第二个参数，const string&类型的winname，填窗口的名字，表示这个轨迹条会依附到哪个窗口上，即对应namedWindow（）创建窗口时填的某一个窗口名。

第三个参数，int* 类型的value，一个指向整型的指针，表示滑块的位置。并且在创建时，滑块的初始位置就是该变量当前的值。

第四个参数，int类型的count，表示滑块可以达到的最大位置的值。PS:滑块最小的位置的值始终为0。

第五个参数，TrackbarCallback类型的onChange，首先注意他有默认值0。这是一个指向回调函数的指针，每次滑块位置改变时，这个函数都会进行回调。**并且这个函数的原型必须为void XXXX(int,void\*)**;其中第一个参数是轨迹条的位置，第二个参数是用户数据（看下面的第六个参数）。如果回调是NULL指针，表示没有回调函数的调用，仅第三个参数value有变化。

第六个参数，void*类型的userdata，他也有默认值0。这个参数是用户传给回调函数的数据，用来处理轨迹条事件。如果使用的第三个参数value实参是全局变量的话，完全可以不去管这个userdata参数。



**看完第五、六个参数的定义，就可知道我们问题的答案了。因为回调函数要求这种格式，是故一定要遵循这种格式进行定义。而我们已经将第三个参数声明为全局变量，则不需要由第一个参数的值来获得轨迹条的位置，又我们第六个参数采取默认值，则亦可无视回调函数的第二个参数，是故我们仅仅只是把参数的类型放在那里而无需声明参数名来加以调用。**

<pre>
//-----------------------------------【头文件包含部分】---------------------------------------  
//  描述：包含程序所依赖的头文件  
//----------------------------------------------------------------------------------------------   
#include "opencv2/imgproc/imgproc.hpp"  
#include "opencv2/highgui/highgui.hpp"  
#include <iostream>  
  
//-----------------------------------【命名空间声明部分】---------------------------------------  
//  描述：包含程序所使用的命名空间  
//-----------------------------------------------------------------------------------------------     
using namespace cv;  
using namespace std;  
  
//-----------------------------------【全局函数声明部分】--------------------------------------  
//  描述：全局函数声明  
//-----------------------------------------------------------------------------------------------  
Mat img;  
int threshval = 160;            //轨迹条滑块对应的值，给初值160  
  
//-----------------------------【on_trackbar( )函数】------------------------------------  
//  描述：轨迹条的回调函数  
//-----------------------------------------------------------------------------------------------  
static void on_trackbar(int, void*)  
{  
    Mat bw = threshval < 128 ? (img < threshval) : (img > threshval);  
  
    //定义点和向量  
    vector<vector<Point> > contours;  
    vector<Vec4i> hierarchy;  
  
    //查找轮廓  
    findContours( bw, contours, hierarchy, CV_RETR_CCOMP, CV_CHAIN_APPROX_SIMPLE );  
    //初始化dst  
    Mat dst = Mat::zeros(img.size(), CV_8UC3);  
    //开始处理  
    if( !contours.empty() && !hierarchy.empty() )  
    {  
        //遍历所有顶层轮廓，随机生成颜色值绘制给各连接组成部分  
        int idx = 0;  
        for( ; idx >= 0; idx = hierarchy[idx][0] )  
        {  
            Scalar color( (rand()&255), (rand()&255), (rand()&255) );  
            //绘制填充轮廓  
            drawContours( dst, contours, idx, color, CV_FILLED, 8, hierarchy );  
        }  
    }  
    //显示窗口  
    imshow( "Connected Components", dst );  
}  
  
  
//-----------------------------------【main( )函数】--------------------------------------------  
//  描述：控制台应用程序的入口函数，我们的程序从这里开始  
//-----------------------------------------------------------------------------------------------  
int main(  )  
{  
    system("color 5F");    
    //载入图片  
    img = imread("1.jpg", 0);  
    if( !img.data ) { printf("Oh，no，读取img图片文件错误~！ \n"); return -1; }  
  
    //显示原图  
    namedWindow( "Image", 1 );  
    imshow( "Image", img );  
  
    //创建处理窗口  
    namedWindow( "Connected Components", 1 );  
    //创建轨迹条  
    createTrackbar( "Threshold", "Connected Components", &threshval, 255, on_trackbar );  
    on_trackbar(threshval, 0);//轨迹条回调函数  
  
    waitKey(0);  
    return 0;  
}  
</pre>

* <2>获取当前轨迹条的位置——getTrackbarPos函数

<code>C++: int getTrackbarPos(conststring& trackbarname, conststring& winname);  </code>

第一个参数，const string&类型的trackbarname，表示轨迹条的名字。
第二个参数，const string&类型的winname，表示轨迹条的父窗口的名称。