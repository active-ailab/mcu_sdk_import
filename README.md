# mcu-sdk-import

这是一个将 vendor MCU SDK **按模块分批**导入 HMOS `platform/mcu/<mcu_name>/sdk` 的SKILL。

## 准备&输入

1. 下载本SKILL `mcu-sdk-import`

2. 下载原厂SDK，解压好文件如`E:\Apollo330B\AmbiqSuiteC_5.2.0\`

## 怎么提问 / 怎么交代任务（示例）

在Agent里面输入以下提问示例，**务必指定新SDK路径+要更新的平台路径**。

示例1：:
```text
按 platform/mcu/mcu_sdk_import 更新 apollo5 SDK。
新SDK：E:\Apollo330B\AmbiqSuiteC_5.2.0（与 sdk 同构）
更新目标：platform/mcu/apollo5/sdk
范围：全量（或仅 HAL / BLE）
```

示例2：
```text
按 platform/mcu/mcu_sdk_import 规则，
导入新SDK E:\Apollo330B\AmbiqSuiteC_5.2.0 到 platform/mcu/apollo5/sdk。
```

导入完成后，会在`platform/mcu/mcu_sdk_import/out`生成更`新说明+测试用例报告excel表格`。

