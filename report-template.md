# 验证报告（飞书云文档 · 通用模板）

导入结束后交付 **一篇飞书 Docx 验证报告**。

**唯一模板（以飞书线上为准，表结构以用户当前手动调整版为准）：**  
[MCU SDK 导入验证通用模板](https://zepp.feishu.cn/docx/HK6HdjyUco1WTxxxVh0czcfrnlf)

安装 `lark-cli`：[飞书 CLI 安装指南](https://open.feishu.cn/document/no_class/mcp-archive/feishu-cli-installation-guide.md)。WSL：`cmd.exe /c "lark-cli ..."`。

## 流程总览

```text
SDK 分笔提交完成
  → scripts/publish_from_template.py   # 复制模板 → 得到报告 URL + sheet 映射
  → Agent 按【TEMPLATE·AGENT】填写【待填】与各表
  → 无变更域删除对应条件节
  → 删除模板指令层（Agent Callout / 表填规则 / 人工说明）
  → 交付报告 URL
```

**禁止**再从零 `docs +create` + 自建空表拼装（易出现 200×20 空表、列结构漂移）。  
`scripts/gen_verify_feishu.py` 仅作无法复制模板时的回退。

## 质量红线

1. **以模板表为准**：复制后保留模板行列结构；只改【表填规则】允许的列；用户在模板上的手动调整视为权威。
2. **无变更则删节**：无 HAL/总线变更 → 删除「总线/外设 变更验证」整节；无 BLE/Cordio/tools 固件变更 → 删除「蓝牙 变更验证」整节。
3. **变更表必须 AI 归纳**：`变更摘要 / 影响范围风险 / 建议验证点` 基于本轮 diff、history、vendor Readme、各笔确认材料；禁止空话。
4. **检查结果分两档**：与本轮 diff/变更域相关的行默认「未测」；无关修改项选「无修改」（禁止整表一律「未测」）。禁止编造 PASS/通过。  
   **相关判定**：以 SDK checklist **A 列模块组**是否被本轮变更命中为准；明确联动项（如 Controller sleep↔HCI/IPC）一并标相关。底稿行默认保留，勿为无关项删整表。
5. **表尺寸贴合数据**：总线/蓝牙写入后裁掉多余空行；`workbook-info` 的行列 = 有数据的行列。
6. **最终报告给人读**：必须删除【TEMPLATE·AGENT】Callout、所有【表填规则】/【AGENT·条件节】行、「人工使用说明」；不得残留【待填】。

## SKILL 模块域 → 填哪张表

| SKILL 域 | 写入位置 |
|---|---|
| 版本/元数据 | 更新说明「本信息」/版本锚点 |
| HAL/MCU（mcu/CMSIS/utils 或 drivers…） | 总线表 + SDK checklist + 整机关联项 |
| 协议栈/BLE（ambiq_ble/cordio/Radio/HCI…） | 蓝牙表 + checklist + 整机蓝牙项 |
| tools/固件 | 蓝牙表固件行 + 整机固件刷写项 |
| 其它已引用（audio/GPU/…） | 按影响归总线或蓝牙；checklist 有则勾选 |
| 未引用（末笔） | 通常不进验证表 |

## 一键复制模板

```bash
python3 scripts/publish_from_template.py \
  --title "<平台>更新SDK <版本>验证" \
  -o out/<平台>_<版本>_验证 \
  --as user
# 可选：--folder-token <目录token>  ；默认复制到 my_space
# 产出：out/..._state.json 、 out/..._checklist.md
```

脚本会：

1. `lark-cli drive +copy` 复制通用模板并命名为 `--title`
2. `docs +fetch` 解析内嵌 sheet-id（**复制后 id 会变，以 state.json 为准**）
3. 写出 Agent 填写清单

## Agent 填写要点

打开报告 URL / `*_checklist.md`，对照文档顶部【TEMPLATE·AGENT】：

| 位置 | 动作 |
|---|---|
| 标题 | 已用 `--title`；正文 title 若仍含「通用模板」则改掉 |
| 代码提交 Callout | 替换【待填：Gerrit…】 |
| 修改记录 | 追加本轮一行 |
| 原厂资料表 | 只填 B 检查结果、C 备注 |
| SDK checklist | 只填 C/D/E；相关行「未测」+备注，无关行「无修改」（底稿保留，勿删整表） |
| Release Notes | 替换全部【待填】 |
| 整机表 | 只填 D/E/F；A/B/C 保留；相关「未测」、无关「无修改」 |
| 总线/蓝牙表 | 删`（示例）`行 → 按本轮子模块新增并填满各列 |

### 总线 / 蓝牙行字段（与模板列一致）

- 总线/外设 或 模块/子系统  
- 变更摘要（AI）  
- 主要变更文件  
- 影响范围/风险（AI）  
- 建议验证点（AI，可注 SDVT）  
- Owner / Owner验证 / 验证报告（可空）

## 收尾删除

1. 删除【TEMPLATE·AGENT】Callout（整段）  
2. 删除所有【表填规则…】【AGENT·条件节…】行  
3. 删除「人工使用说明」整段  
4. 自检：无【待填】残留；条件节已按变更保留/删除；表尺寸合理  

交付：报告 URL。

## 回退（仅当模板复制失败）

才使用 `scripts/gen_verify_feishu.py` 从零建稿，且必须裁剪默认 200×20，质量对齐
[历史样例](https://zepp.feishu.cn/docx/XOWmdB3NWoq64GxT95DcHM2enMe)。
