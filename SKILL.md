---
name: mcu-sdk-import
description: >-
  将 vendor MCU SDK 按模块分批导入用户指定的 platform/mcu/<mcu_name>/sdk：
  版本闸门（新≥旧）、同名目录覆盖、保留 sdk 内 HM patch（SDK_CHANGE_FOR_HMOS / HMI_*）、
  每批经确认后提交，并生成含完整模块变更说明的 Excel 验证报告。
  适用于导入/升级 apollo4、apollo5、mhs003s 等 MCU SDK，或同步/更新/分模块导入 SDK 场景；
  平台与目标路径须由用户指定。
disable-model-invocation: true
---

# HMOS MCU SDK 导入

将 vendor MCU SDK 同步至目标仓 `sdk/`，**仅按本 Skill 执行**。历史导入仅供参考，不以既有 commit 为规范。适用于任意 `platform/mcu/<mcu_name>/` 独立 git 工程。

## 核心

**按模块分批提交（分笔）**：禁止整包单次提交。按功能域拆分为多次 commit（如版本元数据、HAL/MCU、协议栈、tools；未引用内容置于末笔）。  
每笔对应 `sdk/` 下若干同名顶层目录（如 `mcu/`、`CMSIS/`、`ambiq_ble/`），不得混入无关路径。

**目录同构**：vendor 与仓库 `sdk/` 结构一致（允许微小差异）；按同名目录覆盖，以目录内文件差异界定范围与变更说明。结构显著不一致时中止并请示，禁止臆测路径映射。

**约束**：版本单调非降（`新 ≥ 旧`，相等视为同版本 patch）；保留 `sdk/` 内 `SDK_CHANGE_FOR_HMOS` / `HMI_*`（覆盖丢失须回补）；变更说明按子模块列全并经确认后再覆盖与提交；默认不输出全量 diff 正文。

**路径与平台配置**：平台与目标路径**须由用户指定**，禁止根据目录特征自行推断。须确认：

- 平台名称（如 `apollo4` / `apollo5` / `mhs003s`）
- 目标路径 `platform/mcu/<mcu_name>/sdk`（及对应 MCU 仓库）

用户指定平台后加载对应 Profile；未提供匹配 Profile 时，按 [buckets.md](buckets.md) 依顶层目录拟定提交批次。分笔路径通则见 [buckets.md](buckets.md)。

| 用户指定平台 | Profile |
|---|---|
| `apollo4` / `apollo5` 等 Ambiq 系 | [profiles/ambiq.md](profiles/ambiq.md) |
| `mhs003s` 等自研 HAL | [profiles/mhs003s.md](profiles/mhs003s.md) |
| 其它 / 无对应 Profile | [buckets.md](buckets.md) |

## 执行步骤

### 1. 锁定目标

向用户确认（缺一不可则先问清再继续）：

- **平台名称**（如 `apollo5`、`mhs003s`）
- **目标路径** `platform/mcu/<mcu_name>/sdk`（及 MCU 仓库根）
- 与 `sdk/` 同构的 vendor 根目录
- 旧/新版本、导入范围（可限定模块）、可选测试文档

按用户指定的平台加载 Profile（见上表），不得根据树内文件自行改判平台。

```bash
git status && git rev-parse --show-toplevel
ls sdk | head
ls <vendor_sdk> | head
```

工作区宜干净；默认不修改 `hmos/` / `newlib/` / `system_*.c`（明示要求适配除外）。范围外目录整笔跳过；末笔未引用内容可按需省略。

### 2. 版本闸门（整轮一次；未通过禁止拷贝）

分别读取仓库与 vendor 版本：优先 `VERSION.txt`，其次版本头文件（如 `am_sdk_version.h`），再次包名或由用户提供。

比较时提取数字版本（如 `5.2.0`），禁止整行字符串直接比较；缺位按 0 补齐。

- 新 < 旧，或任一侧无法解析可比较版本：中止。
- 数字版本相等：按同版本 patch 继续；commit message 注明 vendor 包名或 `VERSION.txt` 原文/hash。
- 新 > 旧：通过；记录升级区间，供版本笔与报告使用。

### 3. 拟定提交批次（依顶层目录）

对照两侧顶层目录，结合 profile 与 `module.mk` / `Config.in`；对拟变更目录执行 `diff -rq` 规模统计后，提交批次草案供确认：

```bash
ls sdk
ls <vendor_sdk>
rg -n 'sdk/|SDK_|\$\(SDK' module.mk Config.in 2>/dev/null | head -80
diff -rq sdk/<dir> <vendor_sdk>/<dir> | head -50
```

推荐顺序：

1. 版本/元数据（范围宜小）  
2. 工程已引用模块（按模块分笔）  
3. 未引用内容（末笔；可按范围省略）  

目录增减或更名等微小差异：列明并请示归属。禁止以「拷贝前空 `git diff`」作为分批依据。

### 4. 记录 HM 基线

导入前保存 `sdk/` 内标记命中列表（路径即可）：

```bash
rg -n 'SDK_CHANGE_FOR_HMOS|HMI_' sdk
```

仅扫描 `sdk/`；`hmos/` 中的 `HMI_*` 不视为 SDK patch。

### 5. 逐笔同步与提交

每笔流程：变更说明完备并确认，随后覆盖、校验 HM、再提交。

1. **范围**：列出本笔同名路径（如 `sdk/mcu/`、`sdk/CMSIS/`）。  
2. **差异**：统计 vendor↔仓库文件增删改，整理确认材料。  
3. **确认材料**（禁止默认输出全量 diff；变更说明须按子模块列全）：  
   - 主题与规模（变更文件数；可得则附增删行数）  
   - 完整变更文件清单（可按子目录/子模块分组；过长可附文件）  
   - **修改点（按子模块列全）**：每个有变更的子模块（如 GPIO、IOM、BLE Host）分条说明变更内容与影响；无功能变化则标注「仅同步」。依据文件差异、公共头文件与 Release Notes 归纳，禁止遗漏或抽样摘录。末笔可按目录概括，但仍须覆盖本笔全部变更范围  
   - HM 风险（基线命中数；冲突项单列）  
   - 删除候选（仓库有而 vendor 无；**默认保留**，须明示才删除）  
   - 完整 commit message；首笔或总览附版本闸门结论  
   - 用户指定、HM 冲突或疑似 API 破坏时，再展开相关 diff 片段  
4. **确认后覆盖**：按目录自 vendor 拷贝至仓库对应路径；禁止对整棵 `sdk/` 树盲目 rsync。
5. **HM 闭环**：复扫本笔路径。  
   - 基线存在而覆盖后缺失：以 `git show HEAD:<path>`（或覆盖前备份）取回并回补 HM 改动。  
   - 与 vendor 同文件冲突：标明冲突点，禁止静默丢弃 HM，确认后定稿。  
6. **撤销**：覆盖后若拒绝本笔，对本笔路径执行 `git restore --staged --worktree -- <paths>`（或等价），禁止强行提交。  
7. **提交**：确认无误后 `git add` + `commit`（默认不 push、不改 git config、不 force、不跳过 hook）。  
8. **留存**：按子模块记录本笔全部修改点，供 Excel；提交后不以 vendor `diff -rq` 作为唯一依据。

```text
ref |> [<mcu>]更新SDK <版本名> <模块主题>
ref |> [<mcu>]同步sdk <主题简述>
```

末笔主题可用 `HM工程未使用的文件`。功能变更笔可附 `[主要修改]` / `[测试文档]`。`Change-Id` 由 hook 生成则勿伪造。

注意：`module.mk` 随触发该模块的笔提交；被迫修改 `hmos/` 须单列并获准；注意二进制与密钥样例。

### 6. 收尾与 Excel 验证报告

检查清单：各笔已确认或已跳过；版本锚点与闸门一致；HM 相对基线已回补或已确认；`hmos/` 无意外变更；无未归类残留。

按 [report-template.md](report-template.md) 生成 `.xlsx`。修改点优先取自各笔已列全的说明或 `git show <commit>`（Release Notes 仅辅助）。报告行可按 GPIO/IOM 等子模块拆分，**不必与 commit 笔一一对应**。

```bash
# 大仓根（含 platform/mcu/）:
python3 platform/mcu/mcu_sdk_import/scripts/gen_verify_xlsx.py \
  -o platform/mcu/mcu_sdk_import/out/<平台>_<版本>_验证.xlsx \
  --title "<平台>更新SDK <版本>验证" \
  --version "<版本>" \
  --version-range "<旧> -> <新>" \
  --modules-json /tmp/modules.json

# 独立 MCU 仓：改用 mcu_sdk_import 实际路径
python3 <path-to>/mcu_sdk_import/scripts/gen_verify_xlsx.py \
  -o <path-to>/mcu_sdk_import/out/<平台>_<版本>_验证.xlsx \
  --modules-json /tmp/modules.json
```

须传入实填 `--modules-json`（允许 `[]`）；禁止占位示例行。工作表：`更新说明`、`模块变更验证`（主表）、`整机冒烟`、`蓝牙变更验证`（无 BLE 相关变更可标 NA 或删除）。验证状态默认「未测」；禁止编造 PASS。

## 附加

[buckets.md](buckets.md) · [profiles/ambiq.md](profiles/ambiq.md) · [profiles/mhs003s.md](profiles/mhs003s.md) · [examples.md](examples.md) · [report-template.md](report-template.md)
