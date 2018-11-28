[记录] 给 GEN8 造一片风扇驱动板    

运维教程硬件折腾 mhycy · 4月前

**Github 地址:** [hardware-gen8-pwm-convert](https://github.com/mhycy/hardware-gen8-pwm-convert)

### 前言

HP MicroServer Gen8 虽好, 但风扇略吵    
网络上的各路大神给出了一堆方案, 有屏蔽上温控器的, 有加电阻的    
还有看起来最靠谱的直接替换    
就像这个贴 [无需外部电路，将HP Gen8替换为普通PWM风扇，并保持自动调速](https://www.chiphell.com/thread-1087322-1-1.html)

电源供电, PWM 引脚作为地线, 这样就能控制风扇转速了~    
看起来很美好对吧？能把风扇控制在一个极低的转速, 但是...错了...

开始实验的时候就会发现, 这个接线方式会使得风扇维持在一个**不可思议的低转速**状态    
风扇的实际转速和面板的转速百分比完全不在一个等级上面    
同时测量电源对PWM信号引脚的等效电压会发现等效电压的增减与面板一致....

研究过相关文档以及详细的测量对比后会发现这其实是利用了GEN8特殊的PWM信号定义    
以及其漏极开路输出的特性所达到的控制效果

> 把PWM信号当成地线使用, 随着PWM占空比增加, 等效电压降低, 风扇转速降低

**为什么说这是 Gen8 特殊的PWM定义呢？**    
网络上找到的4-pin风扇PWM信号定义文档可以给出答案    
(点击下载)[源地址(4_wire_pwm_spec.pdf)](http://www.formfactors.org/developer/specs/4_wire_pwm_spec.pdf)    
(点击下载)[本站镜像(4_wire_pwm_spec.pdf)](https://blog.mhycy.me/usr/uploads/2018/07/3619238088.pdf)

按照标准的定义, CPU风扇的PWM信号理应随着PWM等效电压的增加而加速, 随着等效电压的降低而减速    
Gen8 的PWM信号刚好相反而行, 随着等效电压的增加而减速, 显然是个**反相**定义的信号

> **这也是为什么论坛上没有直接替换案例的缘由**    
> 仅仅屏蔽RD引脚, 把风扇接入, 会发现开机过程中风扇会在不可思议的低速, 然后不断增加

那有没有完美的方案呢？    
既可以使用系统的PWM信源, 又可以替换成自己的风扇, 还不需要改动 Gen8 的风扇接口 (Gen8的风扇端口非常特殊)

> 造个驱动板就好了~

**这就是本文的来历**

### 需求

* 无损的引出 Gen8 的风扇端口不造成任何破坏
* 把 Gen8 的 PWM 变成正常的信号, 且合规
* 欺骗 RD(Rotation Detect) 引脚
* 自带多电压输出[12V, 5V]以支持芯片组散热用的迷你风扇 (目标改成水冷风压不够被动散热的)

### 坑点与实现

* 需要找到 Gen8 风扇所使用的插针标准, 最终找到这个: [PHD2.0-2*3](https://item.szlcsc.com/72369.html)
* 需要符合漏极开路输出的信号反相器, 寻找一个需要电平兼容以及漏极开路输出的与非门, 74系列走起~ [CD74HCT03M](http://www.ti.com/lit/gpn/cd54hc03)
* 多电压输出, 原本的输入是12V, 那么只需要实现 5V的输出即可...考虑到电流, DC-DC电路实现

**原理图-电源**    
忘记从哪里抄回来的DC-DC电路, 是个很经典的设计    
_感谢 V2EX 大佬 @zhujinliang 提醒 源DC-DC设计是 **kis3r33s**_   
![原理图-电源.png](https://i.loli.net/2018/07/12/5b46b994045fd.png "原理图-电源.png")

**原理图-逻辑与输出**    
![原理图-逻辑与输出.png](https://i.loli.net/2018/07/12/5b46b9940e890.png "原理图-逻辑与输出.png")

**PCB走线**    
圆润可靠的走线~    
![PCB走线.png](https://i.loli.net/2018/07/12/5b46b994111a4.png "PCB走线.png")

**PCB渲染图**    
真的好看！！！    
![PCB渲染图.png](https://i.loli.net/2018/07/12/5b46b99415675.png "PCB渲染图.png")

**实际成品**    
这嘉立创又把字刷歪了！！！！还断线！！！！说好的数字化印刷呢？  
![成品图.JPG](https://i.loli.net/2018/07/12/5b46ba556dfaa.jpg "成品图.JPG")

[local images](img)