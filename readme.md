# To do list
* 更新同步最新版的ZhipuAI sdk
* 实现基于langchain 调用的流式输出
# 教程

## 环境准备

* 测试环境为 **Python 3.9.17** 更高等级兼容 ，如果使用python3.7一下版本，可能gradio包会报错不兼容，需要降级到 gradio==3.0.12

* 依赖库：

  ```python
  pip install scikit-learn==1.1.2   joblib==1.3.2    zhipuai==1.0.7 gradio==3.41.2
  ```
## 文件说明
* images目录  存储了16张标记的mbti性格卡通人物图片，形象化显示
* modelBK目录 存储了旧的模型pkl文件，备份，BK/目录下的为最新的参数，训练数据新增几十条，增加输入的容错性
* trainclassifer.ipynb 逻辑回归分类器训练代码
* train.xlsx 分类器训练数据，比较少
* MbtiTest.ipynb 主程序文件



## 系统构成

### 一、mbti性格分类器

基于30多条csv数据训练的逻辑回归模型，训练数据见文件**train.csv**

模型参数文件：**model.pkl**

特征编码器文件：**vectorizer.pkl**

标签编码器文件：**label_encoder.pkl**

加载模型方式

```python
model = joblib.load('model.pkl')
vectorizer = joblib.load('vectorizer.pkl')
label_encoder = joblib.load('label_encoder.pkl')
```

### 二、ChatGLM大语言模型

基于智谱ai提供的API接口，免去了本地搭建部署的麻烦，但缺点是数据返回较慢，平均**20s左右**，基于同步传输

交互方式

```python
def chat_with_glm(prompt_input):
    # 和大模型交互
    response = zhipuai.model_api.invoke(
        model="chatglm_std",#模型型号
        prompt=[#可以自定义模型角色，认知
            {"content": "你好", "role": "user"},
            {"content": "我是一个资深的mbti性格分析师", "role": "assistant"},
            {"content": "你叫什么名字", "role": "user"},
            {"content": "我叫chatGLM", "role": "assistant"},
            {"content": prompt_input, "role": "user"}
        ]
    )
    # 返回大模型的回复
    return response["data"]["choices"][0]["content"]
```



## 系统工作流程

用户输入四个问题的答案，输入逻辑回归分类器，进行mbti16种人格分类，然后构建mbti_prompt，输入大模型ChatGLM中，生成大模型建议对策，返回前端展示，同时前端会标出用户属于哪种mbti人格，如下图

![img](https://gitee.com/typora_picture_bed/picture_bed_2/raw/master/pic2/20230829210021.png)



## 运行方式

按照**MbtiTest.ipynb** 文件一步步运行即可

会生成两个url，有可能只有第一个，看平台

![image-20230829134227010](https://gitee.com/typora_picture_bed/picture_bed_2/raw/master/pic2/20230829134227.png)

* Running on local URL:  http://0.0.0.0:7865 #本机可以访问

* 公网都可以访问

  Running on public URL: https://34cdadd0045489be69.gradio.live 
## 改进思路
### 扩大训练数据集train.xlsx
* 目前训练数据太少
### 增加验证集valid.xlsx
* 因为训练数据集太少，没有做验证，可以先在验证集测试，找到分类正确率较高的那个模型
### 更换模型编码器
* 与词嵌入和预训练模型相比，TF-IDF不太可能捕获深层次的语义信息
* 但试验过用word2vec 编码器，效果不是很好
* 如果能用bert之类的预训练大语言模型训练一个embedding，效果会好一些
### 更换分类模型
* 默认用的是逻辑回归
* 还可以考虑 NN 神经网络、随机森林、SVM等
* 或者使用预训练的大语言模型，如GPT、bert 更能捕捉语义信息
