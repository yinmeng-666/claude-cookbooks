# Claude Cookbooks

The Claude Cookbooks provide code and guides designed to help developers build with Claude, offering copy-able code snippets that you can easily integrate into your own projects.

## Prerequisites

To make the most of the examples in this cookbook, you'll need a Claude API key (sign up for free [here](https://www.anthropic.com)).

While the code examples are primarily written in Python, the concepts can be adapted to any programming language that supports interaction with the Claude API.

If you're new to working with the Claude API, we recommend starting with our [Claude API Fundamentals course](https://github.com/anthropics/courses/tree/master/anthropic_api_fundamentals) to get a solid foundation.

## Explore Further

Looking for more resources to enhance your experience with Claude and AI assistants? Check out these helpful links:

- [Anthropic developer documentation](https://docs.claude.com/claude/docs/guide-to-anthropics-prompt-engineering-resources)
- [Anthropic support docs](https://support.anthropic.com)
- [Anthropic Discord community](https://www.anthropic.com/discord)

## Contributing

The Claude Cookbooks thrives on the contributions of the developer community. We value your input, whether it's submitting an idea, fixing a typo, adding a new guide, or improving an existing one. By contributing, you help make this resource even more valuable for everyone.

To avoid duplication of efforts, please review the existing issues and pull requests before contributing.

If you have ideas for new examples or guides, share them on the [issues page](https://github.com/anthropics/anthropic-cookbook/issues).

## Table of recipes

### Capabilities
- [Classification](https://github.com/anthropics/anthropic-cookbook/tree/main/capabilities/classification): Explore techniques for text and data classification using Claude.
- [Retrieval Augmented Generation](https://github.com/anthropics/anthropic-cookbook/tree/main/capabilities/retrieval_augmented_generation): Learn how to enhance Claude's responses with external knowledge.
- [Summarization](https://github.com/anthropics/anthropic-cookbook/tree/main/capabilities/summarization): Discover techniques for effective text summarization with Claude.

### Tool Use and Integration
- [Tool use](https://github.com/anthropics/anthropic-cookbook/tree/main/tool_use): Learn how to integrate Claude with external tools and functions to extend its capabilities.
  - [Customer service agent](https://github.com/anthropics/anthropic-cookbook/blob/main/tool_use/customer_service_agent.ipynb)
  - [Calculator integration](https://github.com/anthropics/anthropic-cookbook/blob/main/tool_use/calculator_tool.ipynb)
  - [SQL queries](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/how_to_make_sql_queries.ipynb)

### Third-Party Integrations
- [Retrieval augmented generation](https://github.com/anthropics/anthropic-cookbook/tree/main/third_party): Supplement Claude's knowledge with external data sources.
  - [Vector databases (Pinecone)](https://github.com/anthropics/anthropic-cookbook/blob/main/third_party/Pinecone/rag_using_pinecone.ipynb)
  - [Wikipedia](https://github.com/anthropics/anthropic-cookbook/blob/main/third_party/Wikipedia/wikipedia-search-cookbook.ipynb/)
  - [Web pages](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/read_web_pages_with_haiku.ipynb)
- [Embeddings with Voyage AI](https://github.com/anthropics/anthropic-cookbook/blob/main/third_party/VoyageAI/how_to_create_embeddings.md)

### Multimodal Capabilities
- [Vision with Claude](https://github.com/anthropics/anthropic-cookbook/tree/main/multimodal): 
  - [Getting started with images](https://github.com/anthropics/anthropic-cookbook/blob/main/multimodal/getting_started_with_vision.ipynb)
  - [Best practices for vision](https://github.com/anthropics/anthropic-cookbook/blob/main/multimodal/best_practices_for_vision.ipynb)
  - [Interpreting charts and graphs](https://github.com/anthropics/anthropic-cookbook/blob/main/multimodal/reading_charts_graphs_powerpoints.ipynb)
  - [Extracting content from forms](https://github.com/anthropics/anthropic-cookbook/blob/main/multimodal/how_to_transcribe_text.ipynb)
- [Generate images with Claude](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/illustrated_responses.ipynb): Use Claude with Stable Diffusion for image generation.

### Advanced Techniques
- [Sub-agents](https://github.com/anthropics/anthropic-cookbook/blob/main/multimodal/using_sub_agents.ipynb): Learn how to use Haiku as a sub-agent in combination with Opus.
- [Upload PDFs to Claude](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/pdf_upload_summarization.ipynb): Parse and pass PDFs as text to Claude.
- [Automated evaluations](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/building_evals.ipynb): Use Claude to automate the prompt evaluation process.
- [Enable JSON mode](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/how_to_enable_json_mode.ipynb): Ensure consistent JSON output from Claude.
- [Create a moderation filter](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/building_moderation_filter.ipynb): Use Claude to create a content moderation filter for your application.
- [Prompt caching](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/prompt_caching.ipynb): Learn techniques for efficient prompt caching with Claude.

## Additional Resources

- [Anthropic on AWS](https://github.com/aws-samples/anthropic-on-aws): Explore examples and solutions for using Claude on AWS infrastructure.
- [AWS Samples](https://github.com/aws-samples/): A collection of code samples from AWS which can be adapted for use with Claude. Note that some samples may require modification to work optimally with Claude.

---

## 中文翻译

### Claude Cookbooks

Claude Cookbooks 提供了用于帮助开发者基于 Claude 进行构建的代码与指南，其中包含可直接复制的代码片段，你可以轻松将其集成到自己的项目中。

### 前置条件

为了充分利用本 cookbook 中的示例，你需要一个 Claude API key（可在[这里](https://www.anthropic.com)免费注册）。

虽然代码示例主要使用 Python 编写，但其中的概念也适用于任何支持与 Claude API 交互的编程语言。

如果你刚开始接触 Claude API，我们建议先学习我们的 [Claude API Fundamentals course](https://github.com/anthropics/courses/tree/master/anthropic_api_fundamentals)，以打下扎实基础。

### 进一步探索

想寻找更多资源来提升你使用 Claude 和 AI 助手的体验吗？可以查看以下实用链接：

- [Anthropic 开发者文档](https://docs.claude.com/claude/docs/guide-to-anthropics-prompt-engineering-resources)
- [Anthropic 支持文档](https://support.anthropic.com)
- [Anthropic Discord 社区](https://www.anthropic.com/discord)

### 贡献

Claude Cookbooks 的成长离不开开发者社区的贡献。无论是提交一个想法、修复一个错字、添加新的指南，还是改进现有内容，我们都非常重视你的参与。你的贡献将帮助这个资源对所有人都更有价值。

为避免重复劳动，贡献前请先查看已有的 issues 和 pull requests。

如果你对新的示例或指南有想法，请在 [issues page](https://github.com/anthropics/anthropic-cookbook/issues) 上分享。

### 配方目录

#### 能力

- [分类](https://github.com/anthropics/anthropic-cookbook/tree/main/capabilities/classification)：探索使用 Claude 进行文本和数据分类的技术。
- [检索增强生成](https://github.com/anthropics/anthropic-cookbook/tree/main/capabilities/retrieval_augmented_generation)：学习如何用外部知识增强 Claude 的回答。
- [摘要总结](https://github.com/anthropics/anthropic-cookbook/tree/main/capabilities/summarization)：了解使用 Claude 进行高效文本摘要的技术。

#### 工具使用与集成

- [工具使用](https://github.com/anthropics/anthropic-cookbook/tree/main/tool_use)：学习如何将 Claude 与外部工具和函数集成，以扩展其能力。
  - [客服代理](https://github.com/anthropics/anthropic-cookbook/blob/main/tool_use/customer_service_agent.ipynb)
  - [计算器集成](https://github.com/anthropics/anthropic-cookbook/blob/main/tool_use/calculator_tool.ipynb)
  - [SQL 查询](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/how_to_make_sql_queries.ipynb)

#### 第三方集成

- [检索增强生成](https://github.com/anthropics/anthropic-cookbook/tree/main/third_party)：使用外部数据源补充 Claude 的知识。
  - [向量数据库（Pinecone）](https://github.com/anthropics/anthropic-cookbook/blob/main/third_party/Pinecone/rag_using_pinecone.ipynb)
  - [Wikipedia](https://github.com/anthropics/anthropic-cookbook/blob/main/third_party/Wikipedia/wikipedia-search-cookbook.ipynb/)
  - [网页](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/read_web_pages_with_haiku.ipynb)
- [使用 Voyage AI 生成 Embeddings](https://github.com/anthropics/anthropic-cookbook/blob/main/third_party/VoyageAI/how_to_create_embeddings.md)

#### 多模态能力

- [使用 Claude 处理视觉任务](https://github.com/anthropics/anthropic-cookbook/tree/main/multimodal)：
  - [图像快速入门](https://github.com/anthropics/anthropic-cookbook/blob/main/multimodal/getting_started_with_vision.ipynb)
  - [视觉最佳实践](https://github.com/anthropics/anthropic-cookbook/blob/main/multimodal/best_practices_for_vision.ipynb)
  - [图表与演示文稿解读](https://github.com/anthropics/anthropic-cookbook/blob/main/multimodal/reading_charts_graphs_powerpoints.ipynb)
  - [从表单中提取内容](https://github.com/anthropics/anthropic-cookbook/blob/main/multimodal/how_to_transcribe_text.ipynb)
- [使用 Claude 生成图像](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/illustrated_responses.ipynb)：将 Claude 与 Stable Diffusion 结合用于图像生成。

#### 高级技巧

- [子代理](https://github.com/anthropics/anthropic-cookbook/blob/main/multimodal/using_sub_agents.ipynb)：学习如何将 Haiku 作为子代理，与 Opus 组合使用。
- [将 PDF 上传给 Claude](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/pdf_upload_summarization.ipynb)：将 PDF 解析为文本并传递给 Claude。
- [自动化评估](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/building_evals.ipynb)：使用 Claude 自动化 prompt 评估流程。
- [启用 JSON mode](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/how_to_enable_json_mode.ipynb)：确保 Claude 输出一致的 JSON。
- [创建内容审核过滤器](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/building_moderation_filter.ipynb)：使用 Claude 为你的应用创建内容审核过滤器。
- [Prompt caching](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/prompt_caching.ipynb)：学习如何高效使用 Claude 的 prompt caching 技术。

### 附加资源

- [AWS 上的 Anthropic](https://github.com/aws-samples/anthropic-on-aws)：探索在 AWS 基础设施上使用 Claude 的示例和解决方案。
- [AWS 示例集合](https://github.com/aws-samples/)：AWS 提供的代码示例集合，可调整后用于 Claude。请注意，部分示例可能需要修改后才能更好地适配 Claude。
