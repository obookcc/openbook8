# 哨戒炮2

>Lee苦海


上回我们说到从摄像头的输出中把目标的颜色提出来，并使用扩散和腐蚀效果对结果做一点加工。这叫二值化。下一步就是根据目标颜色在图像中位置，找出目标的中心。

    imshow( "thresh ", thresh,);//

和之前一样，我们可以使用imshow()来查看效果，之后也一样，只要对象是mat或Iplimage(本文未使用)格式，都可以使用imshow()来查看。

![](http://doask.qiniudn.com/shaojiepao1.png)

当然对于人而言，目标已经很清晰了。但是对计算机来说，计算机看到的还是一堆数字，它并不知道目标的位置。所以我们使用cvFindContours()来让计算机找出目标的轮廓。


        vector<vector<Point> > contours;
        vector<Vec4i> hierarchy;
        findContours(thresh,Rcontours,Rhierarchy,CV_RETR_TREE,CV_CHAIN_APPROX_SIMPLE, Point(0, 0) );

这个函数会找出图像中的轮廓，公式有点长，但多数时侯都不会做任何改写。照抄即可。

在理想情况下，我们通常可以用上期提到的扩散和腐蚀来去掉画面多数不想要的噪声。但是有些噪声可能比较顽固，这里我们可以检查全部轮廓，从中找出最大的轮廓将其当成目标，并把其他轮廓全部当成噪声。


        vector<vector<Point> > contours_poly( contours.size() );
        vector<Rect> boundRect( contours.size() );
        //定义一个Rect矢量来存放轮廓。因为轮廓的外形多数时候是不规则的。所以用一个矩形来代替不规则的轮廓会在各种方面都方便很多。
        vector<Point2f>center( contours.size() );
        vector<float>radius( contours.size() );
        int midX=0;
        int midY=0;
        int maxArea =0;
        int index=0;
        for( unsigned int i = 0; i< contours.size(); i++ ) )//用一个for循环语句查看计算机找到的全部轮廓
        {
            int  area=contourArea(contours[i]);//计算当前轮廓的包含面积
            if(area> maxArea)//找出包含面积最大的轮廓
            {
                maxArea = area;
                index =i;
            }


            approxPolyDP( Mat(contours[i]), contours_poly[i], 3, true );// approxPolyDP()用来找出轮廓的近似多边形。用于简化轮廓的杂度，加速计算过程。
            boundRect[i] = boundingRect( Mat(contours_poly[i]) );//BoundingRect()是一个用来找出轮廓最小包围矩形函数。最小包围矩形的意思就是用4条边从上下左右四个方向把轮廓紧紧夹在中间。这4条边构成的矩形就是最小包围矩形。
            minEnclosingCircle( (Mat)contours_poly[i], center[i], radius[i] );// minEnclosingCircle()是一个用来找出轮廓最小包围圆形的函数。其原理类似最小包围矩形。如果目标是圆形时，找最小包围圆比最小包围方更方便

            drawContours( frame, contours, index, red, 1, 8, hierarchy, 0, Point() );//画出物体的轮廓
            rectangle( frame,  boundRect[index].tl(), boundRect[index].br(), red, 2, 8, 0 );//画出物体的最小包围矩形

            circle( frame, center[index], (int)Rradius[index], red, 2, 8, 0 );//画出物体的最小包围圆形

![](http://doask.qiniudn.com/shaojiepao2.png)

图中有三个框框框住了红宝书，近贴着红宝书的是approxPolyDP()算出的轮廓。
矩形的自然就是boundRect()算出的轮廓。
圆形的自然就是minEnclosingCircle()算出的轮廓。

    //计算目标的中心坐标
            midX = RboundRect[index].width/2 + boundRect[index].x;
            midY = RboundRect[index].height/2 + boundRect[index].y;

        }

    imshow("frame", frame);
    //看看结果怎么样：>

    if(maxArea >5000)
    //如果最大轮廓的面积超过5000个像素，即认为他不是噪声，而是目标

    {
    std::cout<<"midX = "<<midX<<" midY ="<<midY<<std::endl;//打印目标坐标。

作者使用的分辨率为640*480，这一是有组于提高运算速度，二是方便截图写这稿子。
在这个分辨率下，我把摄像头的图像分成9个区域：

| 1 | 2 | 3 |
| -- | -- | -- |
| 4 | 5 | 6 |
| 7 | 8 | 9 |

如过目标中心不在区域5内，则转动云台。比如当目标在区域2，即让云台抬头就好。如果目标在区域4，就要让云台向左转头。如果目标在区域1，那云台就要一边抬头，一边想左转头。当目标中心进入区域5内停止。

在作者的应用中，作者将区域5的大小定位100*100个像素。这对作者的霰弹枪流气动发射炮台足够了。

但是如果你想做个用狙击枪的炮台，这个瞄准精度可能就无法满足。所以你要做的就是在区域5里再画9个格子并重复一样的运算。或者你有能力用上光学变焦，那就JJ BOMBSKY了。但是别忘了你的云台系统是否有最小移动距离，比如步进电机就有最小步长。

另外作者弄了个简单的通信协议：
>a为电机顺时针转动

>b为电机逆时针转动

>c 为电机停止

大小写区分左右转头电机，和仰俯电机。

```

  if(midX > 370){Mega.SerialWrite('a');} //a
else if(midX <= 370 && midX >=270 ){Mega.SerialWrite('c');} // c
//区域5为100像素长宽时，得出X轴的两条分界线在270 和370。
    else {Mega.SerialWrite('b');} //b

    if(midY > 270){Mega.SerialWrite('B');} //B
else if(midY <=270 && midY >200){Mega.SerialWrite('C');}//C
//区域5为100像素长宽时，得出X轴的两条分界线在200 和270。

    else {Mega.SerialWrite('A');}//A
}
else {
std::cout<<"TARGET TOO SMALL OR NOT FOUND"<<std::endl;

//如果目标太小，就开始不断摆头进行搜索，直到发现满足条件的目标。

//左右转头电机的搜索
timer1++;
if(timer1 ==480){timer1=0;}
if(timer1<240){Mega.SerialWrite('a');}
if(timer1>=240){Mega.SerialWrite('b');}

//仰俯电机的搜索
timer2++;
if(timer2==150){timer2=0;}
if(timer2<75){Mega.SerialWrite('A');}
if(timer2>=75){Mega.SerialWrite('B');}

}



```

最后在while循环里加上这句
    if(waitKey(30) >= 0) break;//等待按键，ms为等待时间，单位为毫秒。

如果有按下ESC键便跳出，终止程序。因为cvWaitKey()中有延迟功能，所以很多人使用这句来控制图像处理的频率。比如如果你的图像算法简单，运算时间可以忽略不计的话，那将ms设为30就能每30毫秒处理一幅图像，换而言之，1秒/30毫秒 =33帧的图像。由于Opencv也有将图像存成影片格式的功能，所以通过改延迟就能很容易完成延迟摄影这类功能。

###第三部分：通信

下面要做的就是讲这两个数送往控制云台电机的系统。
当然我知道很多人会想使用树莓派，PCDUINO，Cubieboard之类既有能力运行Opencv又自带GPIO的微型电脑板来一次完成全部功能。但是无奈作者一个都没玩过，所以暂时无能为力，日后等我玩会了，一定补上。


windows下通信:

```
/***********************************分割线大人******主程序内*********************
//初始化串口
//这段放入main()即可
   Serial XBEE;//声明串口
   XBEE.SerialOpen("COM9",GENERIC_WRITE);//串口号和读写许可
   XBEE.MessageParam();//设定串口
   XBEE.CommTimeOut();


//这段用于发送数据，先发送x轴坐标，再发送轴y坐标，最后发送一个0提示下位机发送完毕。
XBEE.SerialWrite(xdistance,ydistance,0);


/***********************************分割线君*****serial.h ******************
#ifndef SERIAL_H
#define SERIAL_H


#include <windows.h>//调用window驱动库
#include <string>
#include <iostream>


using namespace std;


class Serial
{
	public:
		void SerialOpen(LPCSTR, DWORD);//打开串口函数
		void MessageParam();//消息函数
		void CommTimeOut();
		void SerialWrite(char);//发送一个CHAR变量到串口的函数
		void SerialWrite(int, int, int);//发送3个INT变量到串口的函数
		string SerialRead();//读取串口并将结构存入一个STRING
		HANDLE getHandle();	//返回句柄


	private:
		HANDLE HAND;
		DCB PARAMS;
		COMMTIMEOUTS timeouts;
};


#endif
//**********************************分割线酱******************************


//**********************************分割线大小姐*****serial.cpp*************************
#include "Serial.h"


void Serial::SerialOpen(LPCSTR port, DWORD func)									//创建一个串口连接
{
	HAND=CreateFile(port, func, 0, 0, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, 0);		//Opens serial port and assigns a Handle
   if(HAND==INVALID_HANDLE_VALUE)													//测试连接是否接通
   {
       if(GetLastError()==ERROR_FILE_NOT_FOUND)
       {
           cout<<"failed 1"<<endl;													//如果串口连接不存在，报错
       }
       cout<<"failed 2"<<endl;														//如果其他问题导致串口连接不正常，报错
   }
}


void Serial::MessageParam()															//设置串口
{
	DCB param = {0};
	PARAMS = param;			//将串口设定存档
	PARAMS.DCBlength=sizeof(PARAMS);
   if (!GetCommState(HAND, &PARAMS))
   {
       cout<<"failed 3"<<endl;			//无法读取串口设定，报错
   }
   PARAMS.BaudRate=CBR_115200;		//设定波特率 115200
   PARAMS.ByteSize=8;				//设定字节长度为8
   PARAMS.StopBits=ONESTOPBIT;		//设定停止位
   PARAMS.Parity=NOPARITY;			//奇偶校验位
   if(HAND==INVALID_HANDLE_VALUE)
   {
       if(GetLastError()==ERROR_FILE_NOT_FOUND)
       {
           cout<<"failed 4"<<endl;			//无法找到串口，报错.
       }
       cout<<"failed 5"<<endl;			//其他错误导致串口不正常，报错
   }
}


void Serial::CommTimeOut()			//设定断线
{
	COMMTIMEOUTS time = {0};
	timeouts=time;				//在断线时的设定
timeouts.ReadIntervalTimeout=0;
   timeouts.ReadTotalTimeoutConstant=0;
   timeouts.ReadTotalTimeoutMultiplier=0;
   timeouts.WriteTotalTimeoutConstant=0;
   timeouts.WriteTotalTimeoutMultiplier=0;


   if(!SetCommTimeouts(HAND, &timeouts))
   {
       cout<<"failed 6"<<endl;				//无法读取，报错
   }
}


void Serial::SerialWrite(char command)		//向串口写入一个char
{
	DWORD dwWritten;
	char trans[1];
	trans[0]=command;
	if(!WriteFile(HAND, trans, sizeof(trans), &dwWritten, NULL))
	{
		cout<<"failed 7"<<endl;
	}
}


void Serial::SerialWrite(int left, int right, int water)		//向串口写入三个int
{
	DWORD dwWritten;
	char trans[1];
	trans[0]=left;
	if(!WriteFile(HAND, trans, sizeof(trans), &dwWritten, NULL))					//写入第1个INT
	{
		cout<<"failed 8"<<endl;
	}
	trans[0]=right;
	if(!WriteFile(HAND, trans, sizeof(trans), &dwWritten, NULL))					//写入第2个INT
	{
		cout<<"failed 9"<<endl;
	}
	trans[0]=water;
	if(!WriteFile(HAND, trans, sizeof(trans), &dwWritten, NULL))					//写入第3个INT
	{
		cout<<"failed 10"<<endl;
	}
}


string Serial::SerialRead()			//从串口读出数据
{
	string RX;				//声明一个变量用来储存读入的数据
   char temp[2]={'0'};
   DWORD dwRead;
	for(int d=0; d<85; d++)		//保持读入数据直到达到最大值
	{
		if(!ReadFile(HAND, temp, 1, &dwRead, NULL))									//读入一个byte
		{
			cout<<"failed 11"<<endl;
		}
		if(temp[0] >' ')
		{
			RX+=temp[0];			//放入RX中
		}
		else
		{
			d=100;				//如果读入一个空格则停止读入
		}
	}
	return RX;
}



HANDLE Serial::getHandle()			//返回句柄
{
	return HAND;
}
//**********************************分割线OTAKU******************************

```


###第四部分：发射机构。


常见的发射机构有三种：气压，弹簧，加速轮。

**第一种是气压发射机构。**

它使用：
```
一个高压气瓶作为发射动力。
一截软管作为导气通道。
一根铜管（或PVC管）作为炮管。
一个电磁阀作为扳机。
一个自行车轮胎气嘴作为加气口。
另需要若干接头把各部分连起来。
```
之间的各种缝隙可以使用从热熔胶到汽车补胎胶的各种胶水处理。
最后你需要一个自行车打气筒（最好带压力计）来给气瓶充气。

如下图：

![](http://doask.qiniudn.com/shaojiepao3.png)

![](http://doask.qiniudn.com/shaojiepao4.png)

这种发射装置的好处是出力大，射程远，口径可变，对弹药几乎没限制，作者用这个机构发射nerf弹，30米射程问题不大。
缺点也很明显，一次发射后需要大约30秒-2分钟重新灌气。而且基本每个气压发射器都会漏气，而处理漏气的办法要嘛不可靠，要吗很贵。作者用的是汽车用那种灌入轮胎内部的补胎胶。但是还是会多少漏点气。另外充气也是很麻烦的一件事，一个自行车气筒还是挺重挺大的。如果用电动气泵的话很容易弄爆，千万不要用。一般充到3个大气压就足够完成射击了，充多不安全。保存时记得将气放完。

**第二种是弹簧发射机构。**

这类基本都是通过拆解一把NERF玩具枪来完成的。NERF玩具枪种类繁多，作者就不介绍了。发射子弹一般只需要两个动作，上膛和抠扳机。这两个动作各使用一个舵机就好了。控制非常简单。
这种发射装置的优点是弹药量大，通常都有6发以上。装填速度高，只要把子弹塞回去就好。
缺点也很明显，子弹速度慢，射程短，子弹也仅限制于NERF弹。

![](http://doask.qiniudn.com/shaojiepao5.png)
>Nerf家族不完全全家福

**第三种是加速轮发射机构。**

如图：
![](http://doask.qiniudn.com/shaojiepao6.png)
![](http://doask.qiniudn.com/shaojiepao7.png)

作者这款拆自一款叫attacknid的遥控玩具。他带了一个12发的弹鼓。用两个小电机高速旋转，一左一右将子弹加速投掷出来。如果不想花钱买而自己仿制也可以简单。我们只需要2个电机负责给子弹加速，再用一个电机或螺杆不停的将弹药往发射电机推过去就行。

自制这种发射机构的优点是弹药量受限制小，口径可变，弹药外形只要是圆柱形就行，发射距离中，装填速度高，可以通过PWM调节发射电机转速控制出膛速度。

结构大致如下：

![](http://doask.qiniudn.com/shaojiepao8.png)

缺点是买的话不便宜，做的话需要比其他两种办法多花点时间，而且在电池没电时非常蛋疼。

![](http://doask.qiniudn.com/shaojiepao9.png)
>attacknind 遥控蜘蛛。

各种发射机构使用常见配置时横向比较：

|     | 气压| 弹射 | 电机加速 |
| -- | -- | -- | -- |
| 弹夹 | 小 | 中-大 | 没啥限制 |
| 射速 | 超级慢 | 慢 | 快-极快 |
| 射程 | 远 | 中 | 中 |
| 装填速度 | 慢| 快 | 快 |
| 弹药限制 | 小 | 只能用NERF | 小 |
| 自制难易度 | 中 | 中 | 难 |
| 自制价格 | 低 | 中 | 中 |
| 命中精度 | 高 | 糟糕 | 高 |
| 属性伤害 | 无 | 无 | 无 |
| 暴击率 | 0.5% | -10% | 0% |
| 暴击伤害加成 | 1% | -10% | 0% |

当然我知道有人会做磁轨发射或者线圈发射，不过那些太危险了。本文不讨论。

###第五部分：云台控制
云台电机数来数去也就只有三种可能：直流，步进，舵机。

这三种电机的程序电路已经在无数地方出现了无数次，所以这里只比较他们的优劣，不讨论如果使用。下表只作用于通常情况。

|  | 直流电机 | 减速直流电机 | 步进电机 | 舵机 |
| -- | -- | -- | -- | -- |
| 速度 | 高 | 低 | 中-高 | 中-高 |
| 力矩 | 中 | 中 | 高 | 高 |
| 精度 | 低 | 高 | 高 | 糟糕 |
| 电路复杂度 | 低 | 低 | 高 | 非常低 |
| 程序复杂度 | 低 | 低 | 高 | 非常低 |
| 价格 | 高 | 中 | 中 | 中 |


谢谢您看完全文。作者的哨戒炮项目因比赛内容变更被砍。最终只做出一款样机（感谢队友Ryan和Aaron）

![](http://doask.qiniudn.com/shaojiepao10.png)

![](http://doask.qiniudn.com/shaojiepao11.png)
