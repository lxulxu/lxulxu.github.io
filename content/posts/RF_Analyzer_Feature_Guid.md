---
title: 'RF Analyzer 功能详解'
date: 2023-02-27 19:06:33
tags: [xilinx, RF]
categories: [xilinx]
---
# RF-ADC
![img-72](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-72.uyj3mknshqo.jpg?raw=true)

## 转换器设置(Converter Settings)
- 校准模式(Calibration Mode)
  ![img-151](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-151.4s8e5r0arfa0.jpg?raw=true)
  校准子系统由三个主要模块组成：

  - 时间交错偏移校准模块(OCB):校正每个子RF-ADC的**DC偏移**
  - 增益校准模块(GCB):校正交错子RF-ADC之间的**增益差异**
  - 时间偏移校准模块(TSCB):校正交错子RF-ADC之间的**时间偏移**

  除了自动校准外，所有四个校准模块（OCB1、OCB2、GCB、TSCB）都可用于获取和设置用户系数。 应用程序读回校准解冻时生成的系数，并在需要时恢复它们； 这有助于在输入信号不满足校准要求时保持 RF-ADC 性能。 此功能适用于 IP 向导中的每个 RF-ADC。 启用此功能会增加 IP 的大小。

  ```c++
  //以下示例代码显示了 TSCB 的用户系数设置。
  
  u32 Status = XRFDC_FAILURE;
  XRFdc_Calibration_Coefficients Coeffs;
  //使用下面的样本系数
  Coeffs.Coeff0 = 146;
  Coeffs.Coeff1 = 255;
  Coeffs.Coeff2 = 255;
  Coeffs.Coeff3 = 255;
  Coeffs.Coeff4 = 113;
  Coeffs.Coeff5 = 255;
  Coeffs.Coeff6 = 255
  Coeffs.Coeff7 = 255;
  Status = XRFdc_SetCalCoeffients( RFdcInstPtr, Tile, Block,
  XRFDC_CAL_BLOCK_TSCB, &Coeffs);
  If (Status != XRFDC_SUCCESS) {
      /*handle error*/
  }  
  ```
  使用`XRFdc_SetCalCoefficients` API 恢复校准系数会自动禁用实时校准。 提供 `XRFdc_DisableCoefficientsOverride` API 以禁用此用户系数覆盖模式并重新启用实时校准。 禁用实时校准时，实时端口校准冻结无效。

  ![img-74](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-74.4auk0q3tcqw0.jpg?raw=true)

  - Mode 1:优化0.4* Fs 到Fs/2范围内信号，对于输入频率Fsamp/2(Nyquist) ± 10%是最佳的。
  - Mode 2:优化0 到0.4* Fs范围内信号，适用于其他范围输入频率。

- 奈奎斯特区(Nyquist Zone):每个RF-ADC通道可以在第一或第二奈奎斯特区采样信号。第一奈奎斯特区被定义为信号在0和Fs/2之间，第二奈奎斯特区被定义为信号在Fs/2和Fs之间。为了确保RF-ADC的性能最佳，RF-ADC配置应指示预期的操作区域。只要信号符合RF-ADC输入带宽要求，也可以使用其他奈奎斯特区。第1，3，5 ...区称为奇数区，第2，4 ...区称为偶数区。1 用于奇数区域，2 用于偶数区域。
- 校准冻结(Freeze):用于冻结每个通道的交错校准。当模拟输入电压过高或相关的`int_cal_freeze`输入被断时，校准将被冻结。绿灯表示冻结状态。冻结按键可冻结或解冻交错校准。
- 禁用引脚(Disable Pin):禁用校准冻结实时端口控制
- 信号衰减(Attenuation):每个ADC通道片内数字步进衰减器(Digital Step Attenuator, DSA)衰减值(以dB为单位)，disable pin功能可以禁用DSA引脚控制。
- 抖动(Dither):除非采样率低于最大采样率0.75倍，否则启用。
- 输入共模电压范围(VCM)

## 阈值检测(Threshold Detection)
每个 RF-ADC 通道有两个实时输出信号，具有过量程(Over Range)和过压(Over Voltage)功能，当模拟输入超过限度的时候会给出中断，同时上报给通信总线，以便直接访问 PL 设计。下图显示了阈值、过量程和过压电平以及随着输入模拟信号的增加这些电平的响应。
![img-152](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-152.47kp0c0pnvm0.jpg?raw=true)

- Mode

  | 模式 | 功能 |
  | --- | --- |
  | Off | 阈值电路禁用，状态输出为低。|
  | Sticky over | 当来自RF-ADC的数据超过上限阈值时，阈值状态信号为高电平，状态一直保持到发送清除操作为止。|
  | Sticky under | 当来自RF-ADC的数据在用户指定的时间或延迟内保持低于下限阈值时，阈值状态信号为高电平，状态一直保持到发送清除操作为止。使用RFdc驱动程序API设置计数器，使用此机制可防止触发阈值事件的短时间偏移。|
  | Hysteresis | 当超过设定的上限阈值时设置状态输出，当信号在用户指定的延迟值期间保持低于下限阈值时清除。下限阈值的延迟值由32位计数器定义，使用RFdc驱动程序API设置计数器，延迟为阈值检测增加了滞后，以防止触发阈值事件的短时间偏移。|

- Under Level:低阈值
- Over Level:高阈值
- Delay:延迟值

使用 RFdc 驱动程序 API 配置阈值电平和延迟值示例如下。
```c++
// Initial Setup
XRFdc_Threshold_Settings Threshold_Settings;
Threshold_Settings.UpdateThreshold = XRFDC_UPDATE_THRESHOLD_BOTH; // Setup
values
for threshold 0 and 1
Threshold_Settings.ThresholdMode[0] = XRFDC_TRSHD_STICKY_UNDER; // Set
threshold0
mode to Sticky Under
Threshold_Settings.ThresholdUnderVal[0] = 1000; // Measured in 14-bit
unsigned LSBs
Threshold_Settings.ThresholdAvgVal[0] = 10; // Data must be below lower
threshold for 10*8 4 GSPS RF-ADC samples
// Write threshold values to the selected Tile / RF-ADC
XRFdc_SetThresholdSettings(ptr, Tile, Block, &Threshold_Settings);
```

## 抽取设置(Decimation Settings)
降采样滤波器（Decimation Filter）将高采样率信号降采样到较低采样率。通过将输入信号的带宽限制到降采样后的采样率的一半，然后将其下采样来减少数据量。降采样滤波器通常由两部分组成：抗混叠滤波器和下采样器。抗混叠滤波器通过降低输入信号的带宽，以避免在下采样时出现混叠误差，而下采样器则将信号降采样到所需的采样率。

| 选项 | 说明 |
| --- | --- |
| OFF | 滤波器被禁用，RF-ADC 不可用 |
| 1x | 滤波器被绕过 |
| 2x |　2倍抽取，80% Nyquist 通带 |
| 4x | 4倍抽取，80% Nyquist 通带 |
| 8x | 8倍抽取，80% Nyquist 通带 |

RFdc 驱动程序 API 可用于使用以下代码获取 IP 内核中设置的抽取率。
```c++
//获取 Tile0、DDC Block1 的抽取因子
int Tile = 0;
u32 Block = 1;
u32 Decimation_Factor;
if( XRFdc_GetDecimationFactor (ptr, Tile, Block, &DecimationFactor) == XST_SUCCESS) {
	xil_printf("ADC Tile%1d,%1d Decimation Factor is: %d", Tile, Block,Decimation_Factor);
}
```

# RF-DAC

![img-81](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-81.7kckidky6ms0.jpg?raw=true)

## 转换器设置(Converter Settings)
- 数据通路模式(DataPath):四种可用模式

<table>
    <tr>
        <td>Mode</td> 
        <td colspan="2">Full Nyquist DUC</td> 
        <td colspan="2">IMR Low-pass</td> 
        <td colspan="2">IMR High-pass</td>
        <td colspan="2">DUC-Bypass</td>
   </tr>
    <tr>
        <td>IMR x2</td>
        <td colspan="2">OFF</td> 
        <td colspan="2">ON</td> 
        <td colspan="2">ON</td>
        <td colspan="2">OFF</td>   
    </tr>
    <tr>
        <td>Mix-Mode</td>
        <td>OFF</td> 
        <td>ON</td>
        <td>OFF</td> 
        <td>ON</td>
        <td>OFF</td> 
        <td>ON</td>
        <td>OFF</td> 
        <td>ON</td>
    </tr>
    <tr>
        <td>Usable Bandwidth(Fs)</td>
        <td>0-0.45</td> 
        <td>0.55-0.95</td> 
        <td>0-0.2</td>
        <td>0.8-0.95</td> 
        <td>0.3-0.45</td> 
        <td>0.55-0.7</td>
        <td>0-0.45</td> 
        <td>0.55-0.95</td>    
    </tr>
    <tr>
        <td>Reconstruction Filter</td>
        <td>Low-pass</td> 
        <td>Band-pass</td>
        <td>Low-pass</td> 
        <td>Band-pass</td>
        <td>Low-pass</td> 
        <td>Band-pass</td>
        <td>Low-pass</td> 
        <td>Band-pass</td>
    </tr>
</table>

- 解码模式(Decoder Mode):选择要优化的性能：噪声基底或线性。必须为通信应用选择噪声基底优化。宽带调制信号推荐使用低噪声模式。
- 奈奎斯特区(Nyquist Zone):选择信号将位于哪个奈奎斯特区域，1正常模式，2混合模式。
  ![img-58](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-58.6mswppffpl40.jpg?raw=true)
  - 正常模式:蓝线表示理想的RF-DAC输出陡峭的sinc响应。在这种模式下，第二奈奎斯特区中的输出图像将被严重衰减。对于此模式，可用反sinc滤波器来补偿第一奈奎斯特区的衰减。
  - 混合模式:红线表示理想的RF-DAC输出响应。在这种模式下，第二奈奎斯特区中的输出功率显著增加，并且在大多数区域中也具有近似平坦的响应。可以在Vivado IDE中设置奈奎斯特区。由于sinc响应衰减和可能的带宽衰减，通常不会将RF-DAC在高于2的奈奎斯特区操作。
- 当前值(Current):当前可变输出功率(variable output power, VOP)值

## 反Sinc补偿(Inverse Sinc Settings)
模拟输出响应特性更接近sin(x)/x (冲击响应)，因此如果后级想取得比较平坦的系统响应需要较大的带宽。而做一个比较宽的平坦的模拟滤波器既不经济也不实惠，因此需要在DAC的输出级之前添加一级反sinc滤波器。可以认为和高速串行总线的预加重类似，通过提高信号高频分量的能量来抵消来自后端滤波器在过渡带增益不足的影响。
![img-59](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-59.4087xmd6kx40.jpg?raw=true)
蓝色线代表滤波器本身的频率响应。 可以看出，它随频率增加以补偿输出的 sinc 响应，如红线所示。 复合输出响应由黄色迹线给出，并显示高达 89% 奈奎斯特的平坦通带。
Enable启用反Sinc补偿高频下的Sinc衰减，第一代此功能仅在信号位于奈奎斯特区1时有效，第三代此功能在信号位于奈奎斯特区1/2时有效。

## 插值设置(Interpolation Settings)
插值滤波器（Interpolation Filter）将信号从低采样率上采样到高采样率，通过在输入信号中插入零值来扩展输入信号的频谱，然后对其进行低通滤波来移除扩展后信号中的高频分量。

| 选项 | 说明 |
| --- | --- |
| OFF | 滤波器被禁用，RF-DAC 不可用 |
| 1x | 滤波器被绕过 |
| 2x | 2倍插值，80% Nyquist 通带 |
| 4x | 4倍插值，80% Nyquist 通带 |
| 8x | 8倍插值，80% Nyquist 通带 |

RFdc 驱动程序 API 命令如下。
```c++
// Get Interpolation factor for Tile0, DUC Block1
int Tile = 0;
u32 Block = 1;
u32 Interpolation_Factor;
if( XRFdc_GetInterpolationFactor (ptr, Tile, Block, &Interpolation_Factor) == XST_SUCCESS) {
	xil_printf("DAC Tile%1d,%1d Interpolation Factor is: %d", Tile, Block,Interpolation_Factor);
}
```

# 电源管理(Power Management)
- 电源模式(PowerMode):打开/关闭单个通道电源
- DisableIPControl:启用/禁用RTS控制

# 混频设置(Mixer Settings)
![img-155](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-155.5wcnvorhro00.jpg?raw=true)
建议先设置Crossbar页面，然后设置混频器和NCO的其他参数，因为混频器在real-real模式下被旁路。

- 模拟/数字输出(Analog/Digital Output):仅在转换器启动时可配置，可选项Real和I/Q。当转换器0设为I/Q时，转换器1也必须启用；当转换器2设为I/Q时，转换器3也必须启用。
- 类型(Type):可选项bypassed/coarse/fine，取决于转换器数字输出数据字段的选择。
- 模式(Mode):仅转换器启用时可设置，选项取决于混频器类型和数字输出数据格式，real数据对应bypassed，I/Q数据对应Real to I/Q或I/Q to I/Q。
- 频率(Frequency):混频器类型为coarse时可选项Fs/2，Fs/4和-Fs/4，混频器类型为fine时可选范围-10GHz到10GHz。
- 相位(Phase Init):仅在混频器类型为fine时可用，范围在-180到180。
- 刻度(Mixer Scale):无

混频器RFdc API示例如下。
```c++
XRFdc_Mixer_Settings Mixer_Settings;
for(tile=0;tile<4; tile++) {
	// 确保混合器设置更新使用 Tile 事件
    for(block=0; block<2; block++) {
        XRFdc_GetMixerSettings (ptr, XRFDC_ADC_TILE, tile, block,&Mixer_Settings);
        Mixer_Settings.EventSource = XRFDC_EVNT_SRC_TILE; //使用XRFDC_EVNT_SRC_TILE事件更新混频器设置
        XRFdc_SetMixerSettings (ptr, XRFDC_ADC_TILE, tile, block,&Mixer_Settings);
    }
    //重置 Tile0 中两个 DDC 的 NCO 相位（假设两者都处于活动状态）
    XRFdc_ResetNCOPhase(ptr, XRFDC_ADC_TILE, tile, 0); // DDC Block0
    XRFdc_ResetNCOPhase(ptr, XRFDC_ADC_TILE, tile, 1); // DDC Block1
    XRFds_UpdateEvent(ptr, XRFDC_ADC_TILE, tile, 1, XRFDC_EVENT_MIXER); //生成tile事件
}  
```
# QMC 配置
正交调制校正(quadrature modulator correction, QMC)用于调整输出的幅度和相位，当转换器与外部调制器或解调器接口时，这些用于补偿不匹配的I和Q信号路径。
![img-153](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-153.56t3vz1k4h00.jpg?raw=true)

- 增益调整(Enable Gain/Gain):将信号乘以增益因子来完成的，该因子的范围为0到2.0，可以将各个因子应用于I和Q数据路径，block输出分辨率为16位。
- 相位调整(Enable Phase/Phase Mismatch):通过将Q的比例分数添加到I值来实现，范围在+-26度或+-0.5数量级。这种加法的结果会导致增益误差，必须由增益误差校正块来校正，仅在复杂模式下生效。
- DC偏置(Offset):直流偏移调整，仅对直流耦合有效，通过向采样信号添加一个固定的LSB值来完成的，范围是-2048到2047。

可以使用RFdc驱动程序API设置增益、相位和偏移校正因子值。

```c++
// Initial Setup
XRFdc_QMC_Settings QMC_Settings_I, QMC_Settings_Q; // RF-ADC block0 is I, RF-ADC block1 is Q
QMC_Settings_I.EventSource = XRFDC_EVNT_SRC_TILE; // QMC Settings are updated with a tile event
QMC_Settings_Q.EventSource = XRFDC_EVNT_SRC_TILE; ....
// Update Gain/Phase/Offset for I/Q RF-DACs in tile0
QMC_Settings_I.GainCorrectionFactor = 0.9; // Set Gain for I
QMC_Settings_I.PhaseCorrectionFactor = -5.0; // I/Q imbalance factor applied to I side, approx in degrees.
QMC_Settings_I.EnableGain = 1;
QMC_Settings_I.EnablePhase = 1;
QMC_Settings_Q.GainCorrectionFactor = 0.95;
QMC_Settings_Q.EnableGain = 1;
XRFdc_SetQMCSettings(ptr, XRFDC_ADC_TILE, 0, 0, &QMC_Settings_I); // Write settings for ADC0,0 - I ADC
XRFdc_SetQMCSettings(ptr, XRFDC_ADC_TILE, 0, 1, &QMC_Settings_Q); //Write settings for ADC0,1 - Q ADC
XRFdc_UpdateEvent(ptr, XRFDC_ADC_TILE, 0, 0, XRFDC_EVENT_QMC); //Generate a Tile Update Event - applies all QMC Settings at once
```

还可以使用`XRFdc_GetQMCSettings RFdc`驱动程序API命令从任何转换器读回QMC设置。

# 锁相环(Phase-Locked Loop, PLL)
![img-27](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-27.58oy6tchlwo0.jpg?raw=true)

- 时钟源(Clocking Source)
	- 外部(External):选择时钟源来自外部，通常为几GHz量级。
  - 板上(Onboard):选择时钟源来自板上，输入时钟一般小于1GHz。
	- 参考时钟(Clocking Reference)频率:可以来自外部输入或源tile转发时钟。
	- 输入时钟 (Tile Clock Input):数值和之前时钟配置的值一致，和采样率有关，因为采样时钟是使用这个时钟经过PLL生成的。
- 内部锁相环(Internal PLL)
	- Enable:是否启用内部PLL，当片中的至少一个转换器被启用时，它是可配置的。启用PLL时需确保输入频率在Zynq UltraScale+ RFSoC Data Sheet范围内，绕过PLL时转换器的采样时钟作为输入时钟。
  - Bypassed

IP向导或专用于配置PLL系统的API函数将参考分频器值设置为整数，将反馈分频器值设置为整数，将输出分频器值设置为整数，以达到最佳性能。PLL频率输出`Fs = (Fin/R)*(FBDiv/M)`，其中Fin是参考频率，其他参数(RF-ADC Gen 1/Gen 2/Gen 3/DFE and RF-DAC Gen 1/Gen 2)如下表。
|参数|范围|
| --- | --- |
|参考分频器(Reference Divider)(R)|1 to 4|
|反馈分频器(Feedback Divider)(N)|13 to 160|
|输出分频器(Output Divider)(M)|2, 3, 4 ,6, 8,... (even numbers ≤ 64)|
|压控振荡器(VCO)Frequency(GHz)|8.5 to 13.2|

# Crossbar和多频段(Multi-Bands)
![img-30](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-30.1ev1wv9ss1cw.jpg?raw=true)
Crossbar页面影响混频器的real或complex模式以及multi-bands模式。complex模式激活一对信道以支持同相(I)和正交(Q)信号。由于复数混频器(和NCO)架构，允许实到复(real-to-complex,R2C)或复到复(complex-to-complex,C2C)模式，但不允许复到实(complex-to-real,C2R)模式。这意味着没有可用于RF-ADC的C2R模式和可用于RF-DAC的R2C模式。在complex模式中，偶数信道始终用于I信号，奇数信道用于Q信号。
Multi-bands允许一个DAC/ADC模拟信道能够共享多个DUC或DDC信道来发送或接收多波段载波信号。对于DAC，多个基带信号可以在单独的DUC链中进行上转换，然后在crossbar上合并再发送到模拟DAC模块。Multi-bands页面影响混频器的real或 complex模式以及multi-bands模式。RF-ADC multi-bands功能支持以下配置。

|模式|说明|
|---|---|
|Real Dual-Band|每对2x multi-band实数据，一个RF-ADC模拟输入有效，两个RF-ADC数字DDC通道均已启用。|
|I/Q Dual-Band|每对2x multi-band I/Q数据，两个RF-ADC输入有效，一个用于I，一个用于Q，两个RF-ADC数字DDC通道均已启用。|
|Real Quad-Band (Quad ADC tile only)|每个tile有4x multi-band实数据，一个RF-ADC模拟输入有效，四个RF-ADC数字DDC通道均已启用。|
|I/Q Quad-Band (Quad ADC tile only)|每个tile有4x multi-band I/Q数据，两个RF-ADC输入有效，一个用于I，一个用于Q，四个RF-ADC数字DDC通道均已启用。|

当多频段关闭时，I和Q输入直接通过多频段路由逻辑;当多频段开启时，I和Q输入被路由到tile中的多个DDC 模块。
![img-156](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-156.1rgyzywlxn28.jpg)
RF-ADC多频段是通过将一个RF-ADC模拟模块的输出路由到多个RF-ADC DDC模块来实现的。每个块处理一个数据带，并且可以从多个载波混合到基带。下图中的Quad ADC块显示了这一点。
![img-157](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-157.62vhzzi92m00.jpg?raw=true)
RF-ADC Tile 0 (Tile_224) 配置为实输入到I/Q输出模式。ADC0转换双频信号；ADC1关闭。顶部对可以配置为独立的 RF-ADC。双频输出路由到ADC0和ADC1的DDC模块。DDC模块中的混频器可以配置为从输入数据中提取正确的频带。
RF-ADC Tile 1 (Tile_225) 配置为4x多频段I/Q输入到I/Q输出模式。这里ADC0承载四波段I信号，ADC1承载Q数据。ADC2和ADC3关闭。RF-ADC的输出被路由到所有四个DDC模块。每个DDC都可以配置为从所需的频带中提取数据。

# FIFO深度和时钟(Clock Distribution)
![img-41](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-41.1u5d9itfn97k.jpg?raw=true)

- Fin:输入频率
- MMCM:输入时钟分频器Reference Clock经过分频后变成PLL Reference Clock，在MMCM模块中，MMCM为PL侧的FIFO生成读取或写入时钟，在FIFO数据页面中显示为F(PL)。
- F(PL):MMCM为PL端的FIFO生成的读写时钟，计算公式F(PL)=Fin*M/D/ClkDiv。
- 时钟分频(FabClkDiv):在非MTS位流中，转换器采样时钟(Fs，也称为T1)除以8或4，然后除以FabCLKDiv。作为输入参考传输至MMCM模块。
- 捕获样本数(Number of Samples)
- F(T1):Tile时钟

# FFT频谱分析
单击ADC设置页面中的Acquisition按钮或DAC设置页面中的Generation按钮，将打开FFT页面。在DAC FFT页面中，UI中自带单音和双音发生配置选项。要生成复杂的调制信号，用户可以从文件加载测试向量。Fs表示观察到的RF-ADC或RF-DAC的采样频率，Eff.Fs表示抽取后或插值前原始数据流（基带）的采样频率。FFT图的X轴（频率）反映了Eff.Fs。
![img-68](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-68.6zlmduopqqc0.jpg?raw=true)

传统ADC规格
|参数|描述|
| --- | --- |
|无杂散动态范围(SFDR)|衡量数据转换器在杂散分量干扰基本信号或导致基本信号失真之前可用的动态范围,定义是基本正弦波信号均方根(RMS)值与从0Hz(DC)到二分之一数据转换器采样速率(如fs/2)范围内测得的输出峰值杂散信号均方根值之比，单位为dBc。|
|信噪比(SNR)|量化数据转换器内噪声的参数，输入信号功率与噪声功率的比值，单位为dB。|
|信噪失真比(SNDR)|单位为dBc，代表输入信号的质量，SNDR 越大，输入功率中的噪声和杂散比率越小。|
|有效位数(ENOB)|衡量数据转换器相对于输入信号在奈奎斯特带宽上的转换质量（以位为单位）。|

RF采样数据转换器规格
|参数|描述|
| --- | --- |
|噪声频谱密度(NSD)|RF-ADC满刻度输入下每赫兹频率对应的噪声功率，一般使用 dBFS/Hz 为单位，用于衡量有独特采样率的数据转换器的噪声性能。|
|三阶交调(IM3)|任何复杂信号都同时包含多个频率分量。转换器传输函数的非线性不仅会造成单一频率失真，也会导致两个或更多信号频率相互作用，产生交调产物。|
|邻信道泄漏比(ACLR)|在通过空中接口传输信号时，从被传输信号泄漏到邻近信道的功率会干扰邻近信道中的传输，劣化无线电系统的总体性能。|

其他
|参数|描述|
| --- | --- |
|FundA|fund 信号的 RMS 功率电平，以 dBFS 表示。|
|SFDRxH23|SFDR 不包括以 dBc 为单位的二次和三次谐波失真。谐波失真的位置是可预测的，因此可以在应用中单独处理。因此，列出了单独的 SFDRxH23 供参考。|
|Fspur|第一个奈奎斯特频带中最差 spur 的频率位置，单位为 MHz。|
|FspurxH23|最差杂散的频率位置不包括第一奈奎斯特频带中以 MHz 为单位的二次谐波和三次谐波失真。本参数包括 ADC OIS、GTIS 和 PLL 参考杂散。|
|THD|总谐波失真，单位为 dBc。THD 是 RMS 信号能量与前六次谐波之和的 RMS 值之比。|
|频率杂散|PLL 的输入参考（相位频率检测器的频率）产生的杂散，包括其谐波。当使用外部 PLL 直接计时，必须在 PLL 选项卡中指明此评估工具 GUI 的参考频率，以计算频率杂散。RF-ADC 采用交织技术构建。通过选择交织性能，偏移交织和增益/定时交织的支路列在 RF-ADC FFT 选项卡上。|
|交织偏移|偏移交织杂散的频率位置和振幅。|
|交织增益|增益/定时交织杂散的频率位置和幅度。|

# 中断功能(Interrupts)
在系统异常时，自动回读一些AD/DA寄存器状态。每行开头的复选框启用或禁用(掩码)相应的中断状态。单击Apply按钮应用所选中断状态。Refresh按钮读回当前状态，绿灯显示相应的中断位被设置。Clear按钮尝试清除所有中断位并读回状态。
![img-80](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-80.xcl67cmg8m8.jpg?raw=true)

| 中断 | 描述 |
| --- | --- |
|RF-ADC/RF-DAC Datapath Interrupt	| IP寄存器在0x208 0x210 0x218 0x220中断 |	
|Overflow in RF-DAC<br>Interpolation Stage 0/1/2/3 I or Q datapath	| 插值阶段溢出/饱和，Flag表示发生溢出阶段和位置。|
|Overflow in RF-ADC<br>Decimation Stage 0/1/2/3 I or Q Datapath | RF-ADC抽取阶段溢出/饱和，Flag表示发生溢出阶段和位置。|	
| Overflow in RF-DAC/RF-ADC QMC Gain/Phase | QMC增益/相位校正溢出/饱和，数据幅度或校正系数过大。|
| Overflow in RF-DAC/RF-ADC QMC Offset | QMC偏移校正溢出/饱和，数据幅度或校正系数过大。|
| Overflow in RF-DAC Inverse Sinc Filter(2nd Nyq)	| RF-DAC反Sinc补偿滤波器溢出/饱和，数据过大。|	
| Over/Underflow in RF-DAC Mixer | 混频器溢出 |
| IMR Overflow | 镜像抑制模块溢出 |
| Overflow in Inverse Sync Filter	| 混频模式中DAC反补偿模块溢出	|
| Sub RF-ADC 0/1/2/3 Over/Under Range Interrupt	| 模拟输入超出Sub-RF-ADC范围，应减小输入信号幅度。	|
| ADC Over Range | 模拟输入超出RF-ADC范围，应减小输入信号幅度。要清除此中断错误，需清除sub-RF-ADC超范围中断错误。	|
| ADC Over Voltage | 模拟输入超出RF-ADC缓存区安全输入范围，缓存区关闭，输入信号幅度和共模电压应在范围内。|
| ADC Common Mode Over Voltage | RF-ADC输入共模电压高于指定值 |
| ADC Common Mode Under Voltage	| RF-ADC输入共模电压低于指定值 |
| RF-ADC/RF-DAC FIFO Over/Underflow	| IP寄存器中断在0x208, 0x210, 0x218, 0x220之一，要清除此中断错误，需清除所有路径的子中断错误。|
| RF-ADC/RF-DAC Overflow<br>RF-ADC/RF-DAC Underflow<br>RF-ADC/RF-DAC Marginal Overflow<br>RF-ADC/RF-DAC Marginal Underflow | FIFO接口设置不正确，时钟/数据吞入率不匹配。|

# 多通道同步(Multi-Tile Synchronization)
FIFO为RF-ADC提供灵活的数据和时钟接口。但与所有双时钟FIFO一样，延迟可能会在一个tile和另一个tile之间变化。虽然 tile中的所有通道都具有相同的延迟，但某些应用可能需要使用多个RF-ADC tile，并且需要在所有RF-ADC通道中匹配延迟。这些应用程序可以使用多通道同步(MTS)功能来实现这种块间同步。
![img-36](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-36.317omq9p5gm0.jpg?raw=true)


1. <p name = ''>https://docs.xilinx.com/r/en-US/pg269-rf-data-converter</p>
2. <p name = ''>https://docs.xilinx.com/r/en-US/ug1309-rf-data-converter-interface</p>
3. <p name = ''>https://docs.xilinx.com/v/u/zh-CN/wp509-rfsampling-data-converters</p>
4. <p name = ''>https://blog.csdn.net/weixin_41445387/category_11999717.html</p>