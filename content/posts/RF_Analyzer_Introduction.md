---
title: 'RF Analyzer 简介'
date: 2023-02-28 18:54:18
tags: [xilinx, RF]
categories: [xilinx]
---
参考PG269 Ch1 & Ch2。

# 特点

- 多达16个14-bit RF-DAC
  Gen 1/Gen 2:4个14-bit二倍频RF-ADC tile，2/4个14-bit四倍频RF-ADC tile；
  Gen 3:1/2/4个14-bit二倍频RF-ADC tile，2/4个14-bit四倍频RF-ADC tile。
- 支持多个转换器之间的对齐(多片同步(MTS))
- 支持预编程RF-DAC和RF-ADC，用户可以定义关键参数
- RF-ADC和RF-DAC有多个AX14-Stream数据接口
- 单独的AX14-Lite配置接口
- Gen 1/Gen 2:1x(旁路)，2x, 4x, 8x抽取和插值
  Gen 3:1x(旁路)，2x, 3x, 4x, 5x, 6x, 8x, 10x, 12x, 16x, 20x, 24x, 40x抽取和插值后的额外的2x插值
- 数字复合混频器和数控振荡器(NCO)
- 正交调制校正(QMC)，Gen 3每个RF-ADC有嵌入式数字步进衰减器(DSA)，每个RF-DAC有可变输出功率(VOP)控制
- 片上时钟系统包含每个tile的PLL
- Gen 3:片上时钟分配网络；TDD模式支持省电模式和RX/Obs共享模式

# Dual and Quad RF-ADC/RF-DAC Tiles

|Tile类型|转换器数量|设备类型|说明|
|---|---|---|---|
|Quad RF-ADC|4|Gen 1/Gen 2/Gen 3/DFE|交错系数为4|
|Dual RF-ADC|2|Gen 1/Gen 3/DFE|交错系数为8，采样率为Quad RF-ADC两倍。|
|Quad RF-DAC|4|Gen 1/Gen 2/Gen 3/DFE|一个专用DUC|
|Dual RF-DAC|2|Gen 3/DFE|两个专用DUC，Gen 3/DFE配备Quad RF-DAC或Quad/Dual RF-DAC组合，所有的tile都有外部时钟输入|

下图为tile结构，虚线表示多个波段的情况。
![img-158](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-158.6rfb9iadm1c0.jpg?raw=true)
![img-159](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-159.5uupdrk2apo0.jpg?raw=true)
![img-160](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2023Q1/img-160.ma07k43z6g0.jpg?raw=true)

# RF-ADC/DAC

RF-ADC/DAC结构如下图所示。

<center class="half">
    <img src="../assets/images/Y2023Q1/img-161.2kz79mrpyhq0.jpg" width="50%"><img src="../assets/images/Y2023Q1/img-162.3ilmbd5bf200.jpg" width="50%">
</center>

<center class="half">
    <img src="../assets/images/Y2023Q1/img-163.4ejogkegsdg0.jpg" width="50%"><img src="../assets/images/Y2023Q1/img-164.1063rg051tb4.jpg" width="50%">
</center>

RF-ADC/DAC特点如下。

<table>
    <tr>
        <td> </td>
        <td width="45%">ADC</td> 
        <td width="45%">DAC</td>
    </tr>
    <tr>
        <td rowspan="3">Tile配置</td> 
        <td>每个Tile有2/4个RF-ADC和1个PLL</td>
        <td>Gen 1/Gen 2: 每个Tile有4个RF-DAC和1个PLL<br>Gen 3/DFE: 每个Tile有2/4个RF-DAC和1个PLL</td>
    </tr>
    <tr>
        <td>Gen 1/Gen 2: 具有12位RF-ADC分辨率和16位数字信号处理数据路径；在通过DDC块之前，每个12位数据流都与RF-ADC核心输出的16位样本进行MSB对齐。<br>Gen 3/DFE: 具有14位RF-ADC分辨率和16位数字信号处理数据路径；在通过DDC块之前，每个14位数据流都与RF-ADC核心输出的16位样本进行MSB对齐。</td>
        <td>具有14位RF-DAC分辨率和16位数字信号处理路径；数据与16位MSB对齐。</td> 
    </tr>
    <tr>
        <td>实现为四通道（Quad）或两通道（Dual）（采样率取决于设备）</td>
        <td>采样率取决于设备</td> 
    </tr>
    <tr>
    <td rowspan="2">抽取/插值滤波器</td>
        <td>Gen 1/Gen 2: 1x (旁路滤波器), 2x, 4x, 8x<br>Gen 3/DFE: 1x (旁路滤波器), 2x, 3x, 4x, 5x, 6x, 8x, 10x, 12x, 16x, 20x, 24x, 40x</td>
        <td>Gen 1/Gen 2: 1x(旁路滤波器), 2x, 4x, 8x<br>Gen 3/DFE: 1x(旁路滤波器), 2x, 3x, 4x, 5x, 6x, 8x, 10x, 12x, 16x, 20x, 24x, 40x<br>IMR模式下还可用额外的2x</td> 
    </tr>
    <tr>
        <td>80%的奈奎斯特带宽，89dB的阻带衰减</td>
        <td>80%的通带，89dB的阻带衰减</td> 
    </tr>
    <tr>
    <td rowspan="4">数字复数混频器</td>
        <td colspan="2">全复数混频器支持RF-ADC的实数或I/Q输入</td>
    </tr>
    <tr>
        <td colspan="2">每个RF-ADC/DAC带有48位NCO</td>
    </tr>
    <tr>
        <td colspan="2">固定的Fs/4，Fs/2低功耗频率混合模式</td>
    </tr>
    <tr>
        <td>支持I/Q和实数输入信号</td>
        <td>支持RF-DAC混合模式功能，可最大化第二Nyquist区域中的RF-DAC功率</td> 
    </tr>
    <tr>
    <td rowspan="2">单/多频带灵活性</td>
        <td colspan="2">可配置为实数或I/Q输入</td>
    </tr>
    <tr>
        <td>每对RF-ADC可支持2x频带<br>每个Quad RF-ADC Tile可支持4x频带</td>
        <td>每对RF-DAC可支持2x带宽<br>每个RF-DAC Tile可支持4x带宽</td> 
    </tr>
    <tr>
    <td rowspan="7">其他</td>
        <td colspan="2">数字校正外部模拟正交调制器：支持I/Q输入对（两个RF-ADC/DAC）的增益、相位和偏移校正</td>
    </tr>
    <tr>
        <td colspan="2">SYSREF输入信号用于多通道同步</td>
    </tr>
    <tr>
        <td>在旁路模式下可以访问全带宽</td>
        <td>在旁路模式下可以访问全Nyquist带宽</td> 
    </tr>
    <tr>
        <td>每个Tile采用CML时钟输入缓冲器，具有片上校准的100Ω终止电阻<br>提供RF-ADC采样时钟或提供片上PLL的参考时钟</td>
        <td>每个Tile采用CML时钟输入缓冲器，具有片上校准的100Ω终止电阻<br>提供RF-DAC采样时钟或提供片上PLL参考时钟（不适用于奇数双RF-DAC Tile）</td> 
    </tr>
    <tr>
        <td>DC耦合RF-ADC输入的输出公共模式参考电压</td>
        <td>Gen 1/Gen 2: 支持20mA或32mA输出功率模式<br>Gen 3/DFE: 可变输出功率（VOP）支持满量程电流汲取，向后兼容Gen 1和Gen 2的20/32 mA模式</td> 
    </tr>
    <tr>
        <td>Gen 3/DFE: 数字步进衰减器（DSA）；在时分双工（TDD）应用中的省电模式；时分复用（TDD）应用中RX和观测通道的不同抽取因子和FIFO数据速率</td>
        <td>Gen 3/DFE: 在时分复用（TDD）应用中单个功能块的省电模式</td> 
    </tr>
        <tr>
        <td>输入信号幅度阈值：每个RF-ADC有两个可编程阈值标志；<br>灵活的AXI4-Stream接口支持广泛的可编程逻辑时钟频率和转换器采样率；<br>每个RF-ADC专用高速、高性能差分输入缓冲器，具有片上校准的100Ω终止电阻</td>
        <td>Gen 1/Gen 2: 第一Nyquist区的Sinc校正；<br>Gen 3/DFE: 第一和第二Nyquist区的Sinc校正</td> 
    </tr>
</table>
<br>