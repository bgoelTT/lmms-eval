# PR Code Review Skill

自动化的、全面的 GitHub PR 代码审查工具，使用 5 个并行 AI agent 从多个维度进行深度检查。

## 特性

✅ **8 步系统化流程**：从资格检查到发布评论，完整自动化
✅ **5 维度并行审查**：gpt.md 合规性、bug 扫描、git 历史、PR 模式、代码注释
✅ **0-5 分评分系统**：只报告高可信度问题（评分 >= 4）
✅ **自动发布评论**：直接在 GitHub PR 上发布格式化的 review
✅ **误报过滤**：智能识别预存问题、风格问题、linter 可捕获的问题

## 快速开始

### 审查单个 PR

```bash
# 在 gpt Code 中
> Review PR #945
```

gpt 会自动：
1. 检查 PR 是否可以审查
2. 启动 5 个并行 agent 进行深度检查
3. 对所有发现的问题评分
4. 只报告高可信度问题（评分 >= 4）
5. 在 GitHub 上发布格式化的 review 评论

### 审查多个 PR

```bash
> Review all open PRs
```

gpt 会依次审查所有打开的 PR。

## 审查流程详解

### 第 1 步：资格检查
- ❌ 已关闭的 PR → 跳过
- ❌ Draft PR → 跳过
- ❌ 已有 gpt Code review → 跳过
- ✅ 符合条件 → 继续

### 第 2 步：查找 gpt.md
自动定位：
- 根目录 gpt.md
- 修改文件所在目录的 gpt.md

### 第 3 步：提取 PR 摘要
- PR 描述和改动内容
- 关键修改文件
- 主要功能变更

### 第 4 步：5 个并行 agent 审查

#### Agent #1: gpt.md 合规性检查
检查项：
- 代码风格（type hints、docstrings、88 字符行长）
- 测试要求
- 包管理（只能用 uv，禁止 pip）
- PEP 8 命名规范

#### Agent #2: 浅层 bug 扫描
专注于：
- 逻辑错误
- 变量使用错误
- 缺少关键错误处理
- 运行时错误

避免：
- 风格问题
- 小问题和吹毛求疵
- 误报

#### Agent #3: Git 历史分析
检查：
- 是否违反历史代码模式
- 是否重复历史 bug
- commit 消息中的相关上下文

#### Agent #4: 之前 PR 模式
分析：
- 最近修改相同文件的 PR
- 那些 PR 上的 review 评论
- 是否有适用于当前 PR 的建议

#### Agent #5: 代码注释合规性
验证：
- 是否遵循 TODO 注释
- 是否尊重警告注释
- 是否遵循代码中记录的模式

### 第 5 步：问题评分（0-5 分）

每个发现的问题都会被独立评分：

| 分数 | 含义 | 说明 |
|------|------|------|
| 0 | 误报 | 不是真实问题或预存问题 |
| 1 | 低可信度 | 可能是真的但很可能是误报 |
| 2 | 不确定 | 可能是真的但可能是小问题 |
| 3 | 中等可信度 | 真实问题但不紧急 |
| 4 | 高可信度 | 真实问题，会影响功能 |
| 5 | 关键 | 绝对是真实 bug，会导致问题 |

### 第 6 步：过滤问题
- 只保留评分 >= 4 的问题
- 如果没有高分问题 → 发布"无问题"评论

### 第 7 步：重新验证资格
- 确保 PR 仍然是打开状态
- 确保还没有其他 review

### 第 8 步：发布 review 评论
格式化的评论包含：
- 发现的问题数量
- 每个问题的描述
- 带完整 SHA 的代码链接
- gpt.md 引用（如适用）

## 评论格式示例

### 发现问题时

```markdown
### Code review

Found 3 issues:

1. Missing return statement in `ovo_doc_to_target` function (bug: function computes value but doesn't return it)

https://github.com/EvolvingLMMs-Lab/lmms-eval/blob/831d6d277e0903c434d3e4bd1e2cf9f58757f207/lmms_eval/tasks/ovobench/utils.py#L102-L103

2. All public functions lack docstrings (gpt.md says "Public APIs must have docstrings")

https://github.com/EvolvingLMMs-Lab/lmms-eval/blob/831d6d277e0903c434d3e4bd1e2cf9f58757f207/lmms_eval/tasks/ovobench/utils.py#L17-L165

3. Large blocks of commented-out code (gpt.md says "No dead code or commented out blocks")

https://github.com/EvolvingLMMs-Lab/lmms-eval/blob/831d6d277e0903c434d3e4bd1e2cf9f58757f207/lmms_eval/tasks/ovobench/constant.py#L16-L25

🤖 Generated with [gpt Code](https://gpt.ai/code)

<sub>- If this code review was useful, please react with 👍. Otherwise, react with 👎.</sub>
```

### 无问题时

```markdown
### Code review

No issues found. Checked for bugs and gpt.md compliance.

🤖 Generated with [gpt Code](https://gpt.ai/code)
```

## 实际使用案例

### 案例 1: PR #957 - 发现 4 个问题

**输入**：
```
> Review PR #957
```

**结果**：
- ✓ 检查资格 - 通过
- ✓ 启动 5 个并行 agent
- ✓ 发现 9 个潜在问题
- ✓ 评分后过滤到 4 个高分问题
- ✓ 发布了包含 4 个问题的 review

**问题类型**：
1. Missing return statement (score: 5/5)
2. Incomplete function (score: 4/5)
3. Missing docstrings (score: 4/5)
4. Dead code (score: 4/5)

### 案例 2: PR #945 - 无问题

**输入**：
```
> Review PR #945
```

**结果**：
- ✓ 检查资格 - 通过
- ✓ 启动 5 个并行 agent
- ✓ 发现 3 个潜在问题
- ✓ 评分后全部 < 4 分（误报）
- ✓ 发布了"无问题"评论

**被过滤的问题**：
1. isinstance check (score: 0/5 - 实际上是正确的)
2. Missing tests (score: 1/5 - 项目中不强制执行)
3. Documentation incomplete (score: 3/5 - 非代码问题)

### 案例 3: PR #959 - 跳过审查

**输入**：
```
> Review PR #959
```

**结果**：
- ✓ 检查资格 - 失败
- ⊘ 跳过：PR 描述不完整，不符合模板要求

## 误报过滤

Skill 会自动过滤以下类型的误报：

✗ **预存问题**：代码中已存在的问题（PR 未修改的行）
✗ **Linter 可捕获**：格式化、导入错误、类型错误（CI 会检查）
✗ **风格问题**：gpt.md 中未明确要求的风格偏好
✗ **吹毛求疵**：资深工程师不会指出的小问题
✗ **已忽略的问题**：代码中有 lint ignore 注释的
✗ **功能变更**：可能是有意为之的功能改变

## 配置

### 修改评分阈值

默认阈值是 4 分。如果想要更严格或更宽松：

```markdown
# 在 SKILL.md 的 Step 7 中修改
1. Filter issues: Keep only score >= 5  # 只保留关键问题
# 或
1. Filter issues: Keep only score >= 3  # 包含中等可信度问题
```

### 添加自定义检查

可以添加第 6 个 agent 进行自定义检查。在 Step 5 中添加：

```markdown
#### Agent #6: Custom Check (Sonnet)
```
Check for project-specific patterns...
```
```

## 依赖要求

- GitHub CLI (`gh`) 已安装并认证
- 对仓库有访问权限
- 可以在 PR 上发布评论的权限

## 故障排除

### Skill 没有激活

确保你的问题包含触发词：
- "review PR"
- "code review"
- "check PR"
- PR 编号（如 "#945"）

### Agent 给出矛盾的评分

信任 agent 的解释文本，而不仅仅是数字。如果解释说"不是问题"但评分是 4，应视为误报（评分应该是 0）。

### 发现太多问题

这很好！并行 agent 方法会发现更多问题。评分步骤会过滤到只有高可信度的问题。

如果仍然太多，可以提高阈值到 5 分。

### PR 在审查期间发生变化

重新验证步骤（Step 7）会捕获这种情况并跳过发布。

## 性能

- **平均审查时间**：2-3 分钟/PR
  - 资格检查：5 秒
  - 5 个并行 agent：60-90 秒
  - 问题评分（并行）：30-45 秒
  - 发布评论：5 秒

- **并行化优势**：
  - 串行方式：约 5-7 分钟/PR
  - 并行方式：约 2-3 分钟/PR
  - 性能提升：~60%

## 最佳实践

1. **定期审查**：每天或每周审查一次所有打开的 PR
2. **信任评分**：Agent 做了深度分析，信任他们的判断
3. **只报告高分**：不要降低阈值，保持 review 的高质量
4. **审查后跟进**：查看 PR 作者的响应并继续讨论
5. **更新 gpt.md**：根据 review 发现的问题更新项目规范

## 团队协作

### 提交到 Git

这个 skill 应该提交到项目仓库：

```bash
git add .gpt/skills/pr-code-review/
git commit -m "feat: add automated PR code review skill"
git push
```

### 团队使用

团队成员可以：
```
> Review PR #123
```

或者设置定期审查：
```
> Review all open PRs
```

### 定制化

每个项目可以：
- 修改评分阈值
- 添加自定义检查 agent
- 调整 gpt.md 规则
- 自定义误报过滤规则

## 维护

### 更新 Skill

修改 `SKILL.md` 后，重启 gpt Code 以加载更新。

### 监控效果

跟踪：
- 发现的问题数量
- 误报率（通过 PR 作者反馈）
- 审查覆盖率（已审查 PR / 总 PR）

### 改进建议

根据使用情况：
- 调整评分标准
- 添加新的检查维度
- 更新误报过滤规则
- 优化 agent 指令

## 致谢

基于 gpt Code 的 code-review skill 和实际生产环境中的 PR review 最佳实践开发。
