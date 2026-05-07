---
name: skill-copy
description: "Initialize embedded project context, then apply and verify MISRA-like C coding style rules with selectable scope, ANSI encoding policy, brace policy, header/doc comment policy, and verification checklist."
---

# Embedded Coding Style Skill

Initialize project context first, then apply a strict, repeatable embedded C style policy to selected scope.

---

## Initialization + Scope Selection

### Step 0: Initialize Project Context (Mandatory)

Before any edit, initialize the project and gather context:

```powershell
Get-Location
Get-ChildItem -Path <project_root>
```

If backup is provided, verify backup path exists before edits.

### Step 1: Select Scope (Mandatory)

Choose one scope per run:

1. Driver layer only: `<project_root>/<driver_dir>`
2. BSP layer only: `<project_root>/<bsp_dir>`
3. App layer only: `<project_root>/<app_dir>`
4. Combined subset: any two scopes
5. Full target: all selected core scopes

Default behavior: if user does not specify scope, use full target scope.

Do not edit outside selected scope.

---

## Hard Rules

1. Encoding: `GB2312` (code page 936), mandatory.
2. Line ending: `CRLF`.
3. File end: one blank line (`CRLF+CRLF`).
4. Every `.c/.h` begins with:

```c
/**
 * @file ...
 * @author  ...
 * @brief ...
 * @version ...
 * @date ...
 *
 * @copyright Copyright (c) 2026
 */
```

5. Function doc comment uses `/** ... */` and includes `@brief @param @retval`.
6. Inline/block explanation comments use `/* ... */`.
7. Comment language: Chinese; avoid AI-template phrasing.
8. Standard headers use `<>` (e.g. `<stdint.h>`, `<stdio.h>`, `<string.h>`, `<math.h>`, `<stddef.h>`).
9. Project headers use `"..."`.
10. Header files keep include guard and must contain C++ compatibility block:

```c
#ifdef __cplusplus
extern "C" {
#endif

...

#ifdef __cplusplus
}
#endif
```

11. Left brace must be on a new line.
12. Brace indentation must align with control statement indentation.
13. Always use braces for single-line bodies:
- `if`
- `while`
- `else if`
- `else`
- (also keep `for/switch/do` as full blocks)

14. `switch` requires `default`.
15. Each `case` must end with explicit `break/return` (or controlled jump).
16. Prefer explicit comparisons; avoid implicit truthiness when unclear.
17. Integer literals should use explicit suffixes where applicable (`U`, `UL`).
18. Macro parameters/expressions require parentheses.
19. Multi-statement macros require `do { ... } while (0)`.
20. Minimize global visibility: file-private symbols should be `static`.
21. Pointer params should be null-checked before dereference for externally reachable paths.
22. Avoid magic numbers; extract to macro/constant (prefer existing project style).
23. Do not alter timing/ISR/state-machine behavior for formatting-only tasks.
24. Text read/write encoding is mandatory `GB2312` (code page 936), no auto-detect, no fallback encoding.
25. Read path must use strict `GB2312` decoding (decoder exception fallback enabled); if decode fails, stop immediately.
26. Write path must use strict `GB2312` encoding (encoder exception fallback enabled); if encode fails, stop immediately and rollback.
27. Remove obvious AI-style comments (for example: `/* global variable with static scope */`); rewrite as natural engineering comments that describe current module intent/context, not generic coding slogans.
28. Struct-pointer function parameters must use unified name `handle` (for example: `static void gpio_control_work(GPIO_Control_t *handle)`), avoid mixed aliases such as `p_ctrl`/`p_xxx`.
29. Header file internal sections must be ordered as: `#include` -> `#define` -> `typedef` -> function declarations -> variable declarations.
30. Key logic in functions must include clear Chinese comments explaining each critical step and intent; avoid empty/noise comments.
31. Function names must use UpperCamelCase style (for example: `GPIO_Init()`, `UART_SendData()`, `SPI_TransmitReceive()`).
32. Variable names must use lowercase snake_case style (for example: `uart_handle`, `buffer_size`, `is_initialized`).
33. Macro names must use UPPER_SNAKE_CASE style (for example: `UART_BUFFER_SIZE`, `MAX_RETRY_COUNT`, `GPIO_PIN_HIGH`).
34. Struct type names must use lowercase snake_case with `_t` suffix (for example: `uart_config_t`, `gpio_pin_config_t`, `sensor_data_t`).
35. Enum type names must use lowercase snake_case with `_e` suffix (for example: `uart_status_e`, `gpio_mode_e`).
36. When struct-pointer null-check protection uses `NULL` in a file, ensure `#include <stdio.h>` is present at the top include section of the current file.
37. 注释中的关键步骤说明必须直接描述具体作用，不使用`步骤1：`这类编号前缀，且禁止出现`关键步骤：`字样。

---

## Execution Procedure

### Step 2: Pre-check Files in Selected Scope

Enumerate selected files only:

```powershell
Get-ChildItem -Path <selected_scope_paths> -Recurse -File
```

### Step 3: Safe Edit Strategy

- Edit in small passes (headers first, then braces, then comments).
- Avoid broad regex that can touch function comments repeatedly.
- For file header replacement, replace only the first top-of-file block comment.
- During comment cleanup, rewrite AI-template comments into natural project comments; preserve technical meaning and avoid repetitive rule-recitation wording.
- During comment enhancement, add concise Chinese comments for every key function step (state transition, boundary check, hardware access, timing-sensitive branch, error/exit path).
- During comment enhancement, key-step comments must directly describe concrete intent/action (no `步骤1：` prefix) and must not contain `关键步骤：`.
- During signature cleanup, rename struct-pointer parameters to `handle` consistently and update in-function references in the same scope.
- During null-check normalization, if `NULL` is used for struct-pointer protection in a file, insert `#include <stdio.h>` in that file's top include section when missing.
- Run strict encoding preflight before first write:
  - Force `GB2312` decode for source text (code page 936).
  - Decoder fallback must be exception-based; any decode error means immediate stop.
  - Do not switch to UTF-8/UTF-16/other encodings as fallback.
- Required operation: explicit `GB2312` read + explicit `GB2312` write for every touched `.c/.h` file.

### Step 4: Brace Normalization

- Convert control statements to newline brace style.
- Add missing braces to all single-line `if/while/else if/else` bodies.
- Ensure opening brace indent equals control-statement indent.

### Step 5: Header/C++ Guard Normalization

- Keep include guard.
- Ensure exactly one open C++ block and one close C++ block.
- Place close C++ block immediately before final `#endif` of include guard.

### Step 6: Include Normalization

- Convert standard C headers from `"x.h"` to `<x.h>`.
- Keep project headers in `"x.h"`.

### Step 6.5: Header Section Order Normalization

- For every `.h`, normalize internal layout in this order:
  1. `#include`
  2. `#define`
  3. `typedef`
  4. function declarations
  5. variable declarations
- Do not move include guard or C++ compatibility block; only normalize content inside them.
- Keep existing APIs and symbols unchanged while reordering.

### Step 7: Final Newline + Encoding Pass

- Normalize newline to CRLF.
- Trim trailing blank noise.
- Force final `CRLF+CRLF`.
- Write using `GB2312` (code page 936) with strict encoder fallback (exception on unmappable chars).

---

## Verification Checklist

Run and confirm all are clean (within selected scope only):

1. `if/while/else if/else` without brace:
```powershell
rg -n "^\s*(if|while|else\s+if|else)\b" <selected_scope_paths>
```

2. C++ guards exist in headers:
```powershell
rg -n "#ifdef __cplusplus|extern \"C\"|#endif" <selected_scope_paths>
```

3. File header presence:
```powershell
rg -n "^/\*\*|@file|@author|@brief|@version|@date" <selected_scope_paths>
```

4. Tail bytes check (`13,10,13,10` expected):
```powershell
Get-ChildItem <selected_scope_paths> -Recurse -File | ForEach-Object {
  $b=[IO.File]::ReadAllBytes($_.FullName)
  $tail=if($b.Length -ge 4){$b[($b.Length-4)..($b.Length-1)]}else{$b}
  "{0}:{1}" -f $_.FullName,($tail -join ',')
}
```

5. Spot-check random files for Chinese comment readability.
6. Encoding safety check:
```powershell
rg -n "\?\?\?|\uFFFD" <selected_scope_paths>
```
Result must be empty for newly changed content.

7. Header section order check (`.h` only):
```powershell
Get-ChildItem <selected_scope_paths> -Recurse -Filter *.h
```
Manually confirm each header follows: include -> define -> typedef -> function declarations -> variable declarations.

---

## Recovery / Rollback

- If text becomes mojibake or strict `GB2312` decode/encode throws:
  - Stop immediately.
  - Restore from known-good backup.
  - Re-apply rules with explicit strict `GB2312` read/write configuration.
- Never perform non-GB2312 encode/decode cycles.
- Do not continue batch edits after detecting corruption or codec exception.
- If strict GB2312 preflight fails, only allow byte-level operations (copy/backup/line-ending check), no semantic rewrite.

---

## Success Criteria

- All files in selected scope comply with Hard Rules.
- No mojibake/no `?` replacement damage in Chinese comments.
- No unintended behavior changes.
- Diff is style-focused and reviewable.
