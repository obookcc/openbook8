# 制作你自己的高科技激光3D扫描仪

>文章来源：http://www.instructables.com/id/3-D-Laser-Scanner/

>作者argon  翻译:Rambutan 校对:Lee苦海

![](http://doask.qiniudn.com/3dlaser1.jpg)

激光3D扫描仪是一个激光扫描装置，你可以使用它制作精密的物体或人模型。它的材料非常简单，只需要一个激光笔（激光头也可），一个红酒杯（或一根实心玻璃棒），一个可旋转的平台，和一个数码相机。

>（译者注：原作者本打算使用酒杯来做激光的透镜，可是在制作中并未真正使用。译者推荐各位读者去买一字线激光笔（头），可以省掉这步。）

>注意！直视激光可能会暂时失明甚至永远丧失视力，请选择小功率的激光器件并做好保护工作。

[模型文件下载](http://doask.qiniudn.com/timsmom4.stl)


###步骤1，安装相机，激光笔和酒杯

>（译者注：如果你使用一字线激光笔（头），酒杯部分可省略）

![](http://doask.qiniudn.com/3dlaser2.jpg)

对激光扫描仪进行校准，以保证光束可从柱透镜穿过，形成一条垂直线而非一个点，并能投影在你的目标物上。

找一个架子，将激光笔（头）和相机装在一个架子上，相距20CM左右。相机应保持水平。再使用一个架子安装透镜（即酒杯），使激光笔（头）打出的光穿过酒杯，得到一条垂直于地面的激光线（译者注：如果你使用了一字线激光笔，该步略过）。


最初，透镜是一个酒杯，但在图片里我使用了一个丙烯酸圆柱体。将摄像机与激光仪摆成15度角。

###第二步：在旋转台上固定目标物

![](http://doask.qiniudn.com/3dlaser3.jpg)


制作一个可以匀速旋转的平台。把被扫描的物体固定在平台的中心。对小型的物体来说，一个旋转平台是非常容易的。但是对于大型的被扫描物体，比如一个人，你可能要花些时间和功夫固定一个办公椅，并想办法使用电机让它匀速旋转。

>注意！如果你打算扫描一个人，被扫描的人在扫描时不要睁开眼睛！

###第三步：灯光，相机，开拍！

![](http://doask.qiniudn.com/3dlaser4.jpg)

因为环境光会影响相机采集数据，所以在扫描时，环境越暗越好。最好是在完全黑暗的房间里，这样相机就会只记录激光笔的投影了。匀速转动你的旋转平台，同时开始录像？。你可以看到激光的投影在物体表面勾画出物体的轮廓。
>（译者注：你可能需要对比不同的扫描结果，适当调整激光笔的角度。）

###第四部：处理视频

![](http://doask.qiniudn.com/3dlaser5.jpg)

将相机拍摄的结果存为一个AVI格式的文件。使用边缘检测算法来找到激光线的位置。重建你的3D模型。重建过程可以使用下面的MATLAB脚本，也可以使用互联网上的其他3D重建工具或程序。


```

%     KnowHowScanner -- 3-D laser scanning software
%     Copyright (C) 2007  Kenneth Jensen
%
%     This program is free software: you can redistribute it and/or modify
%     it under the terms of the GNU General Public License as published by
%     the Free Software Foundation, either version 3 of the License, or
%     (at your option) any later version.
%
%     This program is distributed in the hope that it will be useful,
%     but WITHOUT ANY WARRANTY; without even the implied warranty of
%     MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
%     GNU General Public License for more details.
%
%     You should have received a copy of the GNU General Public License
%     along with this program.  If not, see <http://www.gnu.org/licenses/>.
%
%
%
% KNOWHOWSCANNER  Converts laser scanning video to mathematical surface.
% This program takes as input a ".avi" laser scanning video of a SINGLE
% rotation of an object.  In analyzes the video, frame by frame, to find
% the laser line.  Then, it reconstructs a 3-d surface of the object.
%
% X,Y,Z -- two dimensional matrices defining a surface
% R -- two dimensiona matrix of distance of point on the surface from the
% center
% filename -- name of avi movie file
% laserColor -- color of laser line to filter (default 'red')
% calFrame -- number of the frame to use to "calibrate" the video
% camAngle -- angle the camera makes with the laser
%
function [X,Y,Z,R] = KnowHowScanner(filename, laserColor, calFrame, camAngle)

% if you don't know the camera angle, this just assumes its pi/16
if nargin<4
    camAngle = pi/16;
end;

% if no calibration frame is given, it uses the first frame
if nargin<3
    calFrame = 1;
end;

% if no laser color is given, it assumes red.
if nargin<2
    laserColor = 'red';
end;
switch laserColor
    case 'red'
        laserInd = 1;
    case 'green'
        laserInd = 2;
    case 'blue'
        laserInd = 3;
    otherwise
        laserInd = 1;
end;

% get the calibration image and filter out the laser color
m = aviread(filename, calFrame);
calImg = m.cdata(:,:,laserInd);

% show calibration image
% rotate image
figure(1);
imshow(calImg);
xlabel({'Calibrate rotation of image.','Draw line from top to bottom that should be verticle.', 'Left click to start. Right click to end.'});
[rotX, rotY] = getline(1);
rotAngle = 180/pi*atan2(rotY(2)-rotY(1),rotX(2)-rotX(1))-90;

figure(1);clf;
calImg = imrotate(calImg, rotAngle);
imshow(calImg);

% get center line
xlabel('Click mouse for center line');
set(gcf, 'Pointer', 'fullcrosshair');
[x,y]=ginput(1);
line([x, x], [1,size(calImg,2)]);

% get ROI
xlabel('Draw rectangle over region of interest');
rect = getrect;
% the math changes a little depending on whether
% your laser line is to the left or right of the
% camera
if rect(1)>x
    xoffset = rect(1)-x;
else
    xoffset = (rect(1)+rect(3))-x;
end;
rectangle('Position', rect, 'EdgeColor', [1,0,0]);
set(gcf, 'Pointer', 'arrow');

% create blurring filter to smooth the
% image before finding the laser line
h = ones(5,1)/5;

% set threshold for detecting laser line
threshold = 0.1;

% iterate through frames, find laser line
nfo = aviinfo(filename);
Nframes = 1;%round(nfo.NumFrames/360);
frameTab = 1:Nframes:nfo.NumFrames;
for k=1:length(frameTab)
    disp(sprintf('Analyzing frame #%d', frameTab(k)));
    % read from from video file
    m = aviread(filename,frameTab(k));

    % filter out everything but laser color and rotate
    img = imrotate(m.cdata(:,:,laserInd), rotAngle);

    % crop image to the region of interest
    imgCrop = imcrop(img, rect);

    % filter (i.e. blur) image, and convert from uint8 to double
    imgFilt = filter2(h,im2double(imgCrop));

    % for each row find maximum image intensity and x position of maximum
    [mx(k,:), rtmp(k,:)] = max(imgFilt');

    % if there is a laser line, use interp to fill in any "holes"
    % if there is no laser line, set to 0
    if (length(find(mx(k,:)>threshold))>=2)
        r(k,:)=interp1(find(mx(k,:)>threshold),...
            rtmp(k,find(mx(k,:)>threshold)),1:size(rtmp,2));
    else
        r(k,:)=zeros(1,size(rtmp,2));
    end;

    % draw image and calculated laser line
    figure(2);clf;
    imshow(imgFilt);
    hold on;
    plot(r(k,:),1:size(r,2),'r');
end;

% convert x position of laser line in image, to a real radius
if xoffset>0
    R = (r+xoffset)./sin(camAngle);
else
    R = (rect(3)-r-xoffset)./sin(camAngle);
end;
theta = linspace(0,2*pi,length(frameTab))';

% convert to surface coordinates
X = R.*repmat(cos(theta),1,size(imgFilt,1));
Y = R.*repmat(sin(theta),1,size(imgFilt,1));
Z = -repmat(1:size(imgFilt,1),length(theta),1);

% show image
figure(22);
S = surf(X,Y,Z,R);
set(S, 'LineStyle', 'none');
axis square;

```
