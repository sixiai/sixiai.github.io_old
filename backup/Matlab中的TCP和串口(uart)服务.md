一个很简单的TCP例子如下：
```
clc; clear; close all;

t = tcpclient("192.168.1.120", 8080, "Timeout", 5);
PACK = 128;         % 每帧长度
IS_BIG_ENDIAN = false;
sbl_dada = [0,0,0,0,0];
disp("等待二进制帧...");
buf = uint8([]);
i = 0;
a = 1;
h = 1.732;

figure(1)

while true
    n = t.NumBytesAvailable;
    if n > 0
        buf = read(t, n, "uint8");
    end
    % 够一帧就解析
    while numel(buf) == PACK
        distance1_vals = typecast(buf(45:48), 'single');
        distance2_vals = typecast(buf(49:52), 'single');
        distance3_vals = typecast(buf(53:56), 'single');
        distance = [distance1_vals distance2_vals distance3_vals];
        body_position = distance_calculation_xyz(distance,a,h);
        increase_vals = typecast(buf(117:120), 'single');
        energy_vals = typecast(buf(121:124), 'single');
        sbl_dada = [sbl_dada;distance1_vals distance2_vals distance3_vals increase_vals energy_vals];
        fprintf("distance: %f  %f  %f  %f  %f\n", distance1_vals, distance2_vals, distance3_vals, increase_vals, energy_vals);
        buf = uint8([]);
        i = i + 1;
        subplot(3,2,[1 2]);
        plot(i,distance1_vals,'b.',i,distance2_vals,'g.',i,distance3_vals,'r.');
        hold on
        ylim([11 20])
        grid on
        
        subplot(3,2,3);
        plot(i,increase_vals,'b.');
        ylim([-500 300])
        grid on
        hold on
        subplot(3,2,4);
        plot(i,energy_vals,'b.');
        ylim([0 20000000])
        grid on
        hold on

        subplot(3,2,[5 6]);
        plot(body_position(:,1),body_position(:,2),'r*-')
        grid on
        hold on
    end

    pause(0.005);
end

function body_position = distance_calculation_xyz(distance,a,h)
% 三水听器等边三角形布阵
% 1号水听器位于X轴的正半轴，坐标为(a,0,0)，
% 2号水听器位于Y轴的正半轴，坐标为(0,h,0)，
% 3号水听器位于X轴的负半轴，坐标为(-a,0,0)。
% S表示信标其坐标为body_position = [x,y,z]，其分别距离1、2、3号水听器的斜距为distance = [d1,d2,d3]。则有：
d1_2 = distance(1)*distance(1);
d2_2 = distance(2)*distance(2);
d3_2 = distance(3)*distance(3);

body_position(1) = (d3_2-d1_2)/(4*a);                           %x
body_position(2) = (d1_2+d3_2-2*d2_2)/(4*h) - a*a/(2*h) + h/2;  %y
z2 = d1_2-(body_position(1)-a)*(body_position(1)-a)-body_position(2)*body_position(2);  %z2
if(z2<0)
    body_position(3) = sqrt(-z2);
else
    body_position(3) = sqrt(z2);
end
end
```

串口的例子如下：
```
clc; clear; close all;
% 创建到串行端口设备的连接
t = serialport("COM3", 115200);
PACK = 28;         % 每帧长度
disp("等待二进制帧...");
sbl_dada = [0,0,0,0,0];
buf = uint8([]);
i = 0;
figure(1)

while true
    n = t.NumBytesAvailable;
    if n > 0
        buf = read(t, n, "uint8");
    end
    % 够一帧就解析
    while numel(buf) == PACK
        distance1_vals = typecast(uint8(buf(3:6)), 'single');
        distance2_vals = typecast(uint8(buf(7:10)), 'single');
        distance3_vals = typecast(uint8(buf(11:14)), 'single');
        increase_vals = typecast(uint8(buf(19:22)), 'single');
        energy_vals = typecast(uint8(buf(23:26)), 'single');
        sbl_dada = [sbl_dada;distance1_vals distance2_vals distance3_vals increase_vals energy_vals];
        fprintf("distance: %f  %f  %f  %f  %f\n", distance1_vals, distance2_vals, distance3_vals, increase_vals, energy_vals);
        buf = uint8([]);
        i = i + 1;
        subplot(2,2,[1 2]);
        plot(i,distance1_vals,'b.',i,distance2_vals,'g.',i,distance3_vals,'r.');
        ylim([2 20])
        grid on
        hold on
        subplot(2,2,3);
        plot(i,increase_vals,'b.');
        ylim([-500 300])
        grid on
        hold on
        subplot(2,2,4);
        plot(i,energy_vals,'b.');
        ylim([0 20000000])
        grid on
        hold on
    end

    pause(0.005);
end

sbl_data_251104_2 = sbl_dada;
save("sbl_data_251104_2.mat",'sbl_data_251104_2')
```
