---
title: 'FPGA设计流程与EDA工具架构解析——以Vivado为例'
date: 2025-03-04
tags: [eda, Xilinx]
categories: [笔记]
---

## FPGA设计流程概览

FPGA设计遵循一个由上至下的流程，主要包括以下阶段：

1. **功能设计/RTL**：使用HDL(Verilog/VHDL)编写代码，进行功能仿真验证
2. **逻辑综合**：RTL转换为门级网表，进行逻辑优化
3. **物理设计**：包括布局（确定各单元具体位置）和布线（实现单元间互连）
4. **物理验证**：设计规则检查、时序分析、功耗评估
5. **比特流生成**：生成FPGA配置文件
6. **硬件配置与调试**：将设计烧录到FPGA并进行系统测试

一个典型的Vivado项目管理tcl脚本示例：

```tcl
# 创建并配置项目
create_project myproject ./myproject -part xc7a100tcsg324-1

# 添加设计文件
add_files -fileset sources_1 ./src/design/
add_files -fileset constrs_1 ./src/constraints/

# 设置顶层模块
set_property top top_module [current_fileset]

# 创建综合运行
create_run -flow {Vivado Synthesis 2022} synth_1
set_property strategy "Flow_PerfOptimized_high" [get_runs synth_1]

# 创建实现运行
create_run -flow {Vivado Implementation 2022} -parent_run synth_1 impl_1
set_property strategy "Performance_Explore" [get_runs impl_1]
```


## 核心功能模块

### 逻辑综合模块

将RTL代码转换为适合目标FPGA架构的门级网表，主要包括三个子功能：

- **前端解析**：读取RTL源代码(Verilog/VHDL)并将其转换为综合工具的内部数据结构。

- **逻辑优化**：对前端解析生成的电路表示进行一系列变换，以提高设计的性能、减少资源使用或降低功耗。

- **技术映射**：将优化后的逻辑网表转换为目标FPGA架构的基本构建单元，映射到查找表(LUT)、触发器(FF)、数字信号处理块(DSP)、嵌入式存储器(BRAM)等FPGA特定资源上。

![1](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2025Q1/1.png?raw=true)

### 物理设计模块

将逻辑网表转换为实际物理实现。

#### 布局

决定逻辑单元在FPGA上的精确位置，根据连接关系、时序要求和物理约束，为每个逻辑单元分配最优位置。

![3](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2025Q1/3.jpg?raw=true)

#### 布线

将所有逻辑单元按照网表连接起来，决定信号在FPGA内部的实际传输路径。这一阶段需要解决资源竞争、信号延迟和拥塞问题，确保设计满足时序要求。

![3](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2025Q1/2.png?raw=true)

### 分析与验证模块

负责确保设计满足所有功能和性能要求，验证时序性能、功耗特性和物理实现正确性。

- **静态时序分析(STA)**：验证设计是否满足时序要求
- **功耗分析**：评估设计在各种工作条件下的功耗性能
- **形式验证**：确保设计的逻辑正确性，验证设计流程各阶段的逻辑等价性
-  **物理验证**：确保设计满足FPGA的物理实现规则，避免资源冲突和不合规的结构

## EDA模块间数据交互

以Vivado为例，EDA工具内部使用多种数据结构进行模块间交互：

### 内部数据结构

**设计数据库**：包含设计网表存储、约束数据库、属性数据库

**物理数据**：包含布局信息、布线数据库、资源分配表

**时序数据**：包含时序图、延迟计算模型、时序约束数据库

### 设计流程中的数据流动

![4](https://raw.githubusercontent.com/lxulxu/lxulxu.github.io/master/assets/images/Y2025Q1/4.png?raw=true)




设计流程中的关键转换点：

| 转换点      | 输入文件    | 输出文件         | Vivado命令/工具            |
| ----------- | ----------- | ---------------- | -------------------------- |
| RTL→综合    | .v/.vhd/.sv | .dcp(综合检查点) | synth_design               |
| 综合→实现   | .dcp(综合)  | .dcp(实现)       | place_design, route_design |
| 实现→比特流 | .dcp(实现)  | .bit/.bin        | write_bitstream            |
| 比特流→硬件 | .bit/.bin   | FPGA配置         | program_hw_devices         |



## 设计文件格式详解

Vivado设计流程中涉及多种文件格式，每种文件在设计流程中扮演特定角色：

### 设计输入文件

#### RTL源文件

* **Verilog/SystemVerilog**:
    * `.v` - Verilog设计文件
    * `.sv` - SystemVerilog设计文件
    * `.vh` - Verilog头文件
* **VHDL**:
    * `.vhd` - VHDL设计文件
    * `.vhi` - VHDL包和组件声明

Verilog源文件示例：

```verilog
// 计数器模块示例
module counter #(
    parameter WIDTH = 8
)(
    input wire clk,
    input wire rst_n,
    output reg [WIDTH-1:0] count
);
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n)
            count <= {WIDTH{1'b0}};
        else
            count <= count + 1'b1;
    end
endmodule
```

####  约束文件

- `.xdc` - Xilinx设计约束文件，包含物理位置约束、时序约束、IO标准定义、时钟定义等

示例：

```tcl
# 时钟约束
create_clock -period 10.000 -name sys_clk_pin -waveform {0.000 5.000} [get_ports clk]

# IO引脚分配
set_property PACKAGE_PIN W5 [get_ports clk]
set_property IOSTANDARD LVCMOS33 [get_ports clk]

# 位置约束
set_property LOC SLICE_X0Y0 [get_cells counter_reg[0]]

# 时序路径组
set_false_path -from [get_clocks clk_slow] -to [get_clocks clk_fast]
```

#### IP相关文件

* `.xci` - IP核配置文件
* `.xco` - IP核定制文件
* `.mif`/`.coe` - 存储器初始化文件

### 综合阶段文件

#### 综合前文件

* `.prj` - 项目文件，列出所有源文件
* `.tcl` - Tcl脚本文件，控制综合流程
* `.xpr` - Vivado项目文件，包含项目配置

项目文件组织：

```
project_1/
├── project_1.cache/
├── project_1.hw/
├── project_1.ip_user_files/
├── project_1.runs/
├── project_1.srcs/
└── project_1.xpr
```

#### 综合后文件

```
源文件(.v/.vhd) → 综合工具
                  ↓
   检查点文件(.dcp) ← 可重新加载的数据点
                  ↓
      网表文件(.edf)
```

* `.dcp` - 保存综合后的设计状态，包含网表和约束信息，可用于增量设计
* `.edf/.edif` - 综合后的网表表示，可与第三方工具交互的标准格式，包含逻辑单元和连接关系

### 实现阶段文件

#### 布局布线文件

```
综合网表(.dcp) + 约束(.xdc) → 
                              ↓
               实现检查点(.dcp) ← 关键数据点
```

* `.dcp` - 保存布局布线后的设计状态，可用于增量设计和ECO
* `.xdef` - Xilinx设计交换格式，主要用于Xilinx工具内部，包含详细的布局布线信息

#### 时序报告文件

* `.rpt` - 文本格式报告文件
* `.rpx` - Vivado原生报告文件

内容示例：

```
Clock Summary
  Clock Source                         Period (ns)  Waveform (ns)    Attributes   Uncertainty (ns)
  ---------------                      ------------  ---------------  -----------  --------------
  clk                                  10.000       {0.000 5.000}    {P}          0.100

Timing Path Analysis
Path Type: Setup (Max at Slow Process Corner)
From: reg1/C
To: reg2/D
Path Group: clk
Path Type: max
Requirement: 10.000ns
Data Path Delay: 4.326ns (Logic: 1.742ns, Net: 2.584ns)
```

### 生成与调试文件

#### 比特流文件

```
实现结果(.dcp) → 
                 ↓
     比特流文件(.bit) ← 关键数据点
                 ↓
      存储格式(.mcs/.bin)
```

* `.bit` - 比特流文件，包含FPGA配置数据和完整实现信息
* `.mcs`/`.bin` - 存储器格式，`.bin`用于JTAG编程，`.mcs`用于Flash编程

#### 仿真与调试文件

- `.tb.v`/`.tb.sv` - 测试台文件
- `.wcfg` - 波形配置文件
- `.wdb` - 波形数据库
- `.ltx` - 逻辑分析器文件
- `.sdf` - 标准延迟格式文件，包含详细时序延迟信息，用于门级时序仿真

SDF文件示例片段：

```
(DELAYFILE
  (SDFVERSION "3.0")
  (DESIGN "counter")
  (DATE "Wed Jun 28 12:00:00 2023")
  (VENDOR "Xilinx")
  (PROGRAM "Vivado")
  (VERSION "2022.2")
  (DIVIDER /)
  (CELL
    (CELLTYPE "FDRE")
    (INSTANCE counter_reg\[0\])
    (DELAY
      (ABSOLUTE
        (IOPATH C Q (0.282:0.348:0.414) (0.282:0.348:0.414))
      )
    )
  )
)
```

### 其他文件格式

- `.twx` - 时序交叉限制文件（XML格式）

* `.pdi` - 可部分可重构设计接口文件（用于动态部分重构）
* `.ll` - 逻辑位置文件
* `.xpe` - Xilinx功耗估算文件
* `.svf` - 串行向量格式（用于JTAG编程）
