# 验证报告（Excel）

导入结束后交付 `.xlsx`，格式对齐历史文档
`Apollo330更新SDK AmbiqSuiteC_5.2.0验证（PASS）.docx`。

## 生成方式

大仓根：

```bash
python3 platform/mcu/mcu_sdk_import/scripts/gen_verify_xlsx.py \
  -o platform/mcu/mcu_sdk_import/out/<平台>_<版本>_验证.xlsx \
  --title "<平台>更新SDK <版本>验证" \
  --version "<版本名>" \
  --version-range "<旧版> -> <新版>" \
  --version-anchor "<VERSION.txt 或 SHA>" \
  --modules-json /tmp/modules.json
```

独立 MCU 仓：将脚本与 `-o` 替换为 `mcu_sdk_import` 实际路径。

`--modules-json` 须实填（允许 `[]`）。**SDK修改说明**取自各笔确认时按子模块列全的修改点或 `git show <commit>`，禁止占位与编造：

```json
[
  {
    "name": "GPIO",
    "path": "sdk/mcu/apollo330P/hal/am_hal_gpio.c\nsdk/mcu/apollo330P/hal/am_hal_gpio.h",
    "desc": "GPIO 配置、上下拉、中断",
    "changes": "1. 高速 pad 50K 配置修复\n2. 驱动强度枚举调整\n3. …",
    "verify": "",
    "owner": "",
    "status": "未测"
  }
]
```

依赖：`openpyxl`（`pip install openpyxl`）。

## 工作表

### 1. 更新说明

版本、升级区间、锚点、芯片、Gerrit/分笔提交、升级必读、已知问题、HM patch 说明。

### 2. 模块变更验证（主表）

| 列 | 含义 | 填写方 |
|---|---|---|
| 模块名称 | 如 GPIO / IOM / BLE Host / Radio 固件 | Agent；可细于 commit 笔 |
| 源码路径 | 主要 `sdk/` 路径 | Agent |
| 功能描述 | 一句话 | Agent |
| **SDK修改说明（修改点）** | 相对旧版的变更与影响，分条列全 | Agent |
| 验证记录 | 测试项与现象 | 测试 |
| 验证人员 | | 测试 |
| 状态 | 未测 / PASS / FAIL / NA | 测试；Agent 默认「未测」，禁止编造 PASS |

**修改点要求：**

- 优先：各笔已列全说明 / `git show <commit>`；Release Notes 仅辅助。  
- 一笔可拆多行（GPIO、IOM…），不必与分笔 1:1；有变更的子模块均须有行。  
- 每条含变更内容与影响（接口、时序、功耗、兼容等）；多处改动分条列全。  
- 无功能变化：标注「无功能变更（仅同步）」。  
- 无变更模块不硬凑行。  
- 单元格写归纳后的修改点，禁止粘贴全量 diff。

### 3. 整机冒烟

编译、开机、显示/触摸、充电、休眠等；结果默认「未测」。

### 4. 蓝牙变更验证

存在 BLE / radio / 固件脚本变更时保留该表；无相关变更则删除该表或整表标 NA。

## Agent 收尾

1. 汇总各笔/commit 中有变更的模块（可细拆）。  
2. 填写源码路径与完整修改点。  
3. 执行 `gen_verify_xlsx.py`（`--modules-json` 实填或 `[]`）。  
4. 交付路径；测试列留空或「未测」。
