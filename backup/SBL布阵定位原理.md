# 四水听器情况

<img width="343" height="390" alt="Image" src="https://github.com/user-attachments/assets/866ff70c-a416-463e-9515-63093d10345b" />

如图1所示为SBL的坐标系统，1号水听器位于水听器基阵的原点坐标为(0,0,0)、2号水听器位于x坐标轴上坐标为(A,0,0)、3号水听器位于y坐标轴上坐标为(0,B,0)、4号水听器位于z坐标轴上坐标为(0,0,C)。S表示信标其坐标为(x,y,z)，其分别距离1、2、3、4号水听器的斜距为d1、d2、d3、d4。则有：
```math
\left\{\begin{matrix}d_1^2=x_1^2+y_1^2+z_1^2      (1)
 \\d_2^2=(x-A)_1^2+y_1^2+z_1^2      (2)
 \\d_3^2=x_1^2+(y-B)_1^2+z_1^2      (3)
 \\d_4^2=x_1^2+y_1^2+(z-C)_1^2      (4)
\end{matrix}\right.
```
由(1)-(2)、(1)-(3)、(1)-(4)得：
```math
\left\{\begin{matrix}x=(d_1^2-d_2^2+A^2)/2A  (5)
 \\y=(d_1^2-d_3^2+B^2)/2B  (6)
 \\z=(d_1^2-d_4^2+B^2)/2C  (7)
\end{matrix}\right.
```
可知由(5)、(6)、(7)式即可得到信标在水听器基阵下的相对坐标x,y,z。

# 三水听器等腰三角形布阵的情况
如图3所示，1号水听器位于X轴的正半轴，坐标为(a,0,0)，2号水听器位于Y轴的正半轴，坐标为(0,h,0)，3号水听器位于X轴的负半轴，坐标为(-a,0,0)。S表示信标其坐标为(x,y,z)，其分别距离1、2、3号水听器的斜距为d1、d2、d3。则有：
```math
\left\{\begin{matrix}d_{1}^{2} =(x-a)^{2}+y^{2}+z^{2}      （1）
 \\d_{2}^{2} =x^{2}+(y-h)^{2}+z^{2}      （2）
 \\d_{3}^{2} =(x+a)^{2}+y^{2}+z^{2}     （3）
\end{matrix}\right.
```
由(1)-(3)得：
```math
x=\frac{d_{3}^{2}-d_{1}^{2}}{4a} 
```
由(1)-(2)并结合(4)得：
```math
y=\frac{d_{2}^{2}+d_{3}^{2}-2d_{2}^{2}}{4h}-\frac{a^{2}}{2h}+\frac{h}{2}
```
由(1)得：
```math
z=\sqrt{d_{1}^{2}-(x-a)^{2}-y^{2}} 
```
# 计算信标在大地坐标系下的坐标值
为了得到信标在大地坐标系下的坐标值，因此还需要一个姿态仪，我们假设姿态仪自身的坐标系和水听器的坐标系的XYZ坐标轴相互平行，他们之间只差了一个平移量。然后姿态仪输出的横滚角Roll、俯仰角Pitch和艏向角Yaw分别为$`\alpha`$、$`\beta`$、$`\gamma`$，那么有水听器基阵坐标系到大地坐标系的坐标旋转矩阵：
```math
R=\begin{bmatrix}cos(\gamma ) 
  & sin(\gamma) & 0\\-sin(\gamma)
  & cos(\gamma) & 0\\0
  & 0 &1
\end{bmatrix}\cdot \begin{bmatrix}cos(\beta )
  & 0 & -sin(\beta)\\0
  & 1 & 0\\sin(\beta)
  & 0 &cos(\beta)
\end{bmatrix}\cdot \begin{bmatrix}1
  & 0 &0 \\0
  & cos(\alpha ) & sin(\alpha)\\0
  & -sin(\alpha) &cos(\alpha)
\end{bmatrix}=R_3R_2R_1
```
那么信标在大地坐标系中的坐标为：
```math
\begin{bmatrix}x'
 \\y'
 \\z'
\end{bmatrix}=
R\begin{bmatrix}x
 \\y
 \\z
\end{bmatrix}
```
从而我们可以得到信标在大地坐标系下的坐标值x'、y'、z'。