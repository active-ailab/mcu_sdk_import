---
name: mcu-sdk-import
description: >-
  将 vendor MCU SDK 按模块分批导入用户指定的 platform/mcu/<mcu_name>/sdk：
  版本闸门（新≥旧）、同名目录覆盖、保留 sdk 内 HM patch（SDK_CHANGE_FOR_HMOS / HMI_*）、
  每批经确认后提交，并基于通用飞书模板复制填写验证报告（内嵌检查/验证表）。
  适用于导入/升级 apollo4、apollo5、mhs003s 等 MCU SDK，或同步/更新/分模块导入 SDK 场景；
  平台与目标路径须由用户指定。飞书交付依赖本机 lark-cli（FeiShuSkill）。
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
8. **留存**：按子模块记录本笔全部修改点，供飞书验证报告；提交后不以 vendor `diff -rq` 作为唯一依据。

```text
ref |> [<mcu>]更新SDK <版本名> <模块主题>
ref |> [<mcu>]同步sdk <主题简述>
```

末笔主题可用 `HM工程未使用的文件`。功能变更笔可附 `[主要修改]` / `[测试文档]`。`Change-Id` 由 hook 生成则勿伪造。

注意：`module.mk` 随触发该模块的笔提交；被迫修改 `hmos/` 须单列并获准；注意二进制与密钥样例。

### 6. 收尾与飞书验证报告

检查清单：各笔已确认或已跳过；版本锚点与闸门一致；HM 相对基线已回补或已确认；`hmos/` 无意外变更；无未归类残留。

按 [report-template.md](report-template.md) **复制通用模板并填写**，模板以线上为准：  
[MCU SDK 导入验证通用模板](https://zepp.feishu.cn/docx/HK6HdjyUco1WTxxxVh0czcfrnlf)  
（表结构以该文档当前版本为准，含用户手动调整；禁止自行改模板底表结构。）

**强制流程：**

1. **复制模板**（不要从零建空文档/空表）：

```bash
python3 scripts/publish_from_template.py   --title "<平台>更新SDK <版本>验证"   -o out/<平台>_<版本>_验证   --as user
```

记下输出的报告 URL 与 `*_state.json` 中的 sheet-id。

2. **按【TEMPLATE·AGENT】填写**（文档置顶 Callout + 各节【表填规则】）：
   - 替换全部 `【待填】`；可选项无则写「无」
   - SDK checklist / 整机：只改规则允许的列；**与本轮 diff 相关的行结果默认「未测」**，**无关修改项选「无修改」**（勿整表一律「未测」）；禁止编造 PASS。相关判定：以 checklist **A 列模块组**是否被本轮变更命中为准，明确联动项一并标相关；底稿行默认保留
   - 有 HAL/总线变更 → 填「总线/外设」表（先删`（示例）`行）；**无变更删除整节**
   - 有 BLE/Cordio/tools 固件变更 → 填「蓝牙」表；**无变更删除整节**
   - 变更摘要/风险/验证点必须基于本轮各笔说明与 diff

3. **收尾删除模板指令层**（最终报告给人读）：
   - 删除【TEMPLATE·AGENT】Callout 整段
   - 删除所有【表填规则】/【AGENT·条件节】行与「人工使用说明」
   - 确认无残留【待填】

4. 交付报告 URL；`workbook-info` 确认表尺寸贴合数据。

`lark-cli`：[飞书 CLI 安装指南](https://open.feishu.cn/document/no_class/mcp-archive/feishu-cli-installation-guide.md)。WSL：`cmd.exe /c "lark-cli ..."`。

仅当模板复制失败时，才回退 `scripts/gen_verify_feishu.py`（须自行裁剪 200×20）。

## 附加

[buckets.md](buckets.md) · [profiles/ambiq.md](profiles/ambiq.md) · [profiles/mhs003s.md](profiles/mhs003s.md) · [examples.md](examples.md) · [report-template.md](report-template.md)

