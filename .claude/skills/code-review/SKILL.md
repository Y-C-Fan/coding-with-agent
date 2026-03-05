---
name: code-review
description: Conduct thorough, constructive code reviews for quality, security, and best practices. Use when reviewing pull requests, checking code quality, identifying bugs, or auditing security.
---

## Review Framework

当进行代码审查时，请遵循以下框架：

### 1. 正确性检查 (Correctness)
- 边界情况和极端输入是否被处理？
- 错误处理是否完善？
- 是否有潜在的空指针/数组越界？
- 断言是否合理？

### 2. 安全检查 (Security)
- SQL 注入：检查是否使用参数化查询
- XSS：检查用户输入是否被正确转义
- 敏感信息：检查是否硬编码了密钥、密码、API tokens
- 文件操作：检查路径遍历漏洞
- 输入验证：检查是否有必要的验证

### 3. 性能分析 (Performance)
- 时间复杂度是否合理？
- 是否有 N+1 查询问题？
- 是否有不必要的重复计算？
- I/O 操作是否可优化？

### 4. 可维护性 (Maintainability)
- 代码是否符合 SOLID 原则？
- 命名是否清晰直观？
- 是否有重复代码可以提取？
- 注释是否充分？

### 5. 测试覆盖 (Testing)
- 核心逻辑是否有测试？
- 边界情况是否被覆盖？
- 测试是否真正验证功能而非假装通过？

## Output Format

请按以下格式输出审查结果：

```
## 审查摘要
[总体评价]

## 🔴 严重问题
[必须修复的问题]

## 🟡 建议改进
[建议但非强制的改进]

## 🟢 优点
[代码中做得好的地方]

## 最终判定
- [ ] 可以合并
- [ ] 需要修改后重审
```
