# Evaluations with Promptfoo



### Pre-requisities 
To use Promptfoo you will need to have node.js & npm installed on your system. For more information follow [this guide](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)  

You can install promptfoo using npm or run it directly using npx. In this guide we will use npx.  

*Note: For this example you will not need to run `npx promptfoo@latest init` there is already an initialized `promptfooconfig.yaml` file in this directory*  

See the official docs [here](https://www.promptfoo.dev/docs/getting-started)  



### Getting Started
The evaluation is orchestrated by the `promptfooconfig.yaml` file. In this file we define the following sections:

- Prompts
    - Promptfoo enables you to import prompts in many different formats. You can read more about this [here](https://www.promptfoo.dev/docs/configuration/parameters).
    - In this example we will load 3 prompts - the same used in `guide.ipynb` from the `prompts.py` file:
        - The functions are identical to those used in `guide.ipynb` except that instead of calling the Claude API they just return the prompt. Promptfoo then handles the orchestration of calling the API and storing the results.
        - You can read more about prompt functions [here](https://www.promptfoo.dev/docs/configuration/parameters#prompt-functions). Using python allows us to reuse the VectorDB class which is necessary for RAG, this is defined in `vectordb.py`.
- Providers
    - With Promptfoo you can connect to many different LLMs from different platforms, see [here for more](https://www.promptfoo.dev/docs/providers). In `guide.ipynb` we used Haiku with default temperature 0.0. We will use Promptfoo to experiment with an array of different temperature settings to identify the optimal choice for our use case.
- Tests
    - We will use the same data that was used in `guide.ipynb` which can be found in [`dataset.csv`](./dataset.csv).
    - Promptfoo has a wide array of built in tests which can be found [here](https://www.promptfoo.dev/docs/configuration/expected-outputs/deterministic).
    - In this example we will define a test in our `dataset.csv` as the conditions of our evaluation change with each row and a test in the `promptfooconfig.yaml` for conditions that are consistent across all test cases. Read more about this [here](https://www.promptfoo.dev/docs/configuration/parameters/#import-from-csv)
- Transform
    - In the `defaultTest` section we define a transform function. This is a python function which extracts the specific output we want to test from the LLM response. 
- Output
    - We define the path for the output file. Promptfoo can output results in many formats, [see here](https://www.promptfoo.dev/docs/configuration/parameters/#output-file). Alternatively you can use Promptfoo's web UI, [see here](https://www.promptfoo.dev/docs/usage/web-ui).


### Run the eval

To get started with Promptfoo open your terminal and navigate to this directory (`./evaluation`).

Before running your evaluation you must define the following environment variables:

`export ANTHROPIC_API_KEY=YOUR_API_KEY`  
`export VOYAGE_API_KEY=YOUR_API_KEY`

From the `evaluation` directory, run the following command.  

`npx promptfoo@latest eval`

If you would like to increase the concurrency of the requests (default = 4), run the following command.  

`npx promptfoo@latest eval -j 25`  

When the evaluation is complete the terminal will print the results for each row in the dataset.

You can now go back to `guide.ipynb` to analyze the results!



---

## 中文翻译

### 使用 Promptfoo 进行评估

### 前置条件

要使用 Promptfoo，你需要在系统中安装 node.js 和 npm。更多信息请参考[本指南](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)。

你可以使用 npm 安装 promptfoo，也可以直接通过 npx 运行它。本指南中我们将使用 npx。

*注意：在本示例中，你无需运行 `npx promptfoo@latest init`，因为此目录中已经存在初始化好的 `promptfooconfig.yaml` 文件。*

官方文档见[这里](https://www.promptfoo.dev/docs/getting-started)。

### 开始使用

评估由 `promptfooconfig.yaml` 文件统一编排。在这个文件中，我们定义了以下几个部分：

- Prompts
    - Promptfoo 支持导入多种不同格式的 prompt。你可以在[这里](https://www.promptfoo.dev/docs/configuration/parameters)了解更多。
    - 在本示例中，我们会加载 3 个 prompt，它们与 `guide.ipynb` 中 `prompts.py` 文件使用的是同一组：
        - 这些函数与 `guide.ipynb` 中使用的函数完全一致，不同之处在于它们不会调用 Claude API，而只是返回 prompt。随后由 Promptfoo 负责调用 API 并存储结果的编排。
        - 你可以在[这里](https://www.promptfoo.dev/docs/configuration/parameters#prompt-functions)进一步了解 prompt functions。使用 python 还能让我们复用 `vectordb.py` 中定义的、RAG 所需的 VectorDB class。
- Providers
    - 通过 Promptfoo，你可以连接来自不同平台的多种 LLM，详见[这里](https://www.promptfoo.dev/docs/providers)。在 `guide.ipynb` 中，我们使用的是默认 temperature 为 0.0 的 Haiku。这里我们将使用 Promptfoo 尝试一系列不同的 temperature 设置，以找出最适合我们用例的选择。
- Tests
    - 我们将使用与 `guide.ipynb` 中相同的数据，这些数据位于 [`dataset.csv`](./dataset.csv)。
    - Promptfoo 提供了丰富的内置测试类型，可在[这里](https://www.promptfoo.dev/docs/configuration/expected-outputs/deterministic)查看。
    - 在本示例中，由于评估条件会随 `dataset.csv` 中的每一行变化，我们会在 `dataset.csv` 中定义一部分测试；而对于所有测试用例都一致的条件，我们会在 `promptfooconfig.yaml` 中定义测试。更多说明请参见[这里](https://www.promptfoo.dev/docs/configuration/parameters/#import-from-csv)。
- Transform
    - 在 `defaultTest` 部分中，我们定义了一个 transform function。它是一个 python function，用于从 LLM 响应中提取我们想要测试的特定输出。
- Output
    - 我们定义了输出文件的路径。Promptfoo 支持将结果输出为多种格式，详见[这里](https://www.promptfoo.dev/docs/configuration/parameters/#output-file)。另外你也可以使用 Promptfoo 的 web UI，详见[这里](https://www.promptfoo.dev/docs/usage/web-ui)。

### 运行评估

要开始使用 Promptfoo，请打开终端并进入此目录（`./evaluation`）。

在运行评估之前，你必须定义以下环境变量：

`export ANTHROPIC_API_KEY=YOUR_API_KEY`
`export VOYAGE_API_KEY=YOUR_API_KEY`

在 `evaluation` 目录下，运行以下命令。

`npx promptfoo@latest eval`

如果你希望提高请求并发数（默认值 = 4），请运行以下命令。

`npx promptfoo@latest eval -j 25`

评估完成后，终端会输出数据集中每一行的结果。

现在你可以返回 `guide.ipynb` 分析结果了！
