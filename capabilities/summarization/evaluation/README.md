
# Evaluations with Promptfoo

### A Note on This Evaluation Suite

1) Be sure to follow the instructions below - specifically the pre-requisites about required packages.

2) Running the full eval suite may require higher than normal rate limits. Consider only running a subset of tests in promptfoo.

3) Not every test will pass out of the box - we've designed the evaluation to be moderately challenging.

### Pre-requisities 
To use Promptfoo you will need to have node.js & npm installed on your system. For more information follow [this guide](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)  

You can install promptfoo using npm or run it directly using npx. In this guide we will use npx.  

*Note: For this example you will not need to run `npx promptfoo@latest init` there is already an initialized `promptfooconfig.yaml` file in this directory*  

See the official docs [here](https://www.promptfoo.dev/docs/getting-started)  

#### NOTE - Additional Deps
For this example you will need to install the following dependencies in order for our custom_evals to run properly.

`pip install nltk rouge-score`

### Getting Started

To get started, set your ANTHROPIC_API_KEY environment variable, or other required keys for the providers you selected. You can do `export ANTHROPIC_API_KEY=YOUR_API_KEY`.

Then, `cd` into the `evaluation` directory and write `npx promptfoo@latest eval -c promptfooconfig.yaml --output ../data/results.csv`

Afterwards, you can view the results by running `npx promptfoo@latest view`.

### How it Works

The promptfooconfig.yaml file is the heart of our evaluation setup. It defines several crucial sections:

Prompts:
- Prompts are imported from the prompts.py file.
- These prompts are designed to test various aspects of LM performance.


Providers:
- We configure different Claude versions and their settings here.
- This allows us to test across multiple models or with varying parameters (e.g., different temperature settings).


Tests:
- Test cases are defined either in this file, or in this case imported from tests.yaml.
- These tests specify the inputs and expected outputs for our evaluations.
- Promptfoo offers various built-in test types (see docs), or you can define your own. We have 3 custom evaluations and 1 out of the box (contains method):
    - `bleu_eval.py`: Implements the BLEU (Bilingual Evaluation Understudy) score, which measures the similarity between machine-generated text and reference texts.
    - `rouge_eval.py`: Implements the ROUGE (Recall-Oriented Understudy for Gisting Evaluation) score, which assesses the quality of summarization by comparing it to reference summaries.
    - `llm_eval.py`: Contains custom evaluation metrics that leverage Language Models to assess various aspects of generated text, such as coherence, relevance, or factual accuracy.

Output:
- Specifies the format and location of evaluation results.
- Promptfoo supports various output formats too!

### Overriding the Python binary

By default, promptfoo will run python in your shell. Make sure python points to the appropriate executable.

If a python binary is not present, you will see a "python: command not found" error.

To override the Python binary, set the PROMPTFOO_PYTHON environment variable. You may set it to a path (such as /path/to/python3.11) or just an executable in your PATH (such as python3.11).

---

## 中文翻译

### 使用 Promptfoo 进行评估

### 关于本评估套件的说明

1) 请务必遵循以下说明，尤其是有关所需依赖包的前置条件。

2) 运行完整的 eval 套件可能需要高于平时的 rate limits。你可以考虑只在 promptfoo 中运行部分测试。

3) 并非所有测试开箱即过——我们有意将该评估设计得具有中等挑战性。

### 前置条件

要使用 Promptfoo，你需要在系统中安装 node.js 和 npm。更多信息请参考[本指南](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)。

你可以使用 npm 安装 promptfoo，也可以直接通过 npx 运行它。本指南中我们将使用 npx。

*注意：在本示例中，你无需运行 `npx promptfoo@latest init`，因为此目录中已经存在初始化好的 `promptfooconfig.yaml` 文件。*

官方文档见[这里](https://www.promptfoo.dev/docs/getting-started)。

#### 注意 - 额外依赖

在本示例中，为了让我们的 custom_evals 正常运行，你需要安装以下依赖。

`pip install nltk rouge-score`

### 开始使用

开始之前，请设置你的 ANTHROPIC_API_KEY 环境变量，或设置你所选 provider 所需的其他 key。你可以执行 `export ANTHROPIC_API_KEY=YOUR_API_KEY`。

然后，`cd` 进入 `evaluation` 目录，并运行 `npx promptfoo@latest eval -c promptfooconfig.yaml --output ../data/results.csv`

之后，你可以运行 `npx promptfoo@latest view` 查看结果。

### 工作原理

`promptfooconfig.yaml` 文件是我们评估配置的核心。它定义了几个关键部分：

Prompts:
- Prompts 从 `prompts.py` 文件中导入。
- 这些 prompts 旨在测试 LM performance 的不同方面。

Providers:
- 我们在这里配置不同版本的 Claude 及其设置。
- 这使我们能够在多个模型之间，或在不同参数配置下（例如不同的 temperature 设置）进行测试。

Tests:
- 测试用例可以定义在这个文件中，或者像这里一样从 `tests.yaml` 导入。
- 这些测试规定了评估所使用的输入和期望输出。
- Promptfoo 提供了多种内置测试类型（见文档），你也可以自定义。我们这里有 3 个自定义评估和 1 个开箱即用的测试（contains method）：
    - `bleu_eval.py`：实现 BLEU（Bilingual Evaluation Understudy）分数，用于衡量机器生成文本与参考文本之间的相似度。
    - `rouge_eval.py`：实现 ROUGE（Recall-Oriented Understudy for Gisting Evaluation）分数，通过与参考摘要对比来评估摘要质量。
    - `llm_eval.py`：包含利用 Language Models 评估生成文本多个方面的自定义指标，例如连贯性、相关性或事实准确性。

Output:
- 指定评估结果的格式和位置。
- Promptfoo 也支持多种输出格式！

### 覆盖 Python binary

默认情况下，promptfoo 会运行你 shell 中的 python。请确保 python 指向合适的可执行文件。

如果系统中不存在 python binary，你会看到 `"python: command not found"` 错误。

若要覆盖 Python binary，请设置 `PROMPTFOO_PYTHON` 环境变量。你可以将其设为一个路径（例如 /path/to/python3.11），也可以直接设为 PATH 中的某个可执行文件（例如 python3.11）。
