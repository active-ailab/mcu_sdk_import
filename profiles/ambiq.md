# Profile：Ambiq（apollo4 / apollo5 等）

适用于 AmbiqSuite。流程见 [../SKILL.md](../SKILL.md)。vendor 与 `sdk/` 同构，按同名目录分笔覆盖。

## 用户指定时使用

用户声明平台为 `apollo4` / `apollo5` 等 Ambiq 系时加载本 Profile。典型目录含 `VERSION.txt`、`mcu/`、`CMSIS/`、`utils/`、`ambiq_ble` / `*_lite`、`cordio*`、`am_sdk_version.h`（仅作笔序参考，**不作平台判定依据**）。

## 识别与标题

- HM 标记（仅 `sdk/`）：`SDK_CHANGE_FOR_HMOS`、`HMI_*`
- 标题：`ref |> [apollo4|apollo5]更新SDK <版本> <模块主题>`

## 版本

1. 自 `VERSION.txt` 解析数字版本（如 `5.2.0`）。  
2. 与 `am_sdk_version.h` 中 `AM_HAL_VERSION_*` 数字核对。  
3. 新 < 旧则中止；相等则按同版本 patch 继续。

## 推荐笔序

| 顺序 | 模块 | 路径 |
|---|---|---|
| 1 | 版本元数据 | `VERSION.txt`、`am_sdk_version.h`、EULA、`Makefile`、`makedefs/`、`pack/` |
| 2 | MCU/HAL | `mcu/`、`CMSIS/`、`utils/`；必要时仓库根 `module.mk` |
| 3 | BLE | `ambiq_ble*`、`cordio*`、radio `devices/`、相关 open-amp |
| 4 | tools/固件 | `tools/**` |
| 5… | 其它已引用模块 | `ambiq_audio/`、`ThinkSi/`、`bootloader/` 等 |
| 末 | 未引用 | 未引用的 devices / third_party / tools 边角 |

版本笔范围宜小；无对应模块则跳过。每笔先做 `diff -rq` 规模统计，按子模块列全修改点并确认，再覆盖并完成 HM 闭环（缺失以 `git show HEAD:<path>` 回补）。

**apollo4**：ble/cordio 与 `*_lite` 按 `module.mk` 变体归 BLE 笔；未引用变体归末笔。  
**多 pack**：MCU 相关目录通常同笔更新；仅更新一侧须明确范围。
