# Claude Skills Cookbook 🚀

A comprehensive guide to using Claude's Skills feature for document generation, data analysis, and business automation. This cookbook demonstrates how to leverage Claude's built-in skills for Excel, PowerPoint, and PDF creation, as well as how to build custom skills for specialized workflows.

> **🎯 See Skills in Action:** Check out **[Claude Creates Files](https://www.anthropic.com/news/create-files)** to see how these Skills power Claude's ability to create and edit documents directly in Claude.ai and the desktop app!

## What are Skills?

Skills are organized packages of instructions, executable code, and resources that give Claude specialized capabilities for specific tasks. Think of them as "expertise packages" that Claude can discover and load dynamically to:

- Create professional documents (Excel, PowerPoint, PDF, Word)
- Perform complex data analysis and visualization
- Apply company-specific workflows and branding
- Automate business processes with domain expertise

📖 Read our engineering blog post on [Equipping agents for the real world with Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

## Key Features

- ✨ **Progressive Disclosure Architecture** - Skills load only when needed, optimizing token usage
- 📊 **Financial Focus** - Real-world examples for finance and business analytics
- 🔧 **Custom Skills Development** - Learn to build and deploy your own skills
- 🎯 **Production-Ready Examples** - Code you can adapt for immediate use

## Cookbook Structure

### 📚 [Notebook 1: Introduction to Skills](notebooks/01_skills_introduction.ipynb)

Learn the fundamentals of Claude's Skills feature with quick-start examples.

- Understanding Skills architecture
- Setting up the API with beta headers
- Creating your first Excel spreadsheet
- Generating PowerPoint presentations
- Exporting to PDF format

### 💼 [Notebook 2: Financial Applications](notebooks/02_skills_financial_applications.ipynb)

Explore powerful business use cases with real financial data.

- Building financial dashboards with charts and pivot tables
- Portfolio analysis and investment reporting
- Cross-format workflows: CSV → Excel → PowerPoint → PDF
- Token optimization strategies

### 🔧 [Notebook 3: Custom Skills Development](notebooks/03_skills_custom_development.ipynb)

Master the art of creating your own specialized skills.

- Building a financial ratio calculator
- Creating company brand guidelines skill
- Advanced: Financial modeling suite
- [Best practices](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/best-practices) and security considerations

## Quick Start

### Prerequisites

- Python 3.8 or higher
- Anthropic API key ([get one here](https://console.anthropic.com/))
- Jupyter Notebook or JupyterLab

### Installation

1. **Clone the repository**

```bash
git clone https://github.com/anthropics/claude-cookbooks.git
cd claude-cookbooks/skills
```

2. **Create virtual environment** (recommended)

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. **Install dependencies**

```bash
pip install -r requirements.txt
```

4. **Configure API key**

```bash
cp .env.example .env
# Edit .env and add your ANTHROPIC_API_KEY
```

5. **Launch Jupyter**

```bash
jupyter notebook
```

6. **Start with Notebook 1**
   Open `notebooks/01_skills_introduction.ipynb` and follow along!

## Sample Data

The cookbook includes realistic financial datasets in `sample_data/`:

- 📊 **financial_statements.csv** - Quarterly P&L, balance sheet, and cash flow data
- 💰 **portfolio_holdings.json** - Investment portfolio with performance metrics
- 📋 **budget_template.csv** - Department budget with variance analysis
- 📈 **quarterly_metrics.json** - KPIs and operational metrics

## Project Structure

```
skills/
├── notebooks/                    # Jupyter notebooks
│   ├── 01_skills_introduction.ipynb
│   ├── 02_skills_financial_applications.ipynb
│   └── 03_skills_custom_development.ipynb
├── sample_data/                  # Financial datasets
│   ├── financial_statements.csv
│   ├── portfolio_holdings.json
│   ├── budget_template.csv
│   └── quarterly_metrics.json
├── custom_skills/                # Your custom skills
│   ├── financial_analyzer/
│   ├── brand_guidelines/
│   └── report_generator/
├── outputs/                      # Generated files
├── docs/                         # Documentation
├── requirements.txt             # Python dependencies
├── .env.example                 # Environment template
└── README.md                    # This file
```

## API Configuration

Skills require specific beta headers. The notebooks handle this automatically, but here's what's happening behind the scenes:

```python
from anthropic import Anthropic

client = Anthropic(
    api_key="your-api-key",
    default_headers={
        "anthropic-beta": "code-execution-2025-08-25,files-api-2025-04-14,skills-2025-10-02"
    }
)
```

**Required Beta Headers:**

- `code-execution-2025-08-25` - Enables code execution for Skills
- `files-api-2025-04-14` - Required for downloading generated files
- `skills-2025-10-02` - Enables Skills feature

## Working with Generated Files

When Skills create documents (Excel, PowerPoint, PDF, etc.), they return `file_id` attributes in the response. You must use the **Files API** to download these files.

### How It Works

1. **Skills create files** during code execution
2. **Response includes file_ids** for each created file
3. **Use Files API** to download the actual file content
4. **Save locally** or process as needed

### Example: Creating and Downloading an Excel File

```python
from anthropic import Anthropic

client = Anthropic(api_key="your-api-key")

# Step 1: Use a skill to create a file
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=4096,
    container={
        "skills": [
            {"type": "anthropic", "skill_id": "xlsx", "version": "latest"}
        ]
    },
    tools=[{"type": "code_execution_20250825", "name": "code_execution"}],
    messages=[{
        "role": "user",
        "content": "Create an Excel file with a simple budget spreadsheet"
    }]
)

# Step 2: Extract file_id from the response
file_id = None
for block in response.content:
    if block.type == "tool_result" and hasattr(block, 'output'):
        # Look for file_id in the tool output
        if 'file_id' in str(block.output):
            file_id = extract_file_id(block.output)  # Parse the file_id
            break

# Step 3: Download the file using Files API
if file_id:
    file_content = client.beta.files.download(file_id=file_id)

    # Step 4: Save to disk
    with open("outputs/budget.xlsx", "wb") as f:
        f.write(file_content.read())

    print(f"✅ File downloaded: budget.xlsx")
```

### Files API Methods

```python
# Download file content (binary)
content = client.beta.files.download(file_id="file_abc123...")
with open("output.xlsx", "wb") as f:
    f.write(content.read())  # Use .read() not .content

# Get file metadata
info = client.beta.files.retrieve_metadata(file_id="file_abc123...")
print(f"Filename: {info.filename}, Size: {info.size_bytes} bytes")  # Use size_bytes not size

# List all files
files = client.beta.files.list()
for file in files.data:
    print(f"{file.filename} - {file.created_at}")

# Delete a file
client.beta.files.delete(file_id="file_abc123...")
```

**Important Notes:**

- Files are stored temporarily on Anthropic's servers
- Downloaded files should be saved to your local `outputs/` directory
- The Files API uses the same API key as the Messages API
- All notebooks include helper functions for file download
- **Files are overwritten by default** - rerunning cells will replace existing files (you'll see `[overwritten]` in the output)

See the [Files API documentation](https://docs.claude.com/en/api/files-content) for complete details.

## Built-in Skills Reference

Claude comes with these pre-built skills:

| Skill      | ID     | Description                                                                 |
| ---------- | ------ | --------------------------------------------------------------------------- |
| Excel      | `xlsx` | Create and manipulate Excel workbooks with formulas, charts, and formatting |
| PowerPoint | `pptx` | Generate professional presentations with slides, charts, and transitions    |
| PDF        | `pdf`  | Create formatted PDF documents with text, tables, and images                |
| Word       | `docx` | Generate Word documents with rich formatting and structure                  |

## Creating Custom Skills

Custom skills follow this structure:

```
my_skill/
├── SKILL.md           # Required: Instructions for Claude
├── scripts/           # Optional: Python/JS code
│   └── processor.py
└── resources/         # Optional: Templates, data
    └── template.xlsx
```

Learn more in [Notebook 3](notebooks/03_skills_custom_development.ipynb).

## Common Use Cases

### Financial Reporting

- Automated quarterly reports
- Budget variance analysis
- Investment performance dashboards

### Data Analysis

- Excel-based analytics with complex formulas
- Pivot table generation
- Statistical analysis and visualization

### Document Automation

- Branded presentation generation
- Report compilation from multiple sources
- Cross-format document conversion

## Performance Tips

1. **Use Progressive Disclosure**: Skills load in stages to minimize token usage
2. **Batch Operations**: Process multiple files in a single conversation
3. **Skill Composition**: Combine multiple skills for complex workflows
4. **Cache Reuse**: Use container IDs to reuse loaded skills

## Troubleshooting

### Common Issues

**API Key Not Found**

```
ValueError: ANTHROPIC_API_KEY not found
```

→ Make sure you've copied `.env.example` to `.env` and added your key

**Skills Beta Header Missing**

```
Error: Skills feature requires beta header
```

→ Ensure you're using the correct beta headers as shown in the notebooks

**Token Limit Exceeded**

```
Error: Request exceeds token limit
```

→ Break large operations into smaller chunks or use progressive disclosure

## Resources

### Documentation

- 📖 [Claude API Documentation](https://docs.anthropic.com/en/api/messages)
- 🔧 [Skills Documentation](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview)

### Support Articles

- 📚 [Teach Claude your way of working using Skills](https://support.claude.com/en/articles/12580051-teach-claude-your-way-of-working-using-skills) - User guide for working with Skills
- 🛠️ [How to create a skill with Claude through conversation](https://support.claude.com/en/articles/12599426-how-to-create-a-skill-with-claude-through-conversation) - Interactive skill creation guide

### Community & Support

- 💬 [Claude Support](https://support.claude.com)
- 🐙 [GitHub Issues](https://github.com/anthropics/claude-cookbooks/issues)

## Contributing

We welcome contributions! Please see [CONTRIBUTING.md](../CONTRIBUTING.md) for guidelines.

## License

This cookbook is provided under the MIT License. See [LICENSE](../LICENSE) for details.

## Acknowledgments

Special thanks to the Anthropic team for developing the Skills feature and providing the SDK.

---

**Questions?** Check the [FAQ](docs/FAQ.md) or open an issue.

**Ready to start?** Open [Notebook 1](notebooks/01_skills_introduction.ipynb) and let's build something amazing! 🎉

---

## 中文翻译

# Claude Skills Cookbook

一份关于如何使用 Claude Skills 功能进行文档生成、数据分析和业务自动化的综合指南。本 cookbook 展示了如何利用 Claude 内置的 Excel、PowerPoint 和 PDF 创建技能，以及如何为特定工作流构建自定义技能。

&gt; **查看 Skills 的实际效果：** 请查看 **[Claude 创建文件](https://www.anthropic.com/news/create-files)**，了解这些 Skills 如何支撑 Claude 在 Claude.ai 和桌面应用中直接创建和编辑文档的能力。

## 什么是 Skills？

Skills 是由指令、可执行代码和资源组成的有组织的软件包，能够为 Claude 提供面向特定任务的专门能力。你可以把它们理解为 Claude 可动态发现并加载的“专业能力包”，以便：

- 创建专业文档（Excel、PowerPoint、PDF、Word）
- 执行复杂的数据分析与可视化
- 应用企业特定的工作流与品牌规范
- 利用领域专业知识自动化业务流程

阅读我们的工程博客文章：[使用 Skills 为 agent 装备现实世界能力](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

## 核心特性

- **渐进式披露架构** - Skills 仅在需要时加载，以优化 token 使用
- **金融聚焦** - 面向金融和商业分析的真实世界示例
- **自定义 Skills 开发** - 学习如何构建和部署你自己的 skills
- **可用于生产的示例** - 可直接改造并投入使用的代码

## Cookbook 结构

### [Notebook 1：Skills 简介](notebooks/01_skills_introduction.ipynb)

通过快速入门示例学习 Claude Skills 功能的基础知识。

- 理解 Skills 架构
- 使用 beta headers 设置 API
- 创建你的第一个 Excel 电子表格
- 生成 PowerPoint 演示文稿
- 导出为 PDF 格式

### [Notebook 2：金融应用](notebooks/02_skills_financial_applications.ipynb)

结合真实金融数据，探索强大的业务用例。

- 使用图表和数据透视表构建财务仪表板
- 投资组合分析与投资报告
- 跨格式工作流：CSV → Excel → PowerPoint → PDF
- Token 优化策略

### [Notebook 3：自定义 Skills 开发](notebooks/03_skills_custom_development.ipynb)

掌握创建你自己的专用 skills 的方法。

- 构建财务比率计算器
- 创建公司品牌规范 skill
- 进阶：金融建模套件
- [最佳实践](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/best-practices) 与安全注意事项

## 快速开始

### 前置要求

- Python 3.8 或更高版本
- Anthropic API key（[在此获取](https://console.anthropic.com/)）
- Jupyter Notebook 或 JupyterLab

### 安装

1. **克隆仓库**

```bash
git clone https://github.com/anthropics/claude-cookbooks.git
cd claude-cookbooks/skills
```

2. **创建虚拟环境**（推荐）

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. **安装依赖**

```bash
pip install -r requirements.txt
```

4. **配置 API key**

```bash
cp .env.example .env
# Edit .env and add your ANTHROPIC_API_KEY
```

5. **启动 Jupyter**

```bash
jupyter notebook
```

6. **从 Notebook 1 开始**
   打开 `notebooks/01_skills_introduction.ipynb` 并跟着操作！

## 示例数据

本 cookbook 在 `sample_data/` 中包含了逼真的金融数据集：

- **financial_statements.csv** - 季度损益表、资产负债表和现金流数据
- **portfolio_holdings.json** - 带有绩效指标的投资组合
- **budget_template.csv** - 带有差异分析的部门预算
- **quarterly_metrics.json** - KPI 和运营指标

## 项目结构

```
skills/
├── notebooks/                    # Jupyter notebooks
│   ├── 01_skills_introduction.ipynb
│   ├── 02_skills_financial_applications.ipynb
│   └── 03_skills_custom_development.ipynb
├── sample_data/                  # Financial datasets
│   ├── financial_statements.csv
│   ├── portfolio_holdings.json
│   ├── budget_template.csv
│   └── quarterly_metrics.json
├── custom_skills/                # Your custom skills
│   ├── financial_analyzer/
│   ├── brand_guidelines/
│   └── report_generator/
├── outputs/                      # Generated files
├── docs/                         # Documentation
├── requirements.txt             # Python dependencies
├── .env.example                 # Environment template
└── README.md                    # This file
```

## API 配置

Skills 需要特定的 beta headers。notebooks 会自动处理这些内容，但这里说明一下幕后实际发生了什么：

```python
from anthropic import Anthropic

client = Anthropic(
    api_key="your-api-key",
    default_headers={
        "anthropic-beta": "code-execution-2025-08-25,files-api-2025-04-14,skills-2025-10-02"
    }
)
```

**必需的 Beta Headers：**

- `code-execution-2025-08-25` - 为 Skills 启用代码执行
- `files-api-2025-04-14` - 下载生成文件所必需
- `skills-2025-10-02` - 启用 Skills 功能

## 处理生成的文件

当 Skills 创建文档（Excel、PowerPoint、PDF 等）时，它们会在响应中返回 `file_id` 属性。你必须使用 **Files API** 下载这些文件。

### 工作原理

1. **Skills 在代码执行期间创建文件**
2. **响应中包含 file_ids**，对应每个已创建文件
3. **使用 Files API** 下载实际文件内容
4. **保存到本地** 或按需进一步处理

### 示例：创建并下载一个 Excel 文件

```python
from anthropic import Anthropic

client = Anthropic(api_key="your-api-key")

# Step 1: Use a skill to create a file
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=4096,
    container={
        "skills": [
            {"type": "anthropic", "skill_id": "xlsx", "version": "latest"}
        ]
    },
    tools=[{"type": "code_execution_20250825", "name": "code_execution"}],
    messages=[{
        "role": "user",
        "content": "Create an Excel file with a simple budget spreadsheet"
    }]
)

# Step 2: Extract file_id from the response
file_id = None
for block in response.content:
    if block.type == "tool_result" and hasattr(block, 'output'):
        # Look for file_id in the tool output
        if 'file_id' in str(block.output):
            file_id = extract_file_id(block.output)  # Parse the file_id
            break

# Step 3: Download the file using Files API
if file_id:
    file_content = client.beta.files.download(file_id=file_id)

    # Step 4: Save to disk
    with open("outputs/budget.xlsx", "wb") as f:
        f.write(file_content.read())

    print(f"✅ File downloaded: budget.xlsx")
```

### Files API 方法

```python
# Download file content (binary)
content = client.beta.files.download(file_id="file_abc123...")
with open("output.xlsx", "wb") as f:
    f.write(content.read())  # Use .read() not .content

# Get file metadata
info = client.beta.files.retrieve_metadata(file_id="file_abc123...")
print(f"Filename: {info.filename}, Size: {info.size_bytes} bytes")  # Use size_bytes not size

# List all files
files = client.beta.files.list()
for file in files.data:
    print(f"{file.filename} - {file.created_at}")

# Delete a file
client.beta.files.delete(file_id="file_abc123...")
```

**重要说明：**

- 文件会临时存储在 Anthropic 的服务器上
- 下载后的文件应保存到本地 `outputs/` 目录
- Files API 与 Messages API 使用相同的 API key
- 所有 notebooks 都包含用于下载文件的辅助函数
- **默认会覆盖文件** - 重新运行单元格会替换已有文件（你会在输出中看到 `[overwritten]`）

完整细节请参见 [Files API 文档](https://docs.claude.com/en/api/files-content)。

## 内置 Skills 参考

Claude 自带以下预构建 skills：

| Skill      | ID     | 描述 |
| ---------- | ------ | --------------------------------------------------------------------------- |
| Excel      | `xlsx` | 创建和操作带有公式、图表和格式设置的 Excel 工作簿 |
| PowerPoint | `pptx` | 生成带有幻灯片、图表和过渡效果的专业演示文稿 |
| PDF        | `pdf`  | 创建包含文本、表格和图片的格式化 PDF 文档 |
| Word       | `docx` | 生成具有丰富格式和结构的 Word 文档 |

## 创建自定义 Skills

自定义 skills 遵循以下结构：

```
my_skill/
├── SKILL.md           # Required: Instructions for Claude
├── scripts/           # Optional: Python/JS code
│   └── processor.py
└── resources/         # Optional: Templates, data
    └── template.xlsx
```

更多内容请参见 [Notebook 3](notebooks/03_skills_custom_development.ipynb)。

## 常见用例

### 财务报告

- 自动化季度报告
- 预算差异分析
- 投资绩效仪表板

### 数据分析

- 基于 Excel 的复杂公式分析
- 数据透视表生成
- 统计分析与可视化

### 文档自动化

- 品牌化演示文稿生成
- 汇总多个来源生成报告
- 跨格式文档转换

## 性能建议

1. **使用渐进式披露**：Skills 分阶段加载，以尽量减少 token 使用
2. **批量操作**：在一次对话中处理多个文件
3. **Skill 组合**：为复杂工作流组合多个 skills
4. **缓存复用**：使用 container ID 复用已加载的 skills

## 故障排查

### 常见问题

**未找到 API Key**

```
ValueError: ANTHROPIC_API_KEY not found
```

→ 请确认你已经将 `.env.example` 复制为 `.env` 并添加了自己的 key

**缺少 Skills Beta Header**

```
Error: Skills feature requires beta header
```

→ 请确保你使用了 notebooks 中展示的正确 beta headers

**超出 Token 限制**

```
Error: Request exceeds token limit
```

→ 将大型操作拆分为更小的部分，或使用渐进式披露

## 资源

### 文档

- [Claude API 文档](https://docs.anthropic.com/en/api/messages)
- [Skills 文档](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview)

### 支持文章

- [使用 Skills 让 Claude 学会你的工作方式](https://support.claude.com/en/articles/12580051-teach-claude-your-way-of-working-using-skills) - 使用 Skills 的用户指南
- [如何通过对话使用 Claude 创建 skill](https://support.claude.com/en/articles/12599426-how-to-create-a-skill-with-claude-through-conversation) - 交互式 skill 创建指南

### 社区与支持

- [Claude 支持](https://support.claude.com)
- [GitHub Issues](https://github.com/anthropics/claude-cookbooks/issues)

## 贡献

欢迎贡献！请参阅 [CONTRIBUTING.md](../CONTRIBUTING.md) 了解指南。

## 许可证

本 cookbook 以 MIT License 提供。详情请参见 [LICENSE](../LICENSE)。

## 致谢

特别感谢 Anthropic 团队开发 Skills 功能并提供 SDK。

---

**有问题？** 请查看 [FAQ](docs/FAQ.md) 或提交 issue。

**准备开始了吗？** 打开 [Notebook 1](notebooks/01_skills_introduction.ipynb)，让我们一起构建一些很棒的东西。
