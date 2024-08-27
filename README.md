本爬虫使用Python开发,通过[初始化一个Scrapy项目](https://docs.scrapy.org/en/latest/intro/tutorial.html#creating-a-project)并实现一个自定义爬虫。该爬虫配备了`parse_page`函数,每当爬虫访问一个网页时都会调用此函数。这个回调函数将网页上的文本分割成块,为每个块生成向量嵌入,并将这些向量更新插入到您的Upstash向量数据库中。存储在我们数据库中的每个向量都包含原始文本和网站URL作为元数据。

### 配置环境变量

在运行爬虫之前,我们需要配置环境变量。它们让我们可以安全地存储敏感信息,比如与 OpenAI 或 Upstash Vector 通信所需的API密钥。

如果您还没有 Upstash向量数据库,请在[这里](https://console.upstash.com/vector)创建一个,并将向量维度设置为1536。我们在这里设置1536是因为这是我们将使用的嵌入模型所需的数量。

应设置以下环境变量:

```
# Upstash Vector凭证可在此处获取: https://console.upstash.com/vector
UPSTASH_VECTOR_REST_URL=****
UPSTASH_VECTOR_REST_TOKEN=****

# OpenAI密钥可在此处获取: https://platform.openai.com/api-keys
OPENAI_API_KEY=****
```
### 安装所需的Python库

要安装这些库,我们建议设置一个虚拟Python环境。在开始安装之前,请导航到`fflowwebcrawler`目录。

要设置虚拟环境,首先安装`virtualenv`包:

```bash
pip install virtualenv
```

然后,创建一个新的虚拟环境并激活它:

```bash
# 创建环境
python3 -m venv venv

# 激活环境
source venv/bin/activate
```

最后,使用 `requirements.txt` 安装所需的库:

```bash
pip install -r requirements.txt
```

</details>

</br>

设置这些环境变量后,我们几乎准备好运行爬虫了。下一步涉及配置爬虫本身,主要通过`crawler.yaml`文件完成。此外,还必须在`settings.py`文件中处理一个关键设置。

```yaml
crawler:
  start_urls:
    - https://www.some.domain.com
  link_extractor:
    allow: '.*some\.domain.*'
    deny:
      - "#"
      - '\?'
      - about
index:
  openAI_embedding_model: text-embedding-ada-002
  text_splitter:
    chunk_size: 1000
    chunk_overlap: 100
```

在`crawler`部分,有两个子部分:

- `start_urls`: 我们的爬虫将开始搜索的入口点
- `link_extractor`: 作为参数传递给[`scrapy.linkextractors.LinkExtractor`](https://docs.scrapy.org/en/latest/topics/link-extractors.html)的字典。一些重要参数是:
  - `allow`: 只提取匹配给定正则表达式的链接
  - `allow_domains`: 只提取匹配给定域名的链接
  - `deny`: 拒绝匹配给定正则表达式的链接

在`index`部分,有两个子部分:

- `openAI_embedding_model`: 要使用的嵌入模型
- `test_splitter`: 作为参数传递给[`langchain.text_splitter.RecursiveCharacterTextSplitter`](https://api.python.langchain.com/en/latest/text_splitter/langchain.text_splitter.RecursiveCharacterTextSplitter.html)的字典


就是这样! 🎉 我们已经配置了爬虫,准备使用以下命令运行它:

```
scrapy crawl configurable --logfile crawler.log
```

请注意,运行这个可能需要一些时间。您可以通过查看日志文件`fflowwebcrawler.log`或如下所示的Upstash向量数据库仪表板的指标来监控进度。
```