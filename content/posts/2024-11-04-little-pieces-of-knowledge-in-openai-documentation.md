---
title           : OpenAI 文档里的一些小知识
date            : 2024-11-04
isCJKLanguage   : true
---


标题括号里是官方文档里使用的术语，想深入的话可以搜索。

## 语音转录成文字（ASR、STT）

- STT：Speech to text
- ASR：Automatic Speech Recoginition

1. 支持返回时间戳，由参数 `timestamp_granularities` 控制。
2. 两种纠正词汇、提高转录准确度的方法：
    1. 在 prompt 里提供指定词汇，比如“生财有术“。限制是：有长度限制，最多 244 个 tokens，大约 150 个汉字。
    2. 在转录之后，进行后处理。比如用 gpt-4o 来根据我们提供的词汇表进行纠正。

## 函数调用（Function Call）

当一个 response 里包含多个函数调用时，表明模型认为可以并行调用。

## Batch API

价格是标准价格的 50%。适合不追求及时性的场景，比如一些异步处理任务。

可能的使用场景：Slax Reader 的导入功能。

## 内容检查（Moderation）

可检测辱骂、仇恨、自残、色情等类别的内容。

免费使用。但要注意符合公司的数据使用原则。

可能的使用场景：在 Slax Reader 里检查用户收藏的内容是否合适。

注：仅 OpenAI 提供此功能。Azure OpenAI 未提供，但默认为部署的服务提供了 Content filters 功能，且可以增加自定义策略。

## 用户滥用检测（End-user IDs）

在 API 请求中通过 user 字段向 OpenAI 提供用户 ID，供 OpenAI 向我们反馈滥用问题。在 Azure OpenAI 上可以看到被判定为滥用的用户列表。

也可以用来做一些分析和统计，比如统计不同用户的 token 使用量。

可能的使用场景：建议能使用的场景都使用，比如 Slax Reader 和 Slax Note 两款产品里。

## 提示词缓存（Prompt caching）

对长提示词（≥1024 tokens）有效，支持文字和图片 token。缓存由 OpenAI 自动维护，有效期通常是 5～10 分钟，在 OpenAI 负载低峰期可能会长达 1 个小时。
命中缓存的 tokens 费用打 5 折。但和 Batch API 的 5 折不叠加。也就是说，命中缓存 + Batch API 依然是 5 折。

可能的使用场景：Slax Reader 里的 AI Chat 功能，可以把文章内容和功能提示词放在 System Prompt 里，让它命中缓存。

## 预测输出（Predicted Outputs）

用途是减少 API 请求的耗时。

前提是事先能大致预测输出是什么。

代价是额外的费用 。具体是指会对实际输出和预测输出的不同 tokens 收费，大致计算公式是：`fee_rate x diff(predicted_outputs, response_outputs)`。

可能的使用场景：校对。比如 Slax Note 里根据用户自定义术语来修正润色结果（这是一个假设场景，真实业务设计里，把校对融入润色请求里也许更合适）。

## 降低延迟/耗时（Latency optimization）

最有效的方法是减少输出长度。大致的情况是：减少一半的输出，就能减少一半的耗时。

减少输入（比如提示词）的效果则相对会弱很多，大致的情况是：减少一半的输入，能减少 1~5%  的耗时。除非输入里有大量的内容，比如 RAG 模式下的上下文内容。

## 词嵌入（Embedding）与降维

出处：
1. https://openai.com/index/new-embedding-models-and-api-updates/
2. https://platform.openai.com/docs/guides/embeddings

text-embedding-3-* 系列的 embedding 模型在降维后，性能依然比上一代的 text-embedding-ada-002 的性能要好。

降维方法有两种：
1.（官方推荐的）设置参数 dimensions。
2.（手工）获取 embedding 后，根据需要截断，然后进行归一化。可能的场景：评估业务场景中不同维数 embedding 的表现。

## Debug 手段

一、查看 HTTP request 和 response

在调用 SDK 时，增加 with_raw_response 方法的调用。注意，同时还需要在返回的结果上调用 parse 方法。如下，留意第 4 和第 13 行：

```python
from openai import OpenAI

client = OpenAI()
response = client.chat.completions.with_raw_response.create(
    messages=[{
        "role": "user",
        "content": "Say this is a test",
    }],
    model="gpt-4o",
)
print(response.headers.get('X-My-Header'))

completion = response.parse()  # get the object that `chat.completions.create()` would have returned
print(completion)
```


二、输出 SDK 内部日志

通过设置环境变量 OPENAI_LOG 来输出：
```bash
$ export OPENAI_LOG=info
```



