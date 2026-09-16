# 分笔路径归类

相对 `platform/mcu/<mcu_name>/`。按 [SKILL.md](SKILL.md) 分模块归类。  
前提：vendor 与 `sdk/` 目录同构（允许微小差异）。平台由用户指定后加载对应 Profile 笔序；无匹配 Profile 时依顶层目录与 `module.mk` 拟定。已引用模块优先提交，未引用置于末笔。

## 树边界

| 路径 | 导入策略 |
|---|---|
| `sdk/` | 主同步目标；与 vendor 同名目录覆盖 |
| `module.mk` / `Config.in` | 仅必要时随对应模块笔 |
| `hmos/`、`newlib/`、`system_*.c` | 默认不修改 |
| `platform/mcu/mcu_sdk_import/` | 本 Skill；不随 SDK 提交 |

## 引用判定

`module.mk` / `Config.in` 中有引用的路径，归入对应模块笔；无引用则归入末笔「未引用」。无法判定时请示。

## 版本锚点

| 路径 | 说明 |
|---|---|
| `sdk/VERSION.txt` | 优先；解析数字版本 |
| `sdk/mcu/am_sdk_version.h` 等 | 与 VERSION 数字交叉核对 |
| 包名 / 用户声明 | 无版本文件时必取 |

## 无 profile 时

1. 对照两侧顶层目录求交/差；结合 `module.mk` 标注已引用目录。  
2. 默认同名目录成笔（可按功能域合并）；未引用置于末笔。用户确认后再覆盖。  
3. 微小结构差异列明请示，禁止臆测映射。

分批依据：目录对照 + `diff -rq` 规模统计；禁止依赖拷贝前空 `git diff`。  
逐笔确认材料：规模、完整文件清单、按子模块列全的修改点；默认不输出全量 diff。  
覆盖后可用以下命令检查残留归类：

```bash
ls sdk
diff -rq sdk/<dir> <vendor_sdk>/<dir> | head -50
git diff --name-only | awk -F/ '{
  if ($1=="sdk" && NF>=2) print $1"/"$2;
  else print $1;
}' | sort | uniq -c | sort -rn
```

## 模块与典型路径

| 模块 | 常见路径（相对 `sdk/`） |
|---|---|
| 版本/元数据 | `VERSION.txt`、version 头、EULA、`makedefs/`、`pack/`（范围宜小） |
| HAL / MCU | Ambiq：`mcu/`、`CMSIS/`、`utils/`；自研：`drivers/`、`include/`、`riscv/`… |
| 协议栈 | `ambiq_ble*`、`cordio*`、radio devices… |
| tools/固件 | `tools/**` |
| 其它 | audio、GPU、bootloader、crypto、mmc…（有引用则独立成笔） |
| 未引用（末笔） | 未引用的 devices / third_party / 示例等 |

## 冲突归类

1. 同一文件不得跨两笔。  
2. 版本头归版本笔。  
3. `module.mk` 归触发它的模块笔。  
4. 仓库有而 vendor 无：默认保留，待确认。
