---
title: 'RFdc 驱动'
date: 2023-02-21 14:41:35
tags: [xilinx]
published: true
hideInList: false
feature: 
isTop: false
---
参考PG269 Appx.C

## 数据结构

数据结构UML图如下：

![UML](../assets/images/Y2023Q1/src_v1.22tzp77ow85c.jpg)

## API 功能
### 系统设置相关函数
| 函数原型 | 功能 |
| --- | --- |
| u32 XRFdc_CfgInitialize(XRFdc *InstancePtr, XRFdc_Config *ConfigPtr) | 通过从配置结构复制相关条目来填充驱动程序实例中的适当条目。与 RFdc 驱动程序 API 的任何软件交互都需要此函数，并且必须在使用任何其他 API 函数之前首先调用此函数。 |
| XRFdc_Config *XRFdc_LookupConfig(u16 DeviceId) | 根据设备的唯一 ID 查找设备配置 |
| u32 XRFdc_RegisterMetal(XRFdc *InstancePtr, u16 DeviceId, struct metal_device **DevicePtr) | 使用 Libmetal 注册RFDC |
| u32 XRFdc_StartUp(XRFdc *InstancePtr, u32 Type, int Tile_Id) | 根据 Tile_Id 的请求重新启动片。 如果 -1 作为 Tile_Id 传递，该函数将重新启动所有启用的切片。 现有寄存器设置不会在此过程中丢失或更改。 |
| u32 XRFdc_Shutdown(XRFdc *InstancePtr, u32 Type, int Tile_Id) | 根据 Tile_Id 的请求停止片。 如果 -1 作为 Tile_Id 传递，该函数将停止所有启用的切片。 现有的寄存器设置不会被清除。 |
| u32 XRFdc_Reset(XRFdc *InstancePtr, u32 Type, int Tile_Id) | 通过 Tile_Id 请求重置图块。 如果 -1 作为 Tile_Id 传递，它将重置所有启用的图块。 所有现有的寄存器设置都将被清除并替换为最初配置的设置。 |
| u32 XRFdc_CustomStartUp(XRFdc *InstancePtr, u32 Type, int Tile_Id, u32 StartState, u32 EndState) | 按照通过 Tile_Id 的请求，将 IPSM 从 StartState 运行到 EndState。 如果 -1 作为 Tile_Id 传递，则该函数针对所有启用的图块运行。 如果 StartState 为 XRFDC_STATE_OFF，现有寄存器设置将丢失或更改。 |
| void XRFdc_DumpRegs (XRFdc *InstancePtr, u32 Type, int Tile_Id) | 用于调试，将寄存器中的内容打印到控制台。 |
### ADC 状态指示函数
| 函数原型 | 功能 |
| --- | --- |
| u32 XRFdc_Set/GetDecimationFactor(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, u32 *DecimationFactorPtr) | 设置/返回 RF-ADC 抽取因子 |
| u32 XRFdc_Set/GetDecimationFactorObs(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, u32 *DecimationFactorPtr) | 设置/返回 RF-ADC 观察通道的抽取因子 |
| u32 XRFdc_GetFabWrVldWordsObs(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id, u32 *FabricWrVldWordsPtr) | 返回 RF-ADC 观测通道的写PL数据率 |
| u32 XRFdc_Set/GetFabRdVldWordsObs(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id, u32 *FabricRdVldWordsPtr) | 设置/返回 RF-ADC 观测通道的读PL数据率 |
| u32 XRFdc_Set/GetThresholdSettings(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, XRFdc_Threshold_Settings *ThresholdSettingsPtr) | 设置/读取相应寄存器阈值 |
| u32 XRFdc_GetFIFOStatusObs(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u8 *EnablePtr) | 获取 RF-ADC 观察FIFO的当前状态 |
| u32 XRFdc_Set/GetCalibrationMode(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, u8 *CalibrationModePtr | 设置/获取 RF-ADC 的校准模式 |
| u32 XRFdc_Set/GetCalCoefficients(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, u32 CalibrationBlock, XRFdc_Calibration_Coefficients *CoeffPtr) | 设置/获取通用校准系数 |
| u32 XRFdc_Set/GetCalFreeze(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, XRFdc_Cal_Freeze_Settings *CalFreezePtr) | 设置/获取通用校准冻结设置 |
| u32 XRFdc_Set/GetDither(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, u32 *ModePtr) | 用抖动模式来填充/获取所提供的指针 |
| u32 XRFdc_GetLinkCoupling(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id,u32 *ModePtr) | 获取RF-ADC块的链路耦合模式 |
| u32 XRFdc_Set/GetDSA(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, XRFdc_DSA_Settings *SettingsPtr) | 传递的DSA设置被用来更新/获取相应的块级寄存器，仅适用于 RF-ADC 。 |
| u32 XRFdc_Set/GetSignalDetector(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, XRFdc_Signal_Detector_Settings *SettingsPtr) | 设置信号检测器/读取信号检测器设置，仅适用于 RF-ADC 。 |
| u32 XRFdc_GetNoOfADCBlocks(XRFdc *InstancePtr, u32 Tile_Id) | 获取启用的 RF-ADC 的数量 |
| u32 XRFdc_IsADCBlockEnabled(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id) | 如果请求的RF-ADC被启用，函数返回1；否则返回0。 |
| u32 XRFdc_IsHighSpeedADC(XRFdc *InstancePtr, int Tile) | 返回该瓦片是否为高速瓦片。 |
| u32 XRFdc_IsADCDigitalPathEnabled(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id) | 检查ADC数字路径是否被启用或禁用 |
### ADC 设置状态指示函数
| 函数原型 | 功能 |
| --- | --- |
| u32 XRFdc_ThresholdStickyClear(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, u32 ThresholdToUpdate) | 清除阈值配置寄存器中的sticky位 |
| u32 XRFdc_SetThresholdClrMode(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, u32 ThresholdToUpdate, u32 ClrMode) | 设置阈值清除模式 |
| u32 XRFdc_SetupFIFOObs(XRFdc *InstancePtr, u32 Type, int Tile_Id, u8 Enable) | 启用和禁用RF-ADC的观察通道FIFO |
| u32 XRFdc_SetupFIFOBoth(XRFdc *InstancePtr, u32 Type, int Tile_Id, u8 Enable) | 启用和禁用RF-ADC实际和观察通道的FIFO |
| u32 XRFdc_DisableCoefficientsOverride(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, u32 CalibrationBlock) | 禁用所选块的系数覆盖 |
### DAC 状态指示函数
| 函数原型 | 功能 |
| --- | --- |
| u32 XRFdc_Set/GetInterpolationFactor(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, u32 *InterpolationFactorPtr) | 设置/返回 RF-DAC 的插值因子 |
| u32 XRFdc_Set/GetDecoderMode(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, u32 *DecoderModePtr) | 从相关寄存器中设置/读取解码器模式 |
| u32 XRFdc_GetOutputCurr(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, u32 *OutputCurrPtr) | 获取输出电流 |
| u32 XRFdc_Set/GetInvSincFIR(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, u16 *ModePtr) | 设置/获取InvSinc滤波器的状态 |
| u32 XRFdc_GetDACDataScaler(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, u32 *EnablePtr) | 获取数据缩放器的启用状态 |
| u32 XRFdc_Set/GetDACCompMode(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, u32 *Enable) | 设置/返回传统的DAC输出模式 |
| u32 XRFdc_Set/GetDataPathMode(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, u32 *Mode) | 设置/返回数据路径模式 |
| u32 XRFdc_Set/GetIMRPassMode(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, u32 *Mode) | 设置/返回IMR滤波器模式 |
| u32 XRFdc_GetNoOfDACBlock(XRFdc *InstancePtr, u32 Tile_Id) | 返回启用的RF-DAC的数量 |
| u32 XRFdc_IsDACBlockEnabled(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id) | 如果请求的RF-DAC被启用，该函数返回1；否则返回0。 |
| u32 XRFdc_GetInverseSincFilter(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id) | 如果RF-DAC的反向sinc滤波器被启用，函数返回1；否则返回0。 |
| u32 XRFdc_GetMixedMode(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id) | 返回RF-DAC的混频模式设置 |
| u32 XRFdc_IsDACDigitalPathEnabled(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id) | 检查RF-DAC数字路径是否启用 |
### DAC 设置状态指示函数
| 函数原型 | 功能 |
| --- | --- |
| u32 XRFdc_SetFabWrVldWords(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, u32 FabricWrVldWords) | 写入要为RF-DAC设置的结构数据速率 |
| u32 XRFdc_SetDACVOP(XRFdc *InstancePtr, u32 Tile_Id, u32 Block_Id, u32 uACurrent) | VOP μA电流用于更新相应的块级寄存器 |
### ADC/DAC 状态指示函数
| 函数原型 | 功能 |
| --- | --- |
| u32 XRFdc_GetIPStatus(XRFdc *InstancePtr, XRFdc_IPStatus *IPStatusPtr) | 返回 IP 状态 |
| u32 XRFdc_GetBlockStatus(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id, XRFdc_BlockStatus *BlockStatusPtr) | 返回请求的块状态 |
| u32 XRFdc_Set/GetMixerSettings(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id, XRFdc_Mixer_Settings *MixerSettingsPtr) | 设置/返回相应寄存器的混频器/NCO 设置 |
| u32 XRFdc_Set/GetQMCSettings(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id, XRFdc_QMC_Settings *QMCSettingsPtr) | 设置/返回相关寄存器的 QMC 设置 |
| u32 XRFdc_Set/GetCoarseDelaySettings(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id, XRFdc_CoarseDelay_Settings *CoarseDelaySettingsPtr) | 设置/返回相关寄存器的粗略延迟设置 |
| u32 XRFdc_Set/GetFabClkOutDiv(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u16 *FabClkDivPtr) | 设置/获取PL的时钟分频器 |
| u32 XRFdc_GetFabWrVldWords(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id, u32 *FabricWrVldWordsPtr) | 返回PL写数据率 |
| u32 XRFdc_GetFabRdVldWords(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id, u32 *FabricRdVldWordsPtr) | 返回PL读数据率 |
| u32 XRFdc_GetFIFOStatus(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u8 *EnablePtr) | 返回FIFO的当前状态 |
| u32 XRFdc_Set/GetNyquistZone(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id, u32 *NyquistZonePtr) | 设置/返回奈奎斯特区 |
| u32 XRFdc_GetClockSource(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 *ClockSourcePtr | 返回时钟源 |
| u32 XRFdc_GetPLLConfig(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, XRFdc_PLL_Settings *PLLSettings) | 从寄存器中读取PLL设置 |
| u32 XRFdc_GetPLLLockStatus(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 *LockStatusPtr) | 返回PLL锁定状态 |
| u32 XRFdc_Set/GetClkDistribution(XRFdc *InstancePtr, XRFdc_Distribution_System_Settings *DistributionArrayPtr) | 设置/返回时钟分布设置 |
| u32 XRFdc_GetPwrMode(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id, XRFdc_Pwr_Mode_Settings *SettingsPtr) | 返回电源模式 |
| u32 XRFdc_Get_TileBaseAddr(XRFdc *InstancePtr, u32 Type, u32 Tile_Id) | 返回片的基地址 |
| u32 XRFdc_Get_BlockBaseAddr(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id） | 返回转换器的基地址 |
| u32 XRFdc_GetDataType(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id） | 如果数据类型为实数，函数返回0；否则，返回1。 |
| u32 XRFdc_GetDataWidth(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id) | 返回编程数据宽度 |
| u32 XRFdc_GetMasterTile(XRFdc *InstancePtr, u32 Type) | 返回主瓦片ID |
| u32 XRFdc_GetSysRefSource(XRFdc *InstancePtr, u32 Type) | 返回 SYSREF 的来源 |
| u32 XRFdc_Get_IPBaseAddr(XRFdc *InstancePtr) | 返回IP的基地址 |
| double XRFdc_GetFabClkFreq(XRFdc *InstancePtr, u32 Type, u32 Tile_Id) | 返回已编程的PL时钟频率 |
| u32 XRFdc_IsFifoEnabled（XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id) | 如果FIFO被启用，该函数返回1；否则，返回0。 |
| double XRFdc_GetDriverVersion() | 获取驱动程序的版本 |
| int XRFdc_GetConnectedIData(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id) | 获取I数字数据路径的转换器 |
| int XRFdc_GetConnectedQData(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id) | 获取Q数字数据路径连接的转换器 |
| u16 XRFdc_RDReg(XRFdc *InstancePtr, u32 BaseAddr, u32 RegAddr, u16 Mask) | 对值进行读取和屏蔽 |
| u32 XRFdc_CheckDigitalPathEnabled(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id) | 检查数字路径是否被激活 |
| u32 XRFdc_GetMultibandConfig(XRFdc *InstancePtr, u32 Type, u32 Tile_Id) | 获取多频段配置数据 |
| u32 XRFdc_CheckBlockEnabled(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id) | 检查RF-ADC/RF-DAC块是否被激活或禁用 |
| u32 XRFdc_CheckTileEnabled(XRFdc *InstancePtr, u32 Type, u32 Tile_Id) | 检查RF-ADC/RF-DAC瓦片是否被启用或禁用 |
| u32 XRFdc_GetMaxSampleRate(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, double *MaxSampleRatePtr) | 获取瓦片的最大采样率 |
| u32 XRFdc_GetMinSampleRate(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, double *MinSampleRatePtr) | 获取瓦片的最小采样率 |
### ADC/DAC 设置状态指示函数
| 函数原型 | 功能 |
| --- | --- |
| u32 XRFdc_UpdateEvent(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id, u32 Event) | 当事件源为 Slice 或 Tile 时，使用该函数触发事件的更新事件。 |
| u32 XRFdc_ResetNCOPhase(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id) | 启动当前块相位累加器的NCO相位复位 |
| u32 XRFdc_SetupFIFO(XRFdc *InstancePtr, u32 Type, int Tile_Id, u8 Enable) | 启用和禁用RF-ADC/RF-DAC的FIFO |
| u32 XRFdc_MultiBand(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u8 DigitalDataPathMask, u32 DataType, u32 DataConverterMask) | 设置单频和多频配置 |
| void XRFdc_MultiConverter_Init(XRFdc_MultiConverter_Sync_Config *ConfigPtr, int *PLL_CodesPtr, int *T1_CodesPtr, u32 RefTile) | 初始化多轨同步配置结构，必须在XRFdc_MultiConverter_Sync之前调用。 |
| u32 XRFdc_MultiConverter_Sync(XRFdc *InstancePtr, u32 Type, XRFdc_MultiConverter_Sync_Config *ConfigPtr) | 多片同步 |
| u32 XRFdc_DynamicPLLConfig(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u8 Source, double RefClkFreq, double SamplingRate) | 用于在内部PLL和外部时钟源之间的动态切换，以及为RF-ADC/RF-DAC配置内部PLL。 |
| u32 XRFdc_MTS_Sysref_Config(XRFdc *InstancePtr, XRFdc_MultiConverter_Sync_Config *DACSyncConfigPtr, XRFdc_MultiConverter_Sync_Config *ADCSyncConfigPtr, u32 SysRefEnable) | 启用和禁用sysref |
| u32 XRFdc_ResetInternalFIFOWidth(XRFdc *InstancePtr, u32 Type, u32 Tile_Id,u32 Block_Id) | 重置内部FIFO宽度，以符合RF-ADC/RF-DAC的速率变化和混频器设置。 |
| u32 XRFdc_SetPwrMode(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id, XRFdc_Pwr_Mode_Settings *SettingsPtr) | 设置电源模式 |
| void XRFdc_ClrSetReg(XRFdc *InstancePtr, u32 BaseAddr, u32 RegAddr, u16 Mask, u16 Data) | 修改写入 |
| void XRFdc_ClrReg(XRFdc *InstancePtr, u32 BaseAddr, u32 RegAddr, u16 Mask) | 读取和清除 |
| void XRFdc_SetConnectedIQData(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id, int ConnectedIData, int ConnectedQData) | 设置数字数据路径I和Q的连接数据转换器 |
## 中断部分
| 函数原型 | 功能 |
| --- | --- |
| u32 XRFdc_IntrEnable(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id, u32 IntrMask）| 通过将IntrMask作为输入并写入相应的寄存器位来启用相应转换器的中断 |
| u32 XRFdc_IntrDisable(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id, u32 IntrMask)| 禁用中断 |
| void XRFdc_SetStatusHandler(XRFdc *InstancePtr, void *CallBackRefPtr, XRFdc_StatusHandler FunctionPtr) | 设置状态回调函数，当驱动程序遇到必须报告给上层软件的情况时，会调用这个函数。 |
| u32 XRFdc_IntrClr(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id, u32 IntrMask) | 清除中断 |
| u32 XRFdc_GetIntrStatus(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id, u32 *IntrStsPtr) | 返回中断状态 |
| u32 XRFdc_IntrHandler(u32 Vector, void *XRFdcPtr) | 清除中断源并打印中断原因 |
| u32 XRFdc_GetEnabledInterrupts(XRFdc *InstancePtr, u32 Type, u32 Tile_Id, u32 Block_Id, u32 *IntrMask)  | 启用的中断来填充IntrMask |