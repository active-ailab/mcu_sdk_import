# 过往示例（仅参考）

**现行规范以 [SKILL.md](SKILL.md) 为准**（按模块分笔）。下列历史提交可能不符合现行规范，禁止照搬。

## apollo5 · AmbiqSuiteC_5.2.0

可参考主题拆分（版本 / BLE / MCU / tools / 未引用）与 `[主要修改]` 写法。  
应避免：版本笔混入大量非版本文件。

| Commit | Title（摘要） |
|---|---|
| `8424b8c` | 版本号（历史偏杂，勿照搬） |
| `ae7de70` | BLE |
| `d8a75d2` | MCU 总线相关 |
| `1858cfb` | ble 固件等升级脚本 |
| `4204595` | 未使用的文件（现行置于末笔） |

```text
ref |> [apollo5]更新SDK AmbiqSuiteC_5.2.0 MCU总线相关

[主要修改]
同步 … HAL/CMSIS/utils：
…
```

## apollo4 / mhs003s / 其它

按对应 profile 分模块提交；未引用内容置于末笔。标题格式遵循仓史或用户指定。
