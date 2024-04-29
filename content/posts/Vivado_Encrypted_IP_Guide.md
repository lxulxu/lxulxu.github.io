---
title: 'Vivado加密IP'
date: 2022-03-26 18:00:52
tags: [xilinx]
categories: [xilinx]
---
整理 UG1118 Ch6
## 权限管理

- **公共权限(Common Rights)**：适用于所有EDA工具

- **特定权限(Vendor-Specific Rights)**：授予开发者的特定权限（如控制Vivado Logic Analyzer探测器行为），此部分访问权限值覆盖普通权限同名值

- **条件权限(Conditional Rights)**：IEEE-1735-2014 V2 引入，允许不同条件下指定不同访问权限

---
## IEEE 1735 结构

- **定义域(Definition area)**：定义支持的供应商及其访问权限

- **密钥定义(Encrypted Key Definition)**

- **加密负载(Encrypted payload)**：加密IP的Verilog、System Verilog、 VHDL源码

- **纯文本负载(Plain-text payload)**：IP源码未加密部分

以一个完整密钥文件内容为例

```python
`pragma protect version = 2
`pragma protect encrypt_agent = "XILINX"
`pragma protect encrypt_agent_info = "Xilinx Encryption Tool 2021"
`pragma protect begin_commonblock
`pragma protect control error_handling = "delegated"
`pragma protect control child_visibility = "delegated"
`pragma protect control decryption = (activity==simulation)? "false" :"true"
`pragma protect end_commonblock
`pragma protect begin_toolblock
`pragma protect rights_digest_method="sha256"
`pragma protect key_keyowner = "Xilinx", key_keyname= "xilinxt_2021_01", key_method = "rsa", key_public_key
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEApgf7F4kYh0oSFzJBRoRb
nsrAqn24fVbI7xdNG2t9G8pouFfwIXGGmQgYqYZDSmUu0wrrj3ulLvUnjRtmtziJ
1RDOYdyko1SuBEyGT1frzUu9xNitAXxp29hOrVPeKO6kGU81XHJCRJ7uWh7rwoyf
HSUpreifLybt+UT5fyvHu21IxvOR6GHKWaQ4wdL7Txguuyf92XLJIZABEgmuVlPK
/NjJjVRK3c/vMuQLvbihNapkyCiLIWNwDbo9oWXr7NSo3we8u6IlFmP5V8WcOmXZ
/PZqp3QOkY2Jlm1yQt3O8PpU/8qzB7zcHjm3+Q+wB8yUYn/IMwN0t09l2AdBR37G
EwIDAQAB
`pragma protect control xilinx_configuration_visible = "false"
`pragma protect control xilinx_enable_modification = "false"
`pragma protect control xilinx_enable_probing = "false"
`pragma protect control xilinx_enable_netlist_export = "false"
`pragma protect control xilinx_enable_bitstream = "true"
`pragma protect control decryption = (xilinx_activity==simulation)?"false" : "true"
`pragma protect end_toolblock = ""
```

### 版本或其他杂注

```python
`pragma protect version = 2` #遵从IEEE-1735-2014 V2标准

 #标识加密工具
`pragma protect encrypt_agent = "XILINX"
`pragma protect encrypt_agent_info = "Xilinx Encryption Tool 2021"
```

### 公共权限(Common Block Definition)

```python
`pragma protect begin_commonblock
...
`pragma protect end_commonblock
```

公共权限列表

| 名称 | 含义 | 默认值 | 有效值 | Xlinx有效值 |
| :--: | :--: | :--: | :--: | :--: |
| error_handling | 允许展示的错误信息 | "delegated" | "delegated"<br>"srcrefs"<br>"plaintext" | "delegated" |
| runtime_visibility | 运行、tcl或输出报告允许展示的内容 | "delegated" | "delegated"<br>"interface_names"<br>"all_names" | "delegated" |
| child_visibility | 受保护模块实例化未受保护子模块，子模块如何处理error_handling和runtime_visibility<br>显示消息可能会通过受保护区域公开路径名 | "delegated" | "delegated"<br>"allowed"<br>"denied" | "delegated"<br>"allowed" |
| decryption | 是否允许解密模块，一般用于条件权限 | "delegated" | "delegated"<br>"true"<br>"false" | "delegated"<br>"true"<br>"false"<br>**Note:**"delegated"="true" |

### 特定权限(Vendor-Specific Tool Block Definition)
```python
`pragma protect begin_toolblock
...
`pragma protect end_toolblock = ""
```
特定权限列表
| 名称 | 含义 | Xlinx有效值 | 默认值 |
| :--: | :--: | :--: | :--: |
| xilinx_configuration_visible | LUT值在Vivado viewers/editors中是否可见 | "true", "false" | "false" |
| xilinx_enable_modification | 受保护区域网表信息是否可修改 | "true", "false" | "false" |
| xilinx_enable_probing | 用户可否在受保护区域插入或实例化调试探针 | "true", "false" | "false" |
| xilinx_enable_netlist_export | 是否允许导出网表信息 | "true", "false" | "true" |
| xilinx_enable_bitstream | 是否允许生成比特流 | "true", "false" | "true" |
| xilinx_schematic_visibility | 是否允许展示受保护区域模块名称 | "true", "false" | "false" |

### 密钥定义和权限摘要(Key Definition and Rights Digest Method)
强制性定义，包括公钥、密钥相关属性和权限计算方法
```python
`protect key_keyowner = "Xilinx"
`protect key_method = "rsa"
`protect key_keyname = "xilinxt_2019_11"
`protect rights_digest_method = "sha256"
`protect key_public_key
...
```
- 权限定义：&#96;protect control <right> = <rights_expression> 
  e.g. &#96;protect control xilinx_configuration_visible = "false"
- 条件权限定义：&#96;protect control <right> = <condition> ? <true_expression> : <false_expression>
  e.g. &#96;protect control decryption = (xilinx_activity==simulation) ? "false" : "true"

## RTL加密示例
- **VHDL**
```python
`protect version = 2
`protect begin_commonblock
`protect control error_handling = "delegated"
`protect control decryption = (activity==simulation)? "false" : "true"
`protect end_commonblock
`protect begin_toolblock
`protect rights_digest_method=”sha256”
`protect key_keyowner = “Xilinx”, key_method = "rsa", key_keyname =
"xilinxt_2019_11", key_keyowner
...
`protect control xilinx_configuration_visible = "false"
`protect control xilinx_enable_modification = "false"
`protect control xilinx_enable_probing = "false"
`protect control decryption = (xilinx_activity==simulation)? "false" : "true"
`protect end_toolblock = ""
`protect begin
-- Secure Data Block
-- Protected IP source code is inserted here.
...
...
...
`protect end
```
- **Verilog/SystemVerilog**
&#96;pragma protect代替&#96;protect
```python
`pragma protect version = 2
`pragma protect begin_commonblock
`pragma protect control error_handling = "delegated"
`pragma protect control decryption = (activity==simulation)? "false" : "true"
`pragma protect end_commonblock
`pragma protect begin_toolblock
`pragma protect rights_digest_method="sha256"
`pragma protect key_keyowner = "Xilinx", key_method = "rsa", key_keyname =
"xilinxt_2019_11", key_public_key
...
`pragma protect control xilinx_configuration_visible = "false"
`pragma protect control xilinx_enable_modification = "false"
`pragma protect control xilinx_enable_probing = "false"
`pragma protect control decryption = (xilinx_activity==simulation)? "false" : "true"
`pragma protect end_toolblock = ""
`pragma protect begin
// Secure Data Block
// Protected IP source code is inserted here.
...
...
...
`pragma protect end
```

## 权限对vivado工具影响
| 权限/<br>类型 | Simulation | Synthesis | Implementation |
| :--: | - | - | - |
| error_handling/<br>common | <li>delegated = 加密区域内隐藏错误信息引用</li>| <li>delegated = 加密区域内隐藏错误信息引用</li> | <li>NA</li> |
| runtime_visibility/<br>common | <li>delegated = 遵循当前vivado simulator行为-hierarchy viewer隐藏可见性</li> | <li>delegated = 遵循当前vivado默认行为-隐藏可见性</li> | <li>delegated = 所有名字/层级可见</li> |
| child_visibility/<br>common | <li>delegated = 不改变当前simulator行为</li><li>allowed = 展示子模块层级和错误信息</li> | <li>delegated = 不改变当前行为</li><li>allowed = 展示子模块层级和错误信息</li> | <li>delegated = allowed = 不改变当前implementation行为</li> |
| decryption/<br>common | <li>delegated/true = 读入解密文件</li><li>false = 不读入解密文件、报告并停止处理</li> | <li>delegated/true = 读入解密文件</li><li>false = 不读入解密文件、报告并停止处理</li> | <li>delegated/true = 读入解密文件</li><li>false = 不读入解密文件、报告并停止处理</li> |
| xilinx_enable_netlist_export/<br>xilinx-specific | NA | <li>true = write_vhdl/write_verilog输出网表</li><li>false = write_vhdl/write_verilog不运行</li> | <li>true = write_vhdl/write_verilog输出网表</li><li>false = write_vhdl/write_verilog不运行</li> |
| xilinx_configuration_visible/<br>xilinx-specific | NA | <li>true = LUT内容可见</li><li>false = LUT内容不可见</li> | <li>true = LUT内容可见</li><li>false = LUT内容不可见</li> |
| xilinx_enable_modification/<br>xilinx-specific | NA | <li>true = 允许修改网表</li><li>false = 不允许修改网表</li> | <li>true = 允许修改网表</li><li>false = 不允许修改网表</li> |
| xilinx_enable_probing/<br>xilinx-specific | NA | <li>true = 允许用户在受保护区域插入调试探针</li><li>false = 无法插入调试探针</li> | <li>true = 允许用户在受保护区域插入调试探针</li><li>false = 无法插入调试探针</li> |
| xilinx_enable_bitstream/<br>xilinx-specific | NA | NA | <li>true = 可生成位流</li><li>false = 无法生成位流</li> |
注：NA = 对vivado工具无影响

---
## Vivado加密IP
- tcl命令：`encrypt [-key <arg>] -lang <arg> [-quiet] [-verbose] [-ext <arg>] <files>...`
e.g. `encrypt -lang verilog -ext .vp -key keyfile.txt myip.v`

  - **key** - 指定包含Xilinx公钥的RSA密钥文件，如果未指定-key Vivado自动查找密钥。
  - **lang** - 需加密源文件的HDL语言，支持VHDL或verilog。
  - **ext** - 输出加密文件的扩展名，如果未指定源文件会被输出文件覆盖。
  - **&lt;files>** - 加密源文件名称

- Xilinx公钥位置：`<Install_Dir>/Vivado/<version>/data/pubkey/`
- 三方公钥需分别放置在`begin_toolblock/end_toolblock`
  
## Vivado加密Checkpoint
- tcl命令：`write_checkpoint [-key <arg>] -encrypt <file>`
e.g. `write_checkpoint -key keyfile.txt -encrypt my_ip.dcp`
  - **key** - 指定秘钥文件
  - **&lt;files>** - 输出dcp文件名称
*-encrypt选项只有在写出完整的设计检查点时才有用，-cell结合-encrypt没有作用*
---
## 一些技巧
- 多个IP尽量使用相同的密钥文件来优化结果网表
- 使用-ext选项避免无意中覆盖输入文件
- 从工具供应商处获取公钥
- 在一次调用中加密IP的所有文件
- 不要拆分多个加密块/纯文本间的Verilog模块
- 在VHDL语言中，将整个实体和体系结构对放入一个加密块
- 验证加密代码是否正确加载到Vivado和后续的write_verilog重新加密所有安全设计元素
- 验证与三方工具的互操作性