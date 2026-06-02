# Claude Capabilities

Welcome to the Capabilities section of the Claude Cookbooks! This directory contains a collection of guides that showcase specific capabilities where Claude excels. Each guide provides an in-depth exploration of a particular capability, discussing potential use cases, prompt engineering techniques to optimize results, and approaches for evaluating Claude's performance.

## Guides

- **[Classification with Claude](./classification/guide.ipynb)**: Discover how Claude can revolutionize classification tasks, especially in scenarios with complex business rules and limited training data. This guide walks you through data preparation, prompt engineering with retrieval-augmented generation (RAG), testing, and evaluation.

- **[Retrieval Augmented Generation with Claude](./retrieval_augmented_generation/guide.ipynb)**: Learn how to enhance Claude's capabilities with domain-specific knowledge using RAG. This guide demonstrates how to build a RAG system from scratch, optimize its performance, and create an evaluation suite. You'll learn how techniques like summary indexing and re-ranking can significantly improve precision, recall, and overall accuracy in question-answering tasks.

- **[Retrieval Augmented Generation with Contextual Embeddings](./contextual-embeddings/guide.ipynb)**: Learn how to use a new technique to improve the performance of your RAG system. In traditional RAG, documents are typically split into smaller chunks for efficient retrieval. While this approach works well for many applications, it can lead to problems when individual chunks lack sufficient context. Contextual Embeddings solve this problem by adding relevant context to each chunk before embedding. You'll learn how to use contextual embeddings with semantic search, BM25 search, and reranking to improve performance.

- **[Summarization with Claude](./summarization/guide.ipynb)**: Explore Claude's ability to summarize and synthesize information from multiple sources. This guide covers a variety of summarization techniques, including multi-shot, domain-based, and chunking methods, as well as strategies for handling long-form content and multiple documents. We also explore evaluating summaries, which can be a balance of art, subjectivity, and the right approach!

- **[Text-to-SQL with Claude](./text_to_sql/guide.ipynb)**: This guide covers how to generate complex SQL queries from natural language using prompting techniques, self-improvement, and RAG. We'll also explore how to evaluate and improve the accuracy of generated SQL queries, with evals that test for syntax, data correctness, row count, and more.

- **[Knowledge Graph Construction with Claude](./knowledge_graph/guide.ipynb)**: Build a knowledge graph from unstructured text end-to-end — named entity recognition, relation extraction, entity resolution, and multi-hop querying — using structured outputs for schema-validated extraction and Claude-driven deduplication in place of string-similarity heuristics.

## Getting Started

To get started with these guides, simply navigate to the desired guide's directory and follow the instructions provided in the `guide.ipynb` file. Each guide is self-contained and includes all the necessary code, data, and evaluation scripts to reproduce the examples and experiments.

---

## 中文翻译

### Claude Capabilities

欢迎来到 Claude Cookbooks 的 Capabilities 部分！本目录包含一系列指南，展示了 Claude 擅长的具体能力。每份指南都会深入探讨某一项特定能力，介绍潜在用例、用于优化结果的 prompt engineering 技术，以及评估 Claude 表现的方法。

### 指南

- **[使用 Claude 进行分类](./classification/guide.ipynb)**：了解 Claude 如何革新分类任务，尤其适用于业务规则复杂且训练数据有限的场景。本指南将带你完成数据准备、结合检索增强生成（RAG）的 prompt engineering、测试与评估。
- **[使用 Claude 进行检索增强生成](./retrieval_augmented_generation/guide.ipynb)**：学习如何通过 RAG 使用领域知识增强 Claude 的能力。本指南演示了如何从零开始构建一个 RAG 系统、优化其性能并创建评估套件。你将学习诸如 summary indexing 和 re-ranking 等技术如何显著提升问答任务中的 precision、recall 和整体准确率。
- **[结合 Contextual Embeddings 的检索增强生成](./contextual-embeddings/guide.ipynb)**：学习如何使用一种新技术来提升 RAG 系统的性能。在传统 RAG 中，文档通常会被切分为较小的片段以便高效检索。虽然这种方法在很多应用中效果不错，但当单个片段缺乏足够上下文时，就会出现问题。Contextual Embeddings 通过在 embedding 前为每个片段补充相关上下文来解决这个问题。你将学习如何将 contextual embeddings 与 semantic search、BM25 search 以及 reranking 结合使用以提升性能。
- **[使用 Claude 进行摘要总结](./summarization/guide.ipynb)**：探索 Claude 对多来源信息进行总结与综合的能力。本指南涵盖多种摘要技术，包括 multi-shot、基于领域的方法以及 chunking 方法，同时也讨论如何处理长文本内容和多文档。我们还会探讨如何评估摘要，因为摘要评估往往是在艺术性、主观性与正确方法之间寻找平衡。
- **[使用 Claude 进行 Text-to-SQL](./text_to_sql/guide.ipynb)**：本指南介绍如何使用 prompting techniques、self-improvement 和 RAG 从自然语言生成复杂 SQL 查询。我们还将探讨如何评估并提升生成 SQL 查询的准确率，包括通过 evals 测试语法、数据正确性、返回行数等。
- **[使用 Claude 进行知识图谱构建](./knowledge_graph/guide.ipynb)**：从非结构化文本端到端构建知识图谱——包括 named entity recognition、relation extraction、entity resolution 和 multi-hop querying——使用结构化输出实现 schema-validated extraction，并用 Claude 驱动的去重替代基于字符串相似度的启发式方法。

### 开始使用

要开始使用这些指南，只需进入对应指南的目录，并按照 `guide.ipynb` 文件中的说明进行操作。每份指南都是自包含的，包含复现实例和实验所需的全部代码、数据和评估脚本。
