>作者：钟晟，Craze团队成员，CrazePOV项目发起人

旋转时钟（或称POV-LED），也就是利用视觉暂留现象，通过一排（或数目有限）的LED灯快速地在一定区域重复扫过，来实现一个面的图像或文字的显示。相信很多有电子设计基础、会写单片机的朋友都做过。

笔者最初就是在一个宣讲会上看到一位学长做的旋转时钟，从此便热爱上了电子设计，并计划着自己实现一个旋转时钟。通过学习和努力，最终完成了这一制作，在传统的旋转时钟的基础之上，也进行了一些改进，接下来便会同大家分享这一制作。笔者还有很多很有趣的点子，也会持续更新这个项目并实现这些点子。欢迎大家持续关注这个项目，有任何建议也欢迎告知笔者，让我们一起完善这个有趣的东西！

![](http://doask.qiniudn.com/openbook10pov.jpg)

<embed src="http://player.youku.com/player.php/sid/XNzM3MjE2Mjcy/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash"></embed>


## 点阵数据提取原理及其改进
但就如同大多数人看到的一样，由于旋转时钟是圆周形扫描，而现在大多数现有的图片或文字的点阵提取软件都是按照行或者列来进行扫描的。所以使用这些点阵提取软件得到的数据在旋转时钟上面显示出来的效果只会是一个看上去像是扇形的带有一定畸变的图形，并不能得到我们平时经常看到的正向显示的效果。

根据旋转时钟的显示原理：

![](http://doask.qiniudn.com/openbook10pov-theory.png)

可以看到，要想解决图像的畸变问题，只需要将需要显示的图片与显示的点进行一一对应，如下图所示：

![](http://doask.qiniudn.com/openbook10pov-theory-2.jpg)

如果我们想要在旋转时钟上面显示这只兔斯基，那么只需要兔斯基的轮廓覆盖住的相应的点点亮就可以实现理想的显示效果了。

旋转时钟的显示原理不能改变，那么可以做文章的就只有显示的数据了。在进行点阵数据提取的时候，一般传统的都是按照列或者行来提取数据，如下图所示，其中图片中的紫线代表正在扫描的位置，而箭头代表扫描的方向：

![](http://doask.qiniudn.com/openbook10pov-theory-3.jpg)

用这种方法扫描得到的点阵数据再通过旋转时钟进行显示以后自然会出现类似扇形的畸变。那么，我们反其道而行之，在扫描的时候根据旋转时钟显示的扫描方向，亦即以图片中心为原点，扫描线绕该原点旋转360°进行点阵数据的提取，那么最后通过旋转时钟显示得到的图片自然就是正常没有畸变的图片了。如下图所示，其中图片中的紫线代表正在扫描的位置，而箭头代表扫描的方向：

![](http://doask.qiniudn.com/openbook10pov-theory-4.jpg)

那么，在弄明白原理和问题的解决办法以后，就可以开始实践了！利用MATLAB写出相关代码，就可以实现图片的点阵数据的提取了。相关MATLAB代码如下：

```
clear;
clc;

%定义扫描范围是一个以450像素距离为半径的圆
l = 450;

%定义扫描精度，此处为一圈扫描360次，也就是精度为1°，可根据需要自行灵活设置
num = 360;

%生成一个用以暂存数据的数组，由于每个数据是一个0~255的整数，故用个相应维数的向量表示就可以了
output = 1 : num;

%读取图片数据，此处因为一个500*500的图片，且要显示的内容在一个以450像素距离为半径的圆内，可根据需要自行灵活设置
picture = uint8(255*ones(1000)) - imread('picture_1.bmp');

%以每1°为步进开始生成点阵数据
for m = 1 : num
%计算该时刻扫描线所处角度并计算x和y的变化量
a = (89 + m) * pi / 180; %前面加的89是为了重新设置起始扫描位置，以实现图片的显示方向，可根据需要自行灵活设置
    lx = l * cos(a);
    ly = l * sin(a);
    dx = lx / 31;
    dy = ly / 31;

    %确定扫描线的位置以后，以每个led为中心得到该处是否应该亮灯。值得注意的是，由于图片像素较高，需要一个范围内进行确认，此处是在一个5*5的巨星范围内确认
    for n = 0 : 31
        if (      picture(fix(500 - ly + n*dy),fix(500 + lx - n*dx))...
                & picture(fix(501 - ly + n*dy),fix(500 + lx - n*dx)) & picture(fix(499 - ly + n*dy),fix(500 + lx - n*dx))...
                & picture(fix(500 - ly + n*dy),fix(501 + lx - n*dx)) & picture(fix(500 - ly + n*dy),fix(499 + lx - n*dx))...
                & picture(fix(502 - ly + n*dy),fix(500 + lx - n*dx)) & picture(fix(498 - ly + n*dy),fix(500 + lx - n*dx))...
                & picture(fix(500 - ly + n*dy),fix(502 + lx - n*dx)) & picture(fix(500 - ly + n*dy),fix(498 + lx - n*dx))...
                & picture(fix(501 - ly + n*dy),fix(501 + lx - n*dx)) & picture(fix(501 - ly + n*dy),fix(499 + lx - n*dx))...
                & picture(fix(499 - ly + n*dy),fix(501 + lx - n*dx)) & picture(fix(499 - ly + n*dy),fix(499 + lx - n*dx))...
                & picture(fix(502 - ly + n*dy),fix(502 + lx - n*dx)) & picture(fix(502 - ly + n*dy),fix(498 + lx - n*dx))...
                & picture(fix(498 - ly + n*dy),fix(502 + lx - n*dx)) & picture(fix(498 - ly + n*dy),fix(498 + lx - n*dx)) )
%生成二进制数据
output(m) = bitor(output(m) , 2 ^ n);
        end
    end
end

%以十六进制的形式保存得到的点阵数据
fid=fopen('OUTPUT.txt','w');
fprintf(fid,'0x%x,0x%x,0x%x,0x%x,0x%x,0x%x,0x%x,0x%x,0x%x,0x%x,\n',output);
fclose(fid);
```
利用上段提取字模和图片点阵数据的Matlab程序，对需要提取的图片或者文字信息进行处理以后，就可以得到需要的数据了。下面是利用上诉程序得到的兔斯基这张图片的数据

![](http://doask.qiniudn.com/openbook10pov-data-matrix.jpg)

可以看到，每一组数据是由一个8位的十六进制数据构成，刚好是32位二进制数据，二进制“1”就代表是led要亮，“0”则代表灭。

## 单片机控制程序设计思路

在单片机控制程序方面需要的设计思路是利用光电传感器或者霍尔传感器（也可以使用诸如二维陀螺仪等传感器，但是这种一般在设计自行车轮毂的POV等不方便设置参考点的时候才需要用到，笔者在这里描述的POV主要是一个自制的单独模块，所以一般的光电传感器或者或偶尔传感器就可以满足设计要求了）在基板旋转到每一圈中的固定位置时，产生一个定位信号，并以此为基准开始每一度的扫描显示。

![](http://doask.qiniudn.com/openbook10pov-demo.jpg)

如上图所示，在每转一圈的同时进行计时，旋转一圈得到的时间数据除以显示精度（例如精度为1°时，就除以360）就可以得到扫描显示间隔的时间，然后根据旋转该圈起始时间和当前时刻就可以得到基板当前的实际位置，然后根据数据对应显示该位置应显示的内容即可。

有一点值得注意，若使用的单片机的缓存（ram）较大，可以设置一个显示缓存，扫描显示的时候显示的就是这个缓存里面的数据，需要变换图像的时候只需要更新缓存里面的数据即可。

显示控制的程序流程图：

![](http://doask.qiniudn.com/openbook10pov-flow-graph.jpg)

## 硬件环境的搭建
要实际做出一个旋转时钟，少了实物制作当然是不行的，旋转时钟的系统框图如下所示：

![](http://doask.qiniudn.com/openbook10pov-hardware-graph.jpg)

其中，具体需要注意的有以下几点：

* 微控制器的选取

由于要储存大量的图像信息（例如，以笔者试做的旋转时钟为例，取精度为1°，有单色的led共32只，那么在不考虑数据压缩的情况下，旋转一周所显示的图片的数据量为：360*32=11520 bit=1440 Byte，假如加入数字时钟等需要存入预定数据的应用，那么至少需要存储2+10+7+10=29张图片（由于位置的不同，和应用的特殊性，每个不同的位置的不同数字都需要单独的数据），也就是29*1440=41760 Byte的数据，笔者在编写程序的时候，消除了一定的冗余，但是最终的数据量也达到了20KB，显然，一般的单片机不能满足要求），所以微控制器的ROM不能太小，而且需要具备外部中断和定时器的基本功能。笔者之前试做的一个样品使用的控制芯片是MSP430F149，同样也可以考虑功能更强大的STM32F103系列，该系列有大容量的器件，还可以通过增加蓝牙等通讯模块来实现数据的实时透传，笔者后期也会考虑使用arduino等用户群体较大的器件；

* 电机的选取

转速不能太慢，否则显示会出现人眼可见的闪烁；但也不能太快，不然旋转的板子可能会产生脱落，摇晃等危险。故转速在2000rpm左右比较合适，最好是根据电压可调转速的。笔者之前试做的一个样品使用的电机是一个9V-12V可调，转速在2000rpm左右的直流电机；

* 供电方式的选取

具体来讲主要有电池供电、电刷供电、无线传能这三种方法。无线传能的实现难度比较大，而且效率较低，所以不予考虑；电池供电虽然简便，但是在旋转基板上面安装电池以后配重问题比较难解决，而且高速旋转可能导致电池飞出，所以也不予考虑；电刷供电虽然是机械式的供电方式，有机械损耗等问题，但是综合起来时比较靠谱的方案，所以笔者最终选用了电刷供电的供电方式。

![](http://doask.qiniudn.com/openbook10pov-brush-power.jpg)

如上图所示电机轴芯是与电机的供电电路绝缘的，所以可以用作旋转基板的供电（GND），在轴芯外面裹上一层热缩管用以绝缘，在套上一个铜套就可以用作基板的电源（VCC）供电。电刷只需要分别与铜套和转轴接触就可以实现旋转基板的供电。

* 定位方式的选取

在基板旋转到定位点以后，需要一个脉冲信号，用以标记零点位置并重置计数数据，方便在接下来的一圈中准确地对基板的实时位置经行定位。一般来说，主要有霍尔传感器和光电传感器两种方式，这两种方式基本都差不多，任选一种方式即可。笔者之前试做的一个样品使用的是光电传感器的方式。如果在不方便安置磁钢或者挡光片的情况下，也可以考虑使用二维陀螺仪等传感器；

* 旋转基板的配重
由于在旋转时钟工作的时候，旋转基板是高速旋转的，所以需要对旋转基板进行配重，使其能够平稳地工作。具体做法为：可以再旋转基板的两头预留一些孔位，根据重量在合适的位置添加铜柱以使基板的重心在旋转中心处；

基本上，进行实物制作的时候需要注意的就有以上几点。由于笔者在进行实验样品的实物制作的时候选择的是利用洞洞板和跳线手焊的方式，而电路图比较简单，也没有具体画出电路原理图或者绘制出PCB板，所以具体制作过程在此略过。但是，笔者正在制作一个可以实现RGB彩色显示的旋转时钟，而且正在设计相关的PCB板，预计2014年暑期就可以完成。而且该项目是一个开源项目，如果你对此项目感兴趣的话，欢迎关注其进展情况。

下面就是笔者制作的旋转时钟的实物图：

![](http://doask.qiniudn.com/openbook10pov-demo-1.jpg)

![](http://doask.qiniudn.com/openbook10pov-demo-2.jpg)

利用前面提到的提取字模和图片点阵数据的原理，通过Matlab编写相关程序，对需要提取的图片或者文字信息进行处理以后，把数据烧写到单片机中，便可以得到理想的效果。

下面是一些最终效果的展示：

![](http://doask.qiniudn.com/openbook10pov-demo-3.jpg)

![](http://doask.qiniudn.com/openbook10pov-demo-4.jpg)

![](http://doask.qiniudn.com/openbook10pov-demo-5.jpg)

![](http://doask.qiniudn.com/openbook10pov-demo-6.jpg)

## 写在最后
这个旋转时钟是笔者在前年，也就是2012年10月的时候帮做硬件的队友做出来当做生日礼物送给他姐姐的。由于那个时候专业知识还不够完善，没有学习PCB设计，所以无奈只能选择手焊的方式（真的是太痛苦了。。。），所以制作也不算精良。

笔者在写这篇文章的时候由于一些契机，将这个趣味制作翻了出来分享给大家，再加上笔者之前的一些奇思妙想，有了制作RGB彩色旋转显示系统的想法，而且经过这期间的学习，笔者已经会绘制PCB板了。所以，我也在继续完善这个项目，预计2014年暑期就可以完成RGB彩色旋转显示系统，下面是我已经完成设计的PCB板（正在工厂经行加工，很快就可以拿到实物了）的展示：

![](http://doask.qiniudn.com/openbook10pov-pcb-1.png)

![](http://doask.qiniudn.com/openbook10pov-pcb-2.png)

![](http://doask.qiniudn.com/openbook10pov-pcb-3.png)

![](http://doask.qiniudn.com/openbook10pov-pcb-4.png)

考虑到光心的准确重合，采用了RGB三个叶片状的设计，同时也突破了旋转时钟一贯的长条形外形风格。而且将供电模块，无线模块还有时钟模块全部和在一起画在另一层板子上面，同控制显示模块（即最大的那块板子）分立开来，不仅简化了布局布线，方便修改板子，也在一定程度上解决了配重的问题。由于主要的显示控制板采用三叶设计，比较像一个三叶螺旋桨，这也是这个项目名字FlyingPOV的由来。当然，它肯定不能飞啦~（笑）

该系统不仅可以像前述旋转时钟一样无畸变地显示图片和文字，还可以实现彩色图片的显示，理论上可以完成一些简单动画的播放，笔者还将加入蓝牙模块，以实现动画或者图片的实时显示。如果你对笔者的这一项目感兴趣，欢迎关注笔者的进展，相信一定不会令你失望的！


