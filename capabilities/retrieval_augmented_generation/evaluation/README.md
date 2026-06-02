# Evaluations with Promptfoo

### Pre-requisities 
To use Promptfoo you will need to have node.js & npm installed on your system. For more information follow [this guide](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)  

You can install promptfoo using npm or run it directly using npx. In this guide we will use npx.  

*Note: For this example you will not need to run `npx promptfoo@latest init` there is already an initialized `promptfooconfig.yaml` file in this directory*  

See the official docs [here](https://www.promptfoo.dev/docs/getting-started)  


### Getting Started
The evaluation is orchestrated by the `promptfooconfig...` `.yaml` files. In our application we divide the evaluation logic between `promptfooconfig_retrieval.yaml` for evaluating the retrieval system and `promptfooconfig_end_to_end.yaml` to evaluate the end to end performance. In each of these files we define the following sections

### Retrieval Evaluations

- Prompts
    - Promptfoo enables you to import prompts in many different formats. You can read more about this [here](https://www.promptfoo.dev/docs/configuration/parameters).
    - In our case, we skip providing a new prompt each time, and merely pass through the `{{query}}` to each retrieval 'provider' for evaluation
- Providers
    - Instead of using a standard LLM provider, we wrote custom providers for each retrieval method found in `guide.ipynb`
- Tests
    - We will use the same data that was used in `guide.ipynb`. We split it into `end_to_end_dataset.csv` and `retrieval_dataset.csv` and added an `__expected` column to each dataset which allows us to automatically run assertions for each row
    - You can find our retrieval evaluation logic in `eval_end_to_end.py`

### End to End Evaluations

- Prompts
    - Promptfoo enables you to import prompts in many different formats. You can read more about this [here](https://www.promptfoo.dev/docs/configuration/parameters).
    - We have 3 prompts in our end to end evaluation config: each of which corresponds to a method use
        - The functions are identical to those used in `guide.ipynb` except that instead of calling the Claude API they just return the prompt. Promptfoo then handles the orchestration of calling the API and storing the results.
        - You can read more about prompt functions [here](https://www.promptfoo.dev/docs/configuration/parameters#prompt-functions). Using python allows us to reuse the VectorDB class which is necessary for RAG, this is defined in `vectordb.py`.
- Providers
    - With Promptfoo you can connect to many different LLMs from different platforms, see [here for more](https://www.promptfoo.dev/docs/providers). In `guide.ipynb` we used Haiku with default temperature 0.0. We will use Promptfoo to experiment with different models.
- Tests
    - We will use the same data that was used in `guide.ipynb`. We split it into `end_to_end_dataset.csv` and `retrieval_dataset.csv` and added an `__expected` column to each dataset which allows us to automatically run assertions for each row
    - Promptfoo has a wide array of built in tests which can be found [here](https://www.promptfoo.dev/docs/configuration/expected-outputs/deterministic).
    - You can find the test logic for the retrieval system in `eval_retrieval.py` and the test logic for the end to end system in `eval_end_to_end.py`
- Output
    - We define the path for the output file. Promptfoo can output results in many formats, [see here](https://www.promptfoo.dev/docs/configuration/parameters/#output-file). Alternatively you can use Promptfoo's web UI, [see here](https://www.promptfoo.dev/docs/usage/web-ui).


### Run the eval

To get started with Promptfoo open your terminal and navigate to this directory (`./evaluation`).

Before running your evaluation you must define the following enviroment variables:

`export ANTHROPIC_API_KEY=YOUR_API_KEY`  
`export VOYAGE_API_KEY=YOUR_API_KEY`

From the `evaluation` directory, run one of the following commands.  

- To evaluate the end to end system performance: `npx promptfoo@latest eval -c promptfooconfig_end_to_end.yaml --output ../data/end_to_end_results.json`

- To evaluate the retrieval system performance in isolation: `npx promptfoo@latest eval -c promptfooconfig_retrieval.yaml --output ../data/retrieval_results.json`

When the evaluation is complete the terminal will print the results for each row in the dataset. You can also run `npx promptfoo@latest view` to view outputs in the promptfoo UI viewer.

---

## 中文翻译

### 使用 Promptfoo 进行评估

### 前置条件

要使用 Promptfoo，你需要在系统中安装 node.js 和 npm。更多信息请参考[本指南](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)。

你可以使用 npm 安装 promptfoo，也可以直接通过 npx 运行它。本指南中我们将使用 npx。

*注意：在本示例中，你无需运行 `npx promptfoo@latest init`，因为此目录中已经存在初始化好的 `promptfooconfig.yaml` 文件。*

官方文档见[这里](https://www.promptfoo.dev/docs/getting-started)。

### 开始使用

评估由 `promptfooconfig*.yaml` 文件统一编排。在我们的应用中，我们将评估逻辑拆分为 `promptfooconfig_retrieval.yaml`（用于评估检索系统）和 `promptfooconfig_end_to_end.yaml`（用于评估端到端性能）。在这些文件中，我们定义了以下几个部分。

### 检索评估

- Prompts
    - Promptfoo 支持导入多种不同格式的 prompt。你可以在[这里](https://www.promptfoo.dev/docs/configuration/parameters)了解更多。
    - 在我们的场景中，我们不会每次都提供一个新的 prompt，而只是将 `{{query}}` 原样传递给各个检索“provider”进行评估。
- Providers
    - 我们没有使用标准的 LLM provider，而是为 `guide.ipynb` 中的每种检索方法编写了自定义 providers。
- Tests
    - 我们将使用与 `guide.ipynb` 相同的数据。我们将其拆分为 `end_to_end_dataset.csv` 和 `retrieval_dataset.csv`，并在每个数据集里添加了 `__expected` 列，以便自动对每一行运行断言。
    - 你可以在 `eval_end_to_end.py` 中找到我们的检索评估逻辑。

### 端到端评估

- Prompts
    - Promptfoo 支持导入多种不同格式的 prompt。你可以在[这里](https://www.promptfoo.dev/docs/configuration/parameters)了解更多。
    - 在我们的端到端评估配置中有 3 个 prompts：每个都对应一种方法使用场景。
        - 这些函数与 `guide.ipynb` 中使用的函数完全一致，不同之处在于它们不会调用 Claude API，而只是返回 prompt。随后由 Promptfoo 负责调用 API 并存储结果的编排。
        - 你可以在[这里](https://www.promptfoo.dev/docs/configuration/parameters#prompt-functions)进一步了解 prompt functions。使用 python 还能让我们复用 `vectordb.py` 中定义的、RAG 所需的 VectorDB class。
- Providers
    - 通过 Promptfoo，你可以连接来自不同平台的多种 LLM，详见[这里](https://www.promptfoo.dev/docs/providers)。在 `guide.ipynb` 中，我们使用的是默认 temperature 为 0.0 的 Haiku。这里我们将使用 Promptfoo 试验不同的模型。
- Tests
    - 我们将使用与 `guide.ipynb` 相同的数据。我们将其拆分为 `end_to_end_dataset.csv` 和 `retrieval_dataset.csv`，并在每个数据集里添加了 `__expected` 列，以便自动对每一行运行断言。
    - Promptfoo 提供了丰富的内置测试类型，可在[这里](https://www.promptfoo.dev/docs/configuration/expected-outputs/deterministic)查看。
    - 你可以在 `eval_retrieval.py` 中找到检索系统的测试逻辑，在 `eval_end_to_end.py` 中找到端到端系统的测试逻辑。
- Output
    - 我们定义了输出文件的路径。Promptfoo 支持将结果输出为多种格式，详见[这里](https://www.promptfoo.dev/docs/configuration/parameters/#output-file)。另外你也可以使用 Promptfoo 的 web UI，详见[这里](https://www.promptfoo.dev/docs/usage/web-ui)。

### 运行评估

要开始使用 Promptfoo，请打开终端并进入此目录（`./evaluation`）。

在运行评估之前，你必须定义以下环境变量：

`export ANTHROPIC_API_KEY=YOUR_API_KEY`
`export VOYAGE_API_KEY=YOUR_API_KEY`

在 `evaluation` 目录下，运行以下命令之一。

- 若要评估端到端系统性能：`npx promptfoo@latest eval -c promptfooconfig_end_to_end.yaml --output ../data/end_to_end_results.json`

- 若要单独评估检索系统性能：`npx promptfoo@latest eval -c promptfooconfig_retrieval.yaml --output ../data/retrieval_results.json`

评估完成后，终端会输出数据集中每一行的结果。你也可以运行 `npx promptfoo@latest view`，在 promptfoo 的 UI viewer 中查看输出结果。
