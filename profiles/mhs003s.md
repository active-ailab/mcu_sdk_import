# Profile：mhs003s（及相近自研 HAL）

适用于 `drivers` + `include` + `riscv`。流程见 [../SKILL.md](../SKILL.md)。vendor 与 `sdk/` 同构，按同名目录分笔覆盖。

## 用户指定时使用

用户声明平台为 `mhs003s` 或同类自研 HAL 时加载本 Profile。典型目录含 `drivers/`、`include/`、`riscv/`（仅作笔序参考，**不作平台判定依据**）。

## 识别与标题

- 典型子树：`drivers/`、`include/`、`riscv/`、`section/`、`tools/`
- 常无 `VERSION.txt`；标题示例：`ref |> [mhs003s]同步sdk <主题>`
- HM 标记（仅 `sdk/`）：`SDK_CHANGE_FOR_HMOS` / `HMI_*`

## 版本

向用户取得可比较版本；有版本宏则数字交叉核对。无法取得则中止。

## 推荐笔序

| 顺序 | 模块 | 路径 |
|---|---|---|
| 1 | 版本/头文件基线 | 版本宏或相关 `include/` |
| 2 | HAL（可按外设再拆） | `drivers/`、`include/`、`section/` |
| 3 | RISC-V | `riscv/` |
| 4… | 其它已引用子系统 | 如 `mmc/`、`gpu/`、`tools/` |
| 末 | 未引用 | 未引用部分 |

引用以 `module.mk` 为准。每笔先比对目录内差异，按子模块列全修改点并确认，再覆盖并完成 HM 闭环。默认不修改 `hmos/`、`newlib/`。
