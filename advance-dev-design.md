# 生成设计-阅读笔记之一


>俺是Lee苦海，是Processing   QQ群（群号7726768）的管理员之一。这个连载是应OPENBOOK总编大人约稿而开始的。资料来源是processing神书之一：《Generative Design: Visualize, Program, and Create with Processing》。

俺将尽量在不侵犯原文版权的状态下表达原书的技法和思想（因为原书的层次很高，表达不到位请海涵）。有条件的同学最好还是买一本英文版。这里是[购买链接](http://www.amazon.cn/Generative-Design-Visualize-Program-and-Create-with-Processing-Bohnacker-Hartmut/dp/1616890770)书中的程序在官方网站均有下载，这里是[链接](http://www.generative-gestaltung.de/code)。

在此感谢Arduino群和Processing群众神，并特别感谢_ryeCache大神，没有他们的帮组，我绝不敢贸然开启这么大的项目。

本文中的程序有些可能有细微的错误无法运行，这是为了避免熊孩子无脑抄袭交作业而设计的（其实是我没有审稿），如果你有一点点基础知识，或者去社区问问，这些都是很容易解决的。


###1.0纯爷们的梳妆间

1600多平米的梳妆间
以俺悲剧的语言能力，俺对颜色的语言表达只限于黑、棕、红、橙、黄、绿、蓝、紫、灰、白、金、银等12种颜色（其实就是电阻色环上那几个色）。可是在processing的世界里，俺可以用processing表达的颜色一共有16777216种。有没有一种瞬间从战5渣瞬间变成超级赛亚人的感觉？俺现在就教你一套从天而降的掌法，让你从此甩掉老师，当上学霸，迎娶系花，走向人生巅峰。

本门武功不教内功只教招式。只可卖艺不可踢馆。出门在外被人揍了千万别说是跟我学的。

下面咱直接开始：

```
void setup()
{
size (720,720);       		//建立一个长720个像素，宽720个像素的
noCursor();         			//去掉鼠标

}

void draw()
{
colorMode(HSB,360,100,100);    	//将使用的色彩空间设定为HSB
          				//并给HSB三轴赋值定义区间
         				 //例如360 就是说H(hue)只能是0-360之间的正数


noStroke();
background(mouseY/2,100,100);    	 //背景颜色由鼠标Y轴坐标决定

fill(360-mouseY/2, 100,100);    	//方块颜色也由鼠标Y轴决定，不过和上面的颜色不一样，
//这里是逆序的，因为“360-”
//“/2”是因为窗口大小为720，除2刚好是360.

rect(0,0, 720, mouseX+1);  		//画一方长方形
          				//长方形的高度由鼠标的X轴决定


}
```

把上面这段程序复制到Processing里，运行一下。你可以看到窗口被一分为二。分割线和颜色都随着鼠标的坐标变化而变化。这个程序就是使用鼠标的XY轴坐标作为输入变量，来控制颜色和方块面积。在接下去的连载中，你会发现各种类似的手法。


####1.1爷们的脂粉盒
在刚刚的程序中，你也许觉得奇怪，为什么鼠标移到顶部和底部都是全红？其实这是HSB色彩空间的特色。勤快的同学估计已经谷哥了HSB，懒也无所谓，原书刚好有提到色彩体系，我也就顺水推舟介绍下吧。下面这个程序使用了HSB色彩空间的H轴和S轴，并展开成一个2D平米来演示HSB色彩空间的颜色分布。
```
/*程序功能说明
画出HSB色彩空间的颜色
*/


int stepX =2;
int stepY =2;

void setup(){
  size(800, 400);          //窗口大小
  background(0);
}

void draw(){


  noStroke();
  colorMode(HSB, width, height, 100);    //因为之前把窗口大小定义为了800*400像素
              //所以这里width = 800
              //height = 400

  for (int gridY=0; gridY<height; gridY+=stepY){  //两个for循环对应窗口的X轴和Y轴
    for (int gridX=0; gridX<width; gridX+=stepX){
      fill(gridX, height-gridY, 100);    //根据for循环中的grid变换不同的颜色
      rect(gridX, gridY, stepX, stepY);    //根据grid变换色块尺寸
    }
  }

}
```
将上面的程序复制进Processing中，点击运行。你可以发现颜色的变化几乎是连续的，如图：

![](http://doask.qiniudn.com/sheji1.png)

这是因为在最上角时，每个色块的大小只有4（2\*2）个像素。在这个条件下，眼睛已经没法分辨出色块了。更因为单个色块只有4个像素大，所以800\*400的窗口中一共显色了80000种颜色（800\*400/4）。虽说距离之间所说的超过1600万种颜色还有很大一段距离，但是通过这个程序，你应该已经大致理解了HSB色彩空间的模式。它的最大一个特点就是可以通过变换H轴来完成多种色彩的变换。

如果你不理解这的意义，我来说明一下：如果你使用RGB色彩空间，你必需要同时控制三个轴来完成一个常见的颜色。而在生成设计中，你可能没有三个输入参数。而在同样情况下，HSB色彩空间只要一个轴就能得到红橙黄绿蓝紫和它们的过度颜色。所以同样在只要有一个输入参数的情况下，使用HSB色彩空间比使用RGB色彩空间有更宽的选择。

下面这个例子就是利用HSB这一特征来完成一个图像。

```
int segmentCount = 360;
int radius = 300;

void setup(){
  size(800, 800);
}

void draw(){

  noStroke();
  colorMode(HSB, 360, 100, 100);
  background(360);

  float angleStep = 360/segmentCount;    //根据色块的数量 计算每个色块的角度

  beginShape(TRIANGLE_FAN);        //画一个图形TRIANGLE_FAN
  vertex(width/2, height/2);        //图形的开始坐标
  for (float angle=0; angle<=360; angle+=angleStep){
//360度走一圈


float vx = width/2 + cos(radians(angle))*radius;
              //计算色块的在圆外侧的坐标
float vy = height/2 + sin(radians(angle))*radius;


vertex(vx, vy);  //将色块在圆外侧的坐标设为TRIANGLE_FAN           //的角点
fill(angle, 100, 100);      //填上颜色
              //角度为H
              //鼠标X坐标为色饱和度
              //鼠标Y坐标为明度
  }
  endShape();            //图形TRIANGLE_FAN到此结束


}

void keyReleased(){
  if(key >='0' && key <= '8')
segmentCount  = int(pow(2,key-'0')+2);

}

```

在上面这个程序中，圆的角度作为控制HSB色彩空间中H的参数。至于为嘛在360色块时有很好看的花纹,那是没有对图像做柔化/去锯齿的结果。另外，按下数字键0-8的话，图像会变化哦！

在按下数字8后，在你就得到了这个图像：

![](http://doask.qiniudn.com/sheji2.png)

不难发现，这和上个程序中产生的图非常类似。没错，其实就是把它叠成个圆形而已。同时，我们看到另一个事，H轴其实是连续的。在前边程序显示的图中,我们就发现最左侧和最右侧都是红色。而在这个圆饼形状的图中，我们可以看到其实整个H轴是一个循环的。这也是HSB色彩空间的特色之一。利用这种特色，我们可以放心的将各种数据映射到H轴上，而完全不用担心色彩会在某些情况下出现跳变的现象。

![](http://doask.qiniudn.com/sheji3.png)


####1.2独家特色
不管是什么色彩空间，它能表达的颜色都在已知的1600多种颜色之中。可是那么多种颜色中，有大便色也有土豪金，找到一个自己喜欢颜色犹如大海捞飞机，详见MH370。该如何下手呢？

```
// P_1_2_1_01.pde
//
// Generative Gestaltung, ISBN: 978-3-87439-759-9
// First Edition, Hermann Schmidt, Mainz, 2009
// Hartmut Bohnacker, Benedikt Gross, Julia Laub, Claudius Lazzeroni
// Copyright 2009 Hartmut Bohnacker, Benedikt Gross, Julia Laub, Claudius Lazzeroni
//
// http://www.generative-gestaltung.de
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.

/**
 * shows how to interpolate colors in different styles/ color modes
 *
 * MOUSE
 * left click          : new random color set
 * position x          : interpolation resolution
 * position y          : row count
 *
 * KEYS
 * 1-2                 : switch interpolation style
 * s                   : save png
 * p                   : save pdf
 */

 ///*程序功能说明
演示不同色彩空间的特点并做一个调色盘

鼠标：
鼠标按键		：生成新的颜色
鼠标x坐标   		：调色均匀度
鼠标y坐标		：调色盘数量

键盘：
1-2			：改变区间数量
S			:保存成图像文件
P			:保存成PDF文件
*///




import processing.pdf.*;
import java.util.Calendar;

boolean savePDF = false;

int tileCountX = 2;
int tileCountY = 10;

color[] colorsLeft = new color[tileCountY];
color[] colorsRight = new color[tileCountY];
color[] colors;

boolean interpolateShortest = true;


void setup() {
  size(800, 800);
  colorMode(HSB,360,100,100,100); 			//依然HSB空间
							//HSB取值空间依次为 0-360 0-100 0-100
							//最后一个100是Alpha通道

/****************************************************************
Alpha通道是一个8位的灰度通道，该通道用256级灰度来记录图像中的透明度信息，定义透明、不透明和半透明区域，其中黑表示全透明，白表示不透明，灰表示半透明。
————泰勒脑残夫斯基（大神）

******************************************************************/
  noStroke();
  shakeColors();
}


void draw() {
  if (savePDF) {
    beginRecord(PDF, timestamp()+".pdf");
    noStroke();
    colorMode(HSB,360,100,100,100);
  }

  tileCountX = (int) map(mouseX,0,width,2,100); 	//map是个等比变换函数，它将mouseX在0到
 							//width之间的位置，等比映射到2到100这
//个区间中
  tileCountY = (int) map(mouseY,0,height,2,10);	//这两句将鼠标的XY轴坐标作为参数，用于
//变换色盘的数量和渐变的次数

  float tileWidth = width / (float)tileCountX;
  float tileHeight = height / (float)tileCountY;
  color interCol;

  // just for ase export
  colors = new color[tileCountX*tileCountY];
  int i = 0;

  for (int gridY=0; gridY< tileCountY; gridY++) {	//使用一个FOR循环，一行一行的绘制图像
    color col1 = colorsLeft[gridY];			//确定本行左侧的颜色
    color col2 = colorsRight[gridY];			//确定本行右侧的颜色

    for (int gridX=0; gridX< tileCountX ; gridX++) {
      float amount = map(gridX,0,tileCountX-1,0,1);
							//计算调色盘的色块数量

      if (interpolateShortest) {			//如果interpolateShortest为真，
//使用RGB色系渐变
        // switch to rgb
        colorMode(RGB,255,255,255,255);
        interCol = lerpColor(col1,col2, amount); 	// lerpColor函数可以将两个颜色之间颜色
//计算出来，col1，col2分别是起点和终点，
//amount是之间的位置/
        // switch back
        colorMode(HSB,360,100,100,100);
      }
      else {						//反之，使用HSB色系渐变
        interCol = lerpColor(col1,col2, amount);
      }
      fill(interCol);					//填充色块

      float posX = tileWidth*gridX;
      float posY = tileHeight*gridY;
      rect(posX, posY, tileWidth, tileHeight);

      // just for ase export
      colors[i] = interCol;
      i++;
    }
  }

  if (savePDF) {
    savePDF = false;
    endRecord();
  }
}


void shakeColors() {					//新颜色生成函数
							//这里将颜色设定在某个特定区间，如果想
							//试试其他颜色，改变（0，60）和
//（160，190）两个括号中的数字即可
  for (int i=0; i<tileCountY; i++) {
    colorsLeft[i] = color(random(0,60), random(0,100), 100);
    colorsRight[i] = color(random(160,190), 100, random(0,100));
  }
}


void mouseReleased() {
  shakeColors(); 					//变换颜色
}


void keyReleased() {
  if (key == 's' || key == 'S') saveFrame(timestamp()+"_##.png");
  if (key == 'p' || key == 'P') savePDF = true;

  if (key == '1') interpolateShortest = true;
  if (key == '2') interpolateShortest = false;
}

// timestamp
String timestamp() {
  Calendar now = Calendar.getInstance();
  return String.format("%1$ty%1$tm%1$td_%1$tH%1$tM%1$tS", now);
}

```

将上面的程序复制进Processing中，运行，结果类似如下：

![](http://doask.qiniudn.com/sheji4.png)

原程序中本带有一个保存颜色的功能，但是因为涉及库的安装，恐新手不适，我就去掉了。不过各位还是可以通过截屏或者是QQ截图的办法（ctrl+alt+a）来取得颜色。并无大碍。

在运行时，点击鼠标可以变换新的颜色。另外，你还可以修改shakeColors()中color的随机范围来改变随机颜色的分布区间（或者变成你指定的颜色区间）。

总之相信在这个调色盘的帮组下，你在Processing中的颜色把握会比之前更好。

####1.3什么？你找不到自己的颜色？拿咱先借点颜色来用吧
首先，我们需要一张图。俺觉得这张颜色不错，就用了，如下：

![](http://doask.qiniudn.com/sheji5.png)

下面是程序：

```
// P_1_2_2_01.pde
//
// Generative Gestaltung, ISBN: 978-3-87439-759-9
// First Edition, Hermann Schmidt, Mainz, 2009
// Hartmut Bohnacker, Benedikt Gross, Julia Laub, Claudius Lazzeroni
// Copyright 2009 Hartmut Bohnacker, Benedikt Gross, Julia Laub, Claudius Lazzeroni
//
// http://www.generative-gestaltung.de
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.


///*程序功能说明
取出一副图像中的颜色

鼠标：
鼠标x坐标   		：色块数量

键盘：
S			:保存成图像文件
P			:保存成PDF文件
*///


//import generativedesign.*;
import processing.pdf.*;
import java.util.Calendar;

boolean savePDF = false;

PImage img;
color[] colors;

String sortMode = null;



void setup(){
  size(470, 312);
  colorMode(HSB, 360, 100, 100, 100);
  noStroke();
  noCursor();
  img = loadImage("pic1.jpg"); 			//读取图像，请注意你准备的图像名字
//是否和程序中一致
//图像需要和程序放在同个目录下
}


void draw(){
  if (savePDF) {
    beginRecord(PDF, timestamp()+".pdf");
    colorMode(HSB, 360, 100, 100, 100);
    noStroke();
  }

  int tileCount = width / max(mouseX, 5);		//用鼠标X轴坐标控制色块数量
  float rectSize = width / float(tileCount);	//根据数量计算单个方块大小

  // get colors from image
  int i = 0;
  colors = new color[tileCount*tileCount];
  for (int gridY=0; gridY<tileCount; gridY++) {	//两个FOR循环针对素材图像的XY轴
    for (int gridX=0; gridX<tileCount; gridX++) {
      int px = (int) (gridX * rectSize);
      int py = (int) (gridY * rectSize);
      colors[i] = img.get(px, py);			//取出颜色
      i++;
    }
  }

  // sort colors
  //if (sortMode != null) colors = GenerativeDesign.sortColors(this, colors, sortMode);


  // draw grid						//将取出的颜色画出来
  i = 0;
  for (int gridY=0; gridY<tileCount; gridY++) {
    for (int gridX=0; gridX<tileCount; gridX++) {
      fill(colors[i]);
      rect(gridX*rectSize, gridY*rectSize, rectSize, rectSize);
      i++;
    }
  }

  if (savePDF) {
    savePDF = false;
    endRecord();
  }
}


void keyReleased(){					//这段因为库的限制基本没用
 // if (key=='c' || key=='C') GenerativeDesign.saveASE(this, colors, timestamp()+".ase");
  if (key=='s' || key=='S') saveFrame(timestamp()+"_##.png");
  if (key=='p' || key=='P') savePDF = true;

  if (key == '1') img = loadImage("pic1.jpg");
  //if (key == '2') img = loadImage("pic2.jpg");
  //if (key == '3') img = loadImage("pic3.jpg");

  //if (key == '4') sortMode = null;
  //if (key == '5') sortMode = GenerativeDesign.HUE;
  //if (key == '6') sortMode = GenerativeDesign.SATURATION;
  //if (key == '7') sortMode = GenerativeDesign.BRIGHTNESS;
  //if (key == '8') sortMode = GenerativeDesign.GRAYSCALE;
}


// timestamp
String timestamp() {
  Calendar now = Calendar.getInstance();
  return String.format("%1$ty%1$tm%1$td_%1$tH%1$tM%1$tS", now);
}

```
将上面的程序复制进Processing中，运行结果如下：

![](http://doask.qiniudn.com/sheji6.png)

![](http://doask.qiniudn.com/sheji7.png)

色块的大小会随着鼠标的移动而变化。因为原图的色彩就已经不错，所以我们不需要再想大海捞飞机一样从全部颜色中找出好看的颜色。而是像直接走进精品店，从琳琅满目的商品中调几样回家。像是这幅图中的金色和蓝色都不错的说。

####1.4花格子布
找颜色的最终目标还是生成图案。这里简单的使用随机数和HSB色彩空间，将生成出的几个随机颜色有规律的排列起来，得到一种花格子布的效果，如图：

![](http://doask.qiniudn.com/sheji8.png)

它的思路很简单。假设我有三种颜色，它们的名字为ABC。如果将它们用下面的规律排列起来，如：

```
ABCABCABCABCABCABCABCABCABCABCABCABC
BCABCABCABCABCABCABCABCABCABCABCABCA
CABCABCABCABCABCABCABCABCABCABCABCAB
ABCABCABCABCABCABCABCABCABCABCABCABC
BCABCABCABCABCABCABCABCABCABCABCABCA
CABCABCABCABCABCABCABCABCABCABCABCAB
ABCABCABCABCABCABCABCABCABCABCABCABC
BCABCABCABCABCABCABCABCABCABCABCABCA
CABCABCABCABCABCABCABCABCABCABCABCAB
```

或者类似的办法排列起来，就是本小节的目标，程序如下：

```
// P_1_2_3_01.pde
//
// Generative Gestaltung, ISBN: 978-3-87439-759-9
// First Edition, Hermann Schmidt, Mainz, 2009
// Hartmut Bohnacker, Benedikt Gross, Julia Laub, Claudius Lazzeroni
// Copyright 2009 Hartmut Bohnacker, Benedikt Gross, Julia Laub, Claudius Lazzeroni
//
// http://www.generative-gestaltung.de
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.

///*程序功能说明
通过特定模式排列随机颜色

鼠标：
鼠标XY坐标   		：色块的XY轴数量


键盘：
0-9			：使用不同的规则生成颜色
S			:保存成图像文件
P			:保存成PDF文件
*///




//import generativedesign.*;
import processing.pdf.*;
import java.util.Calendar;

boolean savePDF = false;

int tileCountX = 50;						//最多50*10个色块
int tileCountY = 10;

// arrays for color components values
int[] hueValues = new int[tileCountX];
int[] saturationValues = new int[tileCountX];
int[] brightnessValues = new int[tileCountX];


void setup() {
  size(800,800);
  colorMode(HSB,360,100,100,100);				//HSB正在大杀特杀
  noStroke();

  // init with random values
  for (int i=0; i<tileCountX; i++) {				//随机生成若干颜色
    hueValues[i] = (int) random(0,360);			//随机生成H
    saturationValues[i] = (int) random(0,100);		//随机生成S
    brightnessValues[i] = (int) random(0,100);		//随机生成B
  }
}


void draw() {
  if (savePDF) {
    beginRecord(PDF, timestamp()+".pdf");
    noStroke();
    colorMode(HSB,360,100,100,100);
  }
  // white back
  background(0,0,100);

  // count every tile
  int counter = 0;

  // map mouse to grid resolution				//还是一样的用鼠标控制色块大小
  int currentTileCountX = (int) map(mouseX, 0,width, 1,tileCountX);
  int currentTileCountY = (int) map(mouseY, 0,height, 1,tileCountY);
  float tileWidth = width / (float) currentTileCountX;
  float tileHeight = height / (float) currentTileCountY;

  for (int gridY=0; gridY< tileCountY; gridY++) {		//依然还是两个FOR，分别XY轴
    for (int gridX=0; gridX< tileCountX; gridX++) {
      float posX = tileWidth*gridX;
      float posY = tileHeight*gridY;
      int index = counter % currentTileCountX;

      // get component color values
      fill(hueValues[index],saturationValues[index],brightnessValues[index]);
								//将随机产生的颜色填充进色块
      rect(posX, posY, tileWidth, tileHeight);		//根据鼠标参数画出色块
      counter++;
    }
  }

  if (savePDF) {
    savePDF = false;
    endRecord();
  }
}


void keyReleased() {
  if (key == 's' || key == 'S') saveFrame(timestamp()+"_##.png");
  if (key == 'p' || key == 'P') savePDF = true;
  if (key == 'c' || key == 'C') {
    // ------ save an ase file (adobe swatch export) ------
    // create palette
    color[] colors = new color[hueValues.length];
    for (int i=0; i<hueValues.length; i++) {
      colors[i] = color(hueValues[i],saturationValues[i],brightnessValues[i]);
								//将HSB的值整合为一个颜色
    }
    //GenerativeDesign.saveASE(this, colors, timestamp()+".ase");
  }

  if (key == '1') {						//按键1->完全随机的HSB值
    for (int i=0; i<tileCountX; i++) {
      hueValues[i] = (int) random(0,360);
      saturationValues[i] = (int) random(0,100);
      brightnessValues[i] = (int) random(0,100);
    }
  }
  if (key == '2') { 						//按键2->B值固定为100，
//HS完全随机
    for (int i=0; i<tileCountX; i++) {
      hueValues[i] = (int) random(0,360);
      saturationValues[i] = (int) random(0,100);
      brightnessValues[i] = 100;
    }
  }
  if (key == '3') {  						//按键3->S值固定为100
								//HB完全随机
    for (int i=0; i<tileCountX; i++) {
      hueValues[i] = (int) random(0,360);
      saturationValues[i] = 100;
      brightnessValues[i] = (int) random(0,100);
    }
  }

  if (key == '4') {  						//下面的大家应该都看明白了
    for (int i=0; i<tileCountX; i++) {
      hueValues[i] = 0;
      saturationValues[i] = 0;
      brightnessValues[i] = (int) random(0,100);
    }
  }
  if (key == '5') {
    for (int i=0; i<tileCountX; i++) {
      hueValues[i] = 195;
      saturationValues[i] = 100;
      brightnessValues[i] = (int) random(0,100);
    }
  }
  if (key == '6') {
    for (int i=0; i<tileCountX; i++) {
      hueValues[i] = 195;
      saturationValues[i] = (int) random(0,100);
      brightnessValues[i] = 100;
    }
  }

  if (key == '7') {
    for (int i=0; i<tileCountX; i++) {
      hueValues[i] = (int) random(0,180);
      saturationValues[i] = (int) random(80,100);
      brightnessValues[i] = (int) random(50,90);
    }
  }
  if (key == '8') {
    for (int i=0; i<tileCountX; i++) {
      hueValues[i] = (int) random(180,360);
      saturationValues[i] = (int) random(80,100);
      brightnessValues[i] = (int) random(50,90);
    }
  }

  if (key == '9') {
    for (int i=0; i<tileCountX; i++) {
      if (i%2 == 0) {
        hueValues[i] = (int) random(0,360);
        saturationValues[i] = 100;
        brightnessValues[i] = (int) random(0,100);
      }
      else {
        hueValues[i] = 195;
        saturationValues[i] = (int) random(0,100);
        brightnessValues[i] = 100;
      }
    }
  }
  if (key == '0') {
    for (int i=0; i<tileCountX; i++) {
      if (i%2 == 0) {
        hueValues[i] = (int) 192;
        saturationValues[i] = (int) random(0,100);
        brightnessValues[i] = (int) random(10,100);
      }
      else {
        hueValues[i] = 273;
        saturationValues[i] = (int) random(0,100);
        brightnessValues[i] = (int) random(10,90);
      }
    }
  }

}


// timestamp
String timestamp() {
  Calendar now = Calendar.getInstance();
  return String.format("%1$ty%1$tm%1$td_%1$tH%1$tM%1$tS", now);
}


```

将程序复制进processing，点击运行。可以使用键盘上的数字0-9改变随意颜色的生成规则。原作者很好心的将各种好玩的颜色的生成规则列出来了，省去了读者自己改参数学习的时间。

已经写了不少了，只用颜色能玩的花样也差不多就这么多。这期到此结束。

注：原程序中有使用一个名为Generative Design Library的库。在没有库时该程序无法运行。同时这个库并不在Processing自带的库中，需要另外下载安装。为了避免初学者遇到各种装库带来的问题，俺直接把相关的程序片段删掉了。如果你想使用Generative Design Library带来的功能，请前往http://www.generative-gestaltung.de/code#library下载。库文件的安装教程请前往http://www.learningprocessing.com/tutorials/libraries/。

>扩展阅读：
Processing 官方网站：http://www.processing.org/。
个人比较喜欢的Pocessing入门书籍Nature of Code在线阅读：http://natureofcode.com/ （里边虽然有写10美金，但是你可以不给钱直接看书，在网站上看还带演示）。
Processing常用函数查询：http://www.processing.org/reference/。

下期预告：纯爷们男大72变
