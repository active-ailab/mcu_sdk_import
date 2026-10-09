# mcu-sdk-import

将 vendor MCU SDK **按模块分批**导入 HMOS `platform/mcu/<mcu_name>/sdk` 的 SKILL。

导入完成后交付 **飞书云文档验证报告**：复制并填写通用模板  
[MCU SDK 导入验证通用模板](https://zepp.feishu.cn/docx/HK6HdjyUco1WTxxxVh0czcfrnlf)  
（按本轮真实变更填表；无变更域删除对应节；checklist/整机相关项「未测」、无关项「无修改」；禁止编造 PASS。）

## 准备&输入

1. 下载本 SKILL `mcu-sdk-import`
2. 下载原厂 SDK，解压好文件如 `E:\Apollo330B\AmbiqSuiteC_5.2.0\`
3. 飞书能力：按 [飞书 CLI 安装指南](https://open.feishu.cn/document/no_class/mcp-archive/feishu-cli-installation-guide.md) 安装并登录 `lark-cli`

## 怎么提问 / 怎么交代任务（示例）

在 Agent 里输入以下提问示例，**务必指定新 SDK 路径 + 要更新的平台路径**。

示例1：

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

导入完成后，Agent 会：

1. `python3 scripts/publish_from_template.py --title "…" -o out/…` 复制通用模板  
2. 按模板内【TEMPLATE·AGENT】填写各表与【待填】  
3. 删除模板指令层后返回报告链接  

详见 [report-template.md](report-template.md)、[SKILL.md](SKILL.md)。
