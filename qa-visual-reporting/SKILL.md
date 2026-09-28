---
name: qa-visual-reporting
description: "自动化生成自包含的 HTML 交互式测试报告（Allure 或 pytest-html 降级），并通过 multica attachment upload 交付可视化质检卡片。"
disable-model-invocation: true
---

# QA Visual Reporting & Attachment Delivery

## 核心目标
本技能用于在测试与审查环节，将黑屏文本测试日志转化为**单文件交互式 HTML 质检报告**，并通过 Multica 附件系统交付给人类审查者。

## 执行流程

### 1. 运行测试并导出单文件报告
优先检查环境是否有 `allure` CLI；若无则自动降级为 `pytest-html`（`--self-contained-html`）：

```bash
python -c "
import shutil, subprocess
if shutil.which('allure'):
    subprocess.run(['pytest', '--alluredir=./allure-results', '-q'], check=False)
    subprocess.run(['allure', 'generate', '--single-file', './allure-results', '-o', './report', '--clean'], check=True)
    shutil.copy('./report/index.html', './test-report.html')
else:
    subprocess.run(['pytest', '--html=./test-report.html', '--self-contained-html', '-q'], check=False)
"
```

### 2. 校验与上传报告
1. 检查 `./test-report.html` 文件大小必须大于 0；
2. 执行上传命令：
   ```bash
   multica attachment upload ./test-report.html
   ```
3. 捕获命令返回的 Markdown 附件卡片语法。

### 3. 交付规范
在终审评论或提审总结中：
- 提炼关键指标看板（Passed / Failed / Skipped / 耗时）；
- 附上上传后的附件卡片 Markdown；
- 若存在跳过或失败用例，列出关键阻断原因。
