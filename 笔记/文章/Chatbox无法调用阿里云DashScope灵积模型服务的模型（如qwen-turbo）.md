

# 报错
>遇到了来自 OpenAI API 的错误，一般是由错误设置或账户问题引起的。请检查 AI 设置和账户情况，或者点击这里查看常见问题文档。
API Error: Status Code 400, {"error":{"code":"invalid_parameter_error","param":null,"message":"Range of top_p should be (0.0, 1.0)","type":"invalid_request_error"}}


![qwen-turbo](https://img-blog.csdnimg.cn/direct/69af117d3ed84b90ae8a661a4cbd7bf3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/5e26463e57b74487a82cf7444c5c98e2.png)


# 分析与解决

首先，chatbox默认top p 为1，`Range of top_p should be (0.0, 1.0)` ，

### ***只需要将top p 调整到0.99，***
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/0bfd6c90a89d45febcaa2bb2c91270da.png)
---
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/15346259f7e04c9a88f4242a8bb39555.png)
### 阿里的官方文档
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/414726673f214c1699e14fac2a8cf02f.png)
阿里这里要求float类型

### openai官方文档
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/3800f4f0efa44b10a499020fb9533bbe.png)
这里defaults to 1 ，只要求number类型就行

本质上qwen是兼容openai，但不完全兼容，这部分的兼容没做好

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/df374229e0d849328c36c5b5c6802e83.png)

Reference:
[OpenAI: top p](https://platform.openai.com/docs/api-reference/chat/create#chat-create-top_p)

[如何通过OpenAI接口调用通义千问模型_模型服务灵积(DashScope)-阿里云帮助中心 (aliyun.com)](https://help.aliyun.com/zh/dashscope/developer-reference/compatibility-of-openai-with-dashscope/?spm=a2c4g.11186623.0.0.14314ad0Lt21gv#56e9761e923mg)