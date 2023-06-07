# langchain-ChatGLM-6B

## 介绍

🚀 [_READ THIS IN ENGLISH_](README_en.md)

🤖️ 一种利用 [langchain](https://github.com/hwchase17/langchain) 思想实现的基于本地知识库的问答应用，目标期望建立一套对中文场景与开源模型支持友好、可离线运行的知识库问答解决方案。

💡 受 [GanymedeNil](https://github.com/GanymedeNil) 的项目 [document.ai](https://github.com/GanymedeNil/document.ai) 和 [AlexZhangji](https://github.com/AlexZhangji) 创建的 [ChatGLM-6B Pull Request](https://github.com/THUDM/ChatGLM-6B/pull/216) 启发，建立了全流程可使用开源模型实现的本地知识库问答应用。现已支持使用 [ChatGLM-6B](https://github.com/THUDM/ChatGLM-6B) 等大语言模型直接接入，或通过 [fastchat](https://github.com/lm-sys/FastChat) api 形式接入 Vicuna, Alpaca, LLaMA, Koala, RWKV 等模型。

✅ 本项目中 Embedding 默认选用的是 [GanymedeNil/text2vec-large-chinese](https://huggingface.co/GanymedeNil/text2vec-large-chinese/tree/main)，LLM 默认选用的是 [ChatGLM-6B](https://github.com/THUDM/ChatGLM-6B)。依托上述模型，本项目可实现全部使用**开源**模型**离线私有部署**。

⛓️ 本项目实现原理如下图所示，过程包括加载文件 -> 读取文本 -> 文本分割 -> 文本向量化 -> 问句向量化 -> 在文本向量中匹配出与问句向量最相似的`top k`个 -> 匹配出的文本作为上下文和问题一起添加到`prompt`中 -> 提交给`LLM`生成回答。

📺 [原理介绍视频](https://www.bilibili.com/video/BV13M4y1e7cN/?share_source=copy_web&vd_source=e6c5aafe684f30fbe41925d61ca6d514) 

![实现原理图](img/langchain+chatglm.png)

从文档处理角度来看，实现流程如下：

![实现原理图2](img/langchain+chatglm2.png)


🚩 本项目未涉及微调、训练过程，但可利用微调或训练对本项目效果进行优化。

🌐 [AutoDL 镜像](https://www.codewithgpu.com/i/imClumsyPanda/langchain-ChatGLM/langchain-ChatGLM)

📓 [ModelWhale 在线运行项目](https://www.heywhale.com/mw/project/6480b5e84fc1b36082e59060)

## 支持模型
若存在网络问题可在[此找到本项目涉及的所有模型](https://openi.pcl.ac.cn/Learning-Develop-Union/LangChain-ChatGLM-Webui/datasets):   
| large language model | Embedding model |
| :----: | :----: |
| ChatGLM-6B | text2vec-large-chinese |
| ChatGLM-6B-int8 | ernie-3.0-base-zh |
| ChatGLM-6B-int4 | ernie-3.0-nano-zh |
| ChatGLM-6B-int4-qe | ernie-3.0-xbase-zh | 
| Vicuna-7b-1.1 | simbert-base-chinese | 
| Vicuna-13b-1.1 | paraphrase-multilingual-MiniLM-L12-v2 | 
| BELLE-LLaMA-7B-2M |  | 
| BELLE-LLaMA-13B-2M | | 
| Minimax | |

## 变更日志

参见 [变更日志](docs/CHANGELOG.md)。

## 硬件需求

- ChatGLM-6B 模型硬件需求

    注：如未将模型下载至本地，请执行前检查`$HOME/.cache/huggingface/`文件夹剩余空间，模型文件下载至本地需要 15 GB 存储空间。
    注：一些其它的可选启动项见[项目启动选项](docs/StartOption.md)
    模型下载方法可参考 [常见问题](docs/FAQ.md) 中 Q8。
  
    | **量化等级**   | **最低 GPU 显存**（推理） | **最低 GPU 显存**（高效参数微调） |
    | -------------- | ------------------------- | --------------------------------- |
    | FP16（无量化） | 13 GB                     | 14 GB                             |
    | INT8           | 8 GB                     | 9 GB                             |
    | INT4           | 6 GB                      | 7 GB                              |


- Embedding 模型硬件需求

    本项目中默认选用的 Embedding 模型 [GanymedeNil/text2vec-large-chinese](https://huggingface.co/GanymedeNil/text2vec-large-chinese/tree/main) 约占用显存 3GB，也可修改为在 CPU 中运行。

## Docker 部署
为了能让容器使用主机GPU资源，需要在主机上安装 [NVIDIA Container Toolkit](https://github.com/NVIDIA/nvidia-container-toolkit)。具体安装步骤如下：
```shell
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit-base
sudo systemctl daemon-reload 
sudo systemctl restart docker
```
安装完成后，可以使用以下命令编译镜像和启动容器：
```
docker build -f Dockerfile-cuda -t chatglm-cuda:latest .
docker run --gpus all -d --name chatglm -p 7860:7860  chatglm-cuda:latest

#若要使用离线模型，请配置好模型路径，然后此repo挂载到Container
docker run --gpus all -d --name chatglm -p 7860:7860 -v ~/github/langchain-ChatGLM:/chatGLM  chatglm-cuda:latest
```


## 开发部署

### 软件需求

本项目已在 Python 3.8.1 - 3.10，CUDA 11.7 环境下完成测试。已在 Windows、ARM 架构的 macOS、Linux 系统中完成测试。

vue前端需要node18环境

### 从本地加载模型

请参考 [THUDM/ChatGLM-6B#从本地加载模型](https://github.com/THUDM/ChatGLM-6B#从本地加载模型)

### 1. 安装环境

参见 [安装指南](docs/INSTALL.md)。

### 2. 设置模型默认参数

在开始执行 Web UI 或命令行交互前，请先检查 [configs/model_config.py](configs/model_config.py) 中的各项模型参数设计是否符合需求。

如需通过 fastchat 以 api 形式调用 llm，请参考 [fastchat 调用实现](docs/fastchat.md)

### 3. 执行脚本体验 Web UI 或命令行交互

> 注：鉴于环境部署过程中可能遇到问题，建议首先测试命令行脚本。建议命令行脚本测试可正常运行后再运行 Web UI。

执行 [cli_demo.py](cli_demo.py) 脚本体验**命令行交互**：
```shell
$ python cli_demo.py
```

或执行 [webui.py](webui.py) 脚本体验 **Web 交互**

```shell
$ python webui.py
```

或执行 [api.py](api.py) 利用 fastapi 部署 API
```shell
$ python api.py
```
或成功部署 API 后，执行以下脚本体验基于 VUE 的前端页面
```shell
$ cd views 

$ pnpm i

$ npm run dev
```

VUE 前端界面如下图所示：
1. `对话` 界面
![](img/vue_0521_0.png)
2. `知识库问答` 界面
![](img/vue_0521_1.png)
3. `Bing搜索` 界面
![](img/vue_0521_2.png)

WebUI 界面如下图所示：
1. `对话` Tab 界面
![](img/webui_0521_0.png)
2. `知识库测试 Beta` Tab 界面
![](img/webui_0510_1.png)
3. `模型配置` Tab 界面
![](img/webui_0510_2.png)

Web UI 可以实现如下功能：

1. 运行前自动读取`configs/model_config.py`中`LLM`及`Embedding`模型枚举及默认模型设置运行模型，如需重新加载模型，可在 `模型配置` Tab 重新选择后点击 `重新加载模型` 进行模型加载；
2. 可手动调节保留对话历史长度、匹配知识库文段数量，可根据显存大小自行调节；
3. `对话` Tab 具备模式选择功能，可选择 `LLM对话` 与 `知识库问答` 模式进行对话，支持流式对话；
4. 添加 `配置知识库` 功能，支持选择已有知识库或新建知识库，并可向知识库中**新增**上传文件/文件夹，使用文件上传组件选择好文件后点击 `上传文件并加载知识库`，会将所选上传文档数据加载至知识库中，并基于更新后知识库进行问答；
5. 新增 `知识库测试 Beta` Tab，可用于测试不同文本切分方法与检索相关度阈值设置，暂不支持将测试参数作为 `对话` Tab 设置参数。
6. 后续版本中将会增加对知识库的修改或删除，及知识库中已导入文件的查看。

### 常见问题

参见 [常见问题](docs/FAQ.md)。

## 🙇‍ ‍感谢

1. [langchain-ChatGLM](https://github.com/imClumsyPanda/langchain-ChatGLM)提供的基础框架
