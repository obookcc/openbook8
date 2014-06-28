# 基于arduino的放生萤火虫制作

>南京创客空间  沈金鑫  陈大庆

###1 引言

笔者的出生和成长都在农村，依稀记得小的时候捉萤火虫的事儿。每当夏季，小伙伴们都可以在草丛里捉到很多的萤火虫，把它们放在玻璃罐里，黄绿色的光很是好看，如图1所示。为了读大学来到城市，毕业之后便在城市定居，却发现很少有机会能够看到萤火虫，很难再有小时候的乐趣了。想必很多的读者也都怀念小时候捉萤火虫的乐趣，下面教大家如何使用Arduino来制作一个电子萤火虫，重新找回儿时的快乐。

![](http://doask.qiniudn.com/yhc1.jpg)
>图1 发光的萤火虫 From [wordpress](http://wordpress.com)

本文采用Arduino NANO控制器、LED灯来实现仿生萤火虫，Arduino控制器控制4个翠绿色的LED灯以亮灭随机、亮度随机、顺序随机、时间随机的模式来模仿自然界中萤火虫的习性。
###2 原理及材料

仿生萤火虫的系统原理图如图2所示，4个翠绿色的LED灯的正极依次连接至Arduino NANO控制器的数字端口D3、D5、D6、D9上，因为Arduino NANO控制器的只有数字端口3、5、 6、9、10和11具有PWM输出功能，可以实现LED灯亮度的调节；4个LED灯的负极通过排针连接至Arduino NANO控制器的GND引脚上；另外，通过4个AA电池或者2个CR2032纽扣电池为Arduino NANO控制器及4个LED灯提供工作电压。
需要注意的是，在实际制作中使用的是翠绿色的LED灯，而不是图2所示的红色LED灯，因为萤火虫的翠绿色的灯光更加逼真。

![](http://doask.qiniudn.com/yhc2.png)
>图2 仿生萤火虫Arduino原理图

图3列出了制作仿生萤火虫所需要的材料及数量，按需备齐即可开始下面的制作。
![](http://doask.qiniudn.com/yhc3.jpg)
>图3 仿生萤火虫材料清单

###3 手工制作
**第一步：**将杜邦线从中间剪断并剥线，把LED灯的负极（短引脚）引脚剪去一部分，将热缩管剪成7mm～10mm的小段，如图4所示。

**第二步：**将小段的热缩管套进剥好的杜邦线，每一根杜邦线上套一个热缩管段，如图5所示。

![](http://doask.qiniudn.com/yhc4.jpg)
>图4 前期准备

![](http://doask.qiniudn.com/yhc5.jpg)
>图5 将热缩管套进杜邦线上

**第三步：**使用电烙铁或焊台将黑色杜邦线与LED灯的负极焊接在一起，【注意：在焊接时不要使用过多的焊锡】；将排针上面用焊锡连接起来并焊接至蓝色的杜邦线，如图6所示。

**第四步：**使用尖嘴钳将LED灯的正极引脚剪去，并使用电烙铁或焊台焊接至红色的杜邦线，如图所示。

![](http://doask.qiniudn.com/yhc6.jpg)
>图6 焊接好LED灯负极与黑色杜邦线及负极排线

![](http://doask.qiniudn.com/yhc7.jpg)
>图7 焊接好LED灯正极与红色杜邦线

**第五步：**将热风枪的温度调至150℃左右，风量调节至较小风量。将热缩管移至LED引脚与杜邦线的焊接处，并使用热风枪加热热缩管，使热缩管收缩，包住焊接处，如图8所示。经过热风枪加工之后的LED灯，如图9所示。
![](http://doask.qiniudn.com/yhc8.jpg)
>图8 使用热风枪加热热缩管

![](http://doask.qiniudn.com/yhc9.jpg)
>图9 制作完毕的LED灯

![](http://doask.qiniudn.com/yhc10.jpg)
>图10 制作好的LED灯与其他材料

**第六步：**使用电烙铁或焊台将纽扣电池盒的正极（红线）与负极（黑线）焊接至Arduino NANO控制器的5V和GND引脚，如图11所示。

**第七步：**将制作好的LED灯的负极（黑线）连接至含有蓝色杜邦线的排针上，蓝色杜邦线接至Arduino NANO控制器的GND引脚，将LED灯的正极（红线）依次接至Arduino NANO控制器的数字端口D3、D5、D6、D9引脚上，如图12所示。

![](http://doask.qiniudn.com/yhc11.jpg)
>图11 将电池盒焊接至Arduino NANO控制器

![](http://doask.qiniudn.com/yhc12.jpg)
>图12 将LED灯连接至Arduino NANO控制器


###4 Arduino程序设计

**4 .1 数目随机和随机顺序**

首先，通过随机数以获得点亮LED灯的数目，然后再通过随机数以获取LED灯点亮的顺序，并且通过随机数来获得点亮的时间。程序代码如下所示。



```

int randNumber;
int delaytime;

void setup() {
  pinMode(3, OUTPUT);
  pinMode(5, OUTPUT);
  pinMode(6, OUTPUT);
  pinMode(9, OUTPUT);
  Serial.begin(9600);
  randomSeed(analogRead(0));
}
void delay_time(void){
  delaytime = random(1,5);
  switch(delaytime){
  case 1:delay(800);
        break;
  case 2:delay(900);
        break;
  case 3:delay(1000);
        break;
  case 4:delay(1100);
        break;
  }
}

void RandLED_2(void)
{
  randNumber = random(1,5);
  Serial.println(randNumber);
  switch(randNumber)
  {
  case 1: analogWrite(3, 255);
          analogWrite(5, 255);
          delay_time();
          digitalWrite(3, LOW);
          delay_time();
          digitalWrite(5, LOW);
          break;
  case 2: analogWrite(6, 255);
          analogWrite(9, 255);
          delay_time();
          digitalWrite(6, LOW);
          delay_time();
          digitalWrite(9, LOW);
          break;
  case 3: analogWrite(3, 255);
          analogWrite(9, 255);
          delay_time();
          digitalWrite(3, LOW);
          delay_time();
          digitalWrite(9, LOW);
          break;
  case 4: analogWrite(5, 255);
          analogWrite(9, 255);
          delay_time();
          digitalWrite(5, LOW);
          delay_time();
          digitalWrite(9, LOW);
          break;
  case 5: analogWrite(3, 255);
          analogWrite(6, 255);
          delay_time();
          digitalWrite(6, LOW);
          delay_time();
          digitalWrite(3, LOW);
          break;
  case 6: analogWrite(5, 255);
          analogWrite(6, 255);
          delay_time();
          digitalWrite(6, LOW);
          delay_time();
          digitalWrite(5, LOW);
          break;
  }
}
void RandLED_3(void)
{
  randNumber = random(1,5);
  Serial.println(randNumber);
  switch(randNumber)
  {
  case 1: analogWrite(5,255);
          analogWrite(6,255);
          analogWrite(9,255);
          delay_time();
          digitalWrite(5, LOW);
          delay_time();
          digitalWrite(9, LOW);
          delay_time();
          digitalWrite(6, LOW);
          break;
  case 2: analogWrite(6,255);
          analogWrite(3,255);
          analogWrite(9,255);
          delay_time();
          digitalWrite(9, LOW);
          delay_time();
          digitalWrite(3, LOW);
          delay_time();
          digitalWrite(6, LOW);
          break;
  case 3: analogWrite(5,255);
          analogWrite(3,255);
          analogWrite(9,255);
          delay_time();
          digitalWrite(9, LOW);
          delay_time();
          digitalWrite(5, LOW);
          delay_time();
          digitalWrite(3, LOW);
          break;
  case 4: analogWrite(5,255);
          analogWrite(6,255);
          analogWrite(3,255);
          delay_time();
          digitalWrite(3, LOW);
          delay_time();
          digitalWrite(5, LOW);
          delay_time();
          digitalWrite(6, LOW);
          break;
  }
}
void RandLED_4(void)
{
  randNumber = random(1,5);
  Serial.println(randNumber);
  switch(randNumber)
  {
  case 1: analogWrite(3,255);
          analogWrite(5,255);
          analogWrite(6,255);
          analogWrite(9,255);
          delay_time();
          digitalWrite(3, LOW);
          delay_time();
          digitalWrite(5, LOW);
          delay_time();
          digitalWrite(6, LOW);
          delay_time();
          digitalWrite(9, LOW);
          break;
  case 2: analogWrite(3,255);
          analogWrite(5,255);
          analogWrite(6,255);
          analogWrite(9,255);
          delay_time();
          digitalWrite(3, LOW);
          delay_time();
          digitalWrite(6, LOW);
          delay_time();
          digitalWrite(9, LOW);
          delay_time();
          digitalWrite(5, LOW);
          break;
  case 3: analogWrite(3,255);
          analogWrite(5,255);
          analogWrite(6,255);
          analogWrite(9,255);
          delay_time();
          digitalWrite(9, LOW);
          delay_time();
          digitalWrite(3, LOW);
          delay_time();
          digitalWrite(5, LOW);
          delay_time();
          digitalWrite(6, LOW);
          break;
  case 4: analogWrite(3,255);
          analogWrite(5,255);
          analogWrite(6,255);
          analogWrite(9,255);
          delay_time();
          digitalWrite(6, LOW);
          delay_time();
          digitalWrite(9, LOW);
          delay_time();
          digitalWrite(3, LOW);
          delay_time();
          digitalWrite(5, LOW);
          break;
  }
}

void loop() {
  randNumber = random(1,4);
  switch(randNumber){
  case 1:RandLED_2();
        break;
  case 2:RandLED_3();
        break;
  case 3:RandLED_4();
        break;
  }
}

```

**4 .2 添加随机亮度和优化点亮时间**

首先，通过随机数以获得点亮LED灯的数目，然后再通过随机数以获取LED灯的亮度值，将LED灯全部点亮，然后通过随机数来获得点亮的时间按照顺序来依次熄灭LED灯。程序代码如下所示。

```
int randNumber;
int pwm,delaytime;

void setup() {
  pinMode(5, OUTPUT);
  pinMode(6, OUTPUT);
  pinMode(10, OUTPUT);
  pinMode(11, OUTPUT);
  Serial.begin(9600);
  randomSeed(analogRead(0));
}
void delay_time_2(void){
  delaytime = random(1,5);
  switch(delaytime){
  case 1:delay(1000);
        break;
  case 2:delay(1250);
        break;
  case 3:delay(1500);
        break;
  case 4:delay(1250);
        break;
  }
}

void delay_time_3(void){
  delaytime = random(1,5);
  switch(delaytime){
  case 1:delay(1000);
        break;
  case 2:delay(1250);
        break;
  case 3:delay(1500);
        break;
  case 4:delay(1000);
        break;
  }
}

void delay_time_4(void){
  delaytime = random(1,5);
  switch(delaytime){
  case 1:delay(800);
        break;
  case 2:delay(1000);
        break;
  case 3:delay(900);
        break;
  case 4:delay(700);
        break;
  }
}

int pwm_value(void){
  int pwmvalue;
  pwmvalue = random(1,5);
  switch(pwmvalue){
  case 1: return 255 ;
          break;
  case 2: return 180;
          break;
  case 3: return 110;
          break;
  case 4: return 40;
          break;
  }
}
void RandLED_2(void)
{
  randNumber = random(1,5);
  Serial.println(randNumber);
  switch(randNumber)
  {
  case 1: pwm=pwm_value();
          analogWrite(5, pwm);
          analogWrite(6, pwm);
          delay_time_2();
          digitalWrite(5, LOW);
          delay_time_2();
          digitalWrite(6, LOW);
          break;
  case 2: pwm=pwm_value();
          analogWrite(5, pwm);
          analogWrite(10, pwm);
          delay_time_2();
          digitalWrite(5, LOW);
          delay_time_2();
          digitalWrite(10, LOW);
          break;
  case 3: pwm=pwm_value();
          analogWrite(5, pwm);
          analogWrite(11, pwm);
          delay_time_2();
          digitalWrite(5, LOW);
          delay_time_2();
          digitalWrite(11, LOW);
          break;
  case 4: pwm=pwm_value();
          analogWrite(6, pwm);
          analogWrite(10, pwm);
          delay_time_2();
          digitalWrite(6, LOW);
          delay_time_2();
          digitalWrite(10, LOW);
          break;
  case 5: pwm=pwm_value();
          analogWrite(6, pwm);
          analogWrite(11, pwm);
          delay_time_2();
          digitalWrite(6, LOW);
          delay_time_2();
          digitalWrite(11, LOW);
          break;
  case 6: pwm=pwm_value();
         analogWrite(10, pwm);
          analogWrite(11, pwm);
          delay_time_2();
          digitalWrite(6, LOW);
          delay_time_2();
          digitalWrite(11, LOW);
          break;
  }
}
void RandLED_3(void)
{
  randNumber = random(1,5);
  Serial.println(randNumber);
  switch(randNumber)
  {
  case 1: pwm=pwm_value();
          analogWrite(5, pwm);
          analogWrite(6, pwm);
          analogWrite(10, pwm);
          delay_time_3();
          digitalWrite(5, LOW);
          delay_time_3();
          digitalWrite(10, LOW);
          delay_time_3();
          digitalWrite(6, LOW);
          break;
  case 2: pwm=pwm_value();
          analogWrite(6, pwm);
          analogWrite(10, pwm);
          analogWrite(11, pwm);
          delay_time_3();
          digitalWrite(6, LOW);
          delay_time_3();
          digitalWrite(10, LOW);
          delay_time_3();
          digitalWrite(11, LOW);
          break;
  case 3: pwm=pwm_value();
          analogWrite(5, pwm);
          analogWrite(10, pwm);
          analogWrite(11, pwm);
          delay_time_3();
          digitalWrite(10, LOW);
          delay_time_3();
          digitalWrite(5, LOW);
          delay_time_3();
          digitalWrite(11, LOW);
          break;
  case 4: pwm=pwm_value();
          analogWrite(5, pwm);
          analogWrite(6, pwm);
          analogWrite(11, pwm);
          delay_time_3();
          digitalWrite(6, LOW);
          delay_time_3();
          digitalWrite(5, LOW);
          delay_time_3();
          digitalWrite(11, LOW);
          break;
  }
}
void RandLED_4(void)
{
  randNumber = random(1,5);
  Serial.println(randNumber);
  switch(randNumber)
  {
  case 1: pwm=pwm_value();
          analogWrite(5, pwm);
          analogWrite(6, pwm);
          analogWrite(10, pwm);
          analogWrite(11, pwm);
          delay_time_4();
          digitalWrite(6, LOW);
          delay_time_4();
          digitalWrite(10, LOW);
          delay_time_4();
          digitalWrite(5, LOW);
          delay_time_4();
          digitalWrite(11, LOW);
          break;
  case 2: pwm=pwm_value();
          analogWrite(5, pwm);
          analogWrite(6, pwm);
          analogWrite(10, pwm);
          analogWrite(11, pwm);
          delay_time_4();
          digitalWrite(10, LOW);
          delay_time_4();
          digitalWrite(6, LOW);
          delay_time_4();
          digitalWrite(11, LOW);
          delay_time_4();
          digitalWrite(5, LOW);
          break;
  case 3: pwm=pwm_value();
          analogWrite(5, pwm);
          analogWrite(6, pwm);
          analogWrite(10, pwm);
          analogWrite(11, pwm);
          delay_time_4();
          digitalWrite(6, LOW);
          delay_time_4();
          digitalWrite(11, LOW);
          delay_time_4();
          digitalWrite(5, LOW);
          delay_time_4();
          digitalWrite(10, LOW);
          break;
  case 4: pwm=pwm_value();
          analogWrite(5, pwm);
          analogWrite(6, pwm);
          analogWrite(10, pwm);
          analogWrite(11, pwm);
          delay_time_4();
          digitalWrite(11, LOW);
          delay_time_4();
          digitalWrite(10, LOW);
          delay_time_4();
          digitalWrite(6, LOW);
          delay_time_4();
          digitalWrite(5, LOW);
          break;
  }
}

void loop() {
  randNumber = random(1,4);
  Serial.println(randNumber);
  switch(randNumber){
  case 1:RandLED_2();
        break;
  case 2:RandLED_3();
        break;
  case 3:RandLED_4();
        break;

  }


}

```

###5 总结
为了更好的仿真萤火虫的发光习性，可以将所有可能发光的情况都列举出来，并通过随机数来筛选本次执行的情况。另外，通过选择Microduino替代Arduino NANO控制器，可以将控制器及电池盒放置于瓶盖下面，这会更加美观。还可以在瓶子内放置一些绿色的丝带，以模仿野外草丛的情况。最后，还可以选用可充电锂电池，并使用太阳能电池来为锂电池充电，这样一来，白天晒晒太阳，晚上就可以享受萤火虫的夜晚，环保健康。

