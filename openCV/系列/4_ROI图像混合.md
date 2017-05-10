###【OpenCV入门教程之四】 ROI区域图像叠加&初级图像混合 全剖析
在这篇文章里，我们一起学习了在OpenCV中如何定义感兴趣区域ROI，如何使用addWeighted函数进行图像混合操作，以及将ROI和addWeighted函数结合起来使用，对指定区域进行图像混合操作。
####一、设定感兴趣区域——ROI（region of interest）
定义ROI区域有两种方法，

第一种是使用cv:Rect.顾名思义，cv::Rect表示一个矩形区域。指定矩形的左上角坐标（构造函数的前两个参数）和矩形的长宽（构造函数的后两个参数）就可以定义一个矩形区域。

另一种定义ROI的方式是指定感兴趣行或列的范围（Range）。Range是指从起始索引到终止索引（不包括终止索引）的一连段连续序列。cv::Range可以用来定义Range。如果使用cv::Range来定义ROI，那么前例中定义ROI的代码可以重写为：

<pre>
//----------------------------------【ROI_AddImage( )函数】----------------------------------  
// 函数名：ROI_AddImage（）  
//     描述：利用感兴趣区域ROI实现图像叠加  
//----------------------------------------------------------------------------------------------  
bool ROI_AddImage()  
{  
   
       //【1】读入图像  
       Mat srcImage1= imread("dota_pa.jpg");  
       Mat logoImage= imread("dota_logo.jpg");  
       if(!srcImage1.data ) { printf("你妹，读取srcImage1错误~！ \n"); return false; }  
       if(!logoImage.data ) { printf("你妹，读取logoImage错误~！ \n"); return false; }  
   
       //【2】定义一个Mat类型并给其设定ROI区域  
       Mat imageROI= srcImage1(Rect(200,250,logoImage.cols,logoImage.rows));  
   
       //【3】加载掩模（必须是灰度图）  
       Mat mask= imread("dota_logo.jpg",0);  
   
       //【4】将掩膜拷贝到ROI  
       logoImage.copyTo(imageROI,mask);  
   
       //【5】显示结果  
       namedWindow("<1>利用ROI实现图像叠加示例窗口");  
       imshow("<1>利用ROI实现图像叠加示例窗口",srcImage1);  
   
       return true;  
}  
</pre>

这个函数首先是载入了两张jpg图片到srcImage1和logoImage中，然后定义了一个Mat类型的imageROI，并使用cv::Rect设置其感兴趣区域为srcImage1中的一块区域，将imageROI和srcImage1关联起来。接着定义了一个Mat类型的的mask并读入dota_logo.jpg，顺势使用Mat:: copyTo把mask中的内容拷贝到imageROI中，于是就得到了最终的效果图，namedWindow和imshow配合使用，显示出最终的结果。

####二、初级图像混合——线性混合操作
线性混合操作是一种典型的二元（两个输入）的像素操作，它的理论公式是这样的：  我们通过在范围0到1之间改变alpha值，来对两幅图像（f0（x）和f1（x））或两段视频（同样为（f0（x）和f1（x））产生时间上的画面叠化（cross-dissolve）效果，就像幻灯片放映和电影制作中的那样。即在幻灯片翻页时设置的前后页缓慢过渡叠加效果，以及电影情节过渡时经常出现的画面叠加效果。
实现方面，我们主要运用了OpenCV中addWeighted函数，我们来全面的了解一下它：g(x) = (1-alpha)f0(x) + alphaf1(x)

实现方面，我们主要运用了OpenCV中addWeighted函数，我们来全面的了解一下它：

**addWeighted函数**,这个函数的作用是，计算两个数组（图像阵列）的加权和。如果用数学公式来表达，addWeighted函数计算如下两个数组（src1和src2）的加权和，得到结果输出给第四个参数。即addWeighted函数的作用可以被表示为为如下的矩阵表达式为：

 dst = src1[I]\*alpha+ src2[I]\*beta + gamma;

结合ROI设置混合

<pre>
//---------------------------------【ROI_LinearBlending（）】-------------------------------------  
// 函数名：ROI_LinearBlending（）  
// 描述：线性混合实现函数,指定区域线性图像混合.利用cv::addWeighted（）函数结合定义  
//                     感兴趣区域ROI，实现自定义区域的线性混合  
//--------------------------------------------------------------------------------------------  
bool ROI_LinearBlending()  
{  
   
       //【1】读取图像  
       Mat srcImage4= imread("dota_pa.jpg",1);  
       Mat logoImage= imread("dota_logo.jpg");  
   
       if(!srcImage4.data ) { printf("你妹，读取srcImage4错误~！ \n"); return false; }  
       if(!logoImage.data ) { printf("你妹，读取logoImage错误~！ \n"); return false; }  
   
       //【2】定义一个Mat类型并给其设定ROI区域  
       Mat imageROI;  
              //方法一  
       imageROI=srcImage4(Rect(200,250,logoImage.cols,logoImage.rows));  
       //方法二  
       //imageROI=srcImage4(Range(250,250+logoImage.rows),Range(200,200+logoImage.cols));  
   
       //【3】将logo加到原图上  
       addWeighted(imageROI,0.5,logoImage,0.3,0.,imageROI);  
   
       //【4】显示结果  
       namedWindow("<4>区域线性图像混合示例窗口 by浅墨");  
       imshow("<4>区域线性图像混合示例窗口 by浅墨",srcImage4);  
        
       return true;  
}  

</pre>

