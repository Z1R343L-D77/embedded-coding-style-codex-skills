# Skill Copy (Embedded Coding Style)

> [!IMPORTANT]
> 使用本 Skill 前，请确认目标 `.c/.h` 文件编码为 `GB2312`（code page 936）。
> 本 Skill 要求严格 `GB2312` 读写（不允许自动识别和 fallback），否则会立即中止并回滚。

## 版本信息

- 当前版本：`v1.2.0`
- 更新日期：`2026-05-07`
- 对齐来源：`embedded-coding-style/SKILL.md`

## 本次更新

1. 添加函数名变量名的矫正
2. 添加使用NULL时，添加#include <stdio.h>
3. 添加关键函数与程序的注释

## Skill 定位

`skill-copy` 用于在嵌入式 C 项目中执行统一、可复用、可验证的 MISRA 风格代码治理，重点覆盖：

1. 作用域可选（`driver/bsp/app/组合/全量`）且禁止越界修改。
2. 严格编码策略（`GB2312 + CRLF + 文件尾 CRLF+CRLF`）。
3. 注释、括号、`switch/case`、宏安全、可见性等硬规则统一。
4. 头文件结构规范化（`#include -> #define -> typedef -> 函数声明 -> 变量声明`）。
5. 可验证输出（`rg` 检查、尾字节检查、编码异常检查、头文件顺序抽查）。

## 关键规则

1. 编码必须为 `GB2312`（code page 936）。
2. 行尾必须为 `CRLF`。
3. 文件结尾必须保留一个空行（`CRLF+CRLF`）。
4. 每个 `.c/.h` 文件开头必须包含标准文件头注释（`@file/@author/@brief/@version/@date`）。
5. 函数文档注释使用 `/** ... */`，并包含 `@brief @param @retval`。
6. 行内/块说明注释使用 `/* ... */`。
7. 注释语言使用中文，避免 AI 模板化措辞。
8. 标准库头文件使用 `<>`（如 `<stdint.h>`、`<stdio.h>`）。
9. 项目头文件使用 `""`。
10. 头文件必须保留 include guard，并包含 C++ 兼容块（`extern "C"`）。
11. 左花括号必须换行。
12. 花括号缩进与控制语句对齐。
13. `if/while/else if/else` 即使单行也必须带花括号（`for/switch/do` 也保持完整块）。
14. `switch` 必须包含 `default`。
15. 每个 `case` 必须显式 `break/return`（或受控跳转）。
16. 倾向显式比较，避免不清晰的隐式真值判断。
17. 整数字面量按需添加显式后缀（如 `U`、`UL`）。
18. 宏参数与宏表达式必须加括号。
19. 多语句宏必须使用 `do { ... } while (0)`。
20. 文件私有符号优先使用 `static` 限制可见性。
21. 对外可达路径中的指针参数在解引用前必须空指针检查。
22. 避免 magic number，按项目风格提取为宏或常量。
23. 仅做风格治理时不得修改时序、ISR、状态机行为。
24. 文本读写必须强制 `GB2312`，禁止自动识别和任何 fallback 编码。
25. 读取路径必须严格 `GB2312` 解码；解码失败立即停止。
26. 写入路径必须严格 `GB2312` 编码；编码失败立即停止并回滚。
27. 清理明显 AI 风格注释，改写为自然工程语义注释。
28. 结构体指针形参统一命名为 `handle`，避免 `p_ctrl/p_xxx` 混用。
29. `.h` 内部顺序必须为：`#include -> #define -> typedef -> 函数声明 -> 变量声明`。
30. 函数关键逻辑必须补充清晰中文注释，避免空洞注释。
31. 函数名使用 UpperCamelCase（如 `GPIO_Init()`）。
32. 变量名使用 lowercase snake_case（如 `uart_handle`）。
33. 宏名使用 UPPER_SNAKE_CASE（如 `UART_BUFFER_SIZE`）。
34. 结构体类型名使用 lowercase snake_case + `_t`（如 `uart_config_t`）。
35. 枚举类型名使用 lowercase snake_case + `_e`（如 `uart_status_e`）。
36. 若文件中用 `NULL` 做结构体指针保护，需确保该文件 include 区存在 `#include <stdio.h>`。
37. 注释中的关键步骤说明必须直接描述具体作用，不使用“步骤1：”这类编号前缀，且禁止出现“关键步骤：”字样。

## 执行流程（对应 SKILL.md）

1. 初始化项目上下文，确认目标目录可访问。
2. 选择处理范围（默认全量），仅枚举并处理选定范围内 `.c/.h` 文件。
3. 小步编辑：文件头 -> 括号规范 -> 注释治理 -> 签名统一 -> include/兼容块 -> 头文件顺序。
4. 严格编码预检后写回：强制 `GB2312` 读写、`CRLF`、文件尾 `CRLF+CRLF`。
5. 执行验证清单并输出结果；若任一编码异常，立即停止并回滚。

## 目录结构

```text
.
|-- embedded-coding-style/
|   |-- SKILL.md
|   |-- SKILL copy.md
|   `-- agents/
|       `-- openai.yaml
|-- LICENSE
`-- README.md
```

## Skill 入口信息

- Skill 名称：`skill-copy`
- 显示名称：`Embedded Coding Style`
- 简介：`Help with Embedded Coding Style tasks`

## 在 Codex 中使用

任务中建议明确给出：

1. 项目根目录。
2. 处理范围（`driver` / `bsp` / `app` / 组合 / 全量）。
3. 备份路径（建议提供）。

示例：

```text
使用 skill-copy。
项目根目录是 <project_root>。
只处理 driver 和 bsp。
按 SKILL.md 规则执行并输出检查结果。
```

## 验证建议

1. 用 `rg` 检查无括号分支、文件头、C++ 兼容块。
2. 检查尾字节是否为 `13,10,13,10`。
3. 抽查中文注释可读性与工程语义。
4. 抽查 `.h` 内部顺序是否符合统一规范。
5. 用 `rg -n "\?\?\?|\uFFFD"` 检查是否存在乱码痕迹。

## 许可证

本仓库使用 [LICENSE](./LICENSE) 中声明的许可证。
