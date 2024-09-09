
### 前言


    近两年`AIGC`发展的非常迅速，从刚开始的只有`ChatGPT`到现在的很百家争鸣。从开始的大参数模型，再到后来的小参数模型，从一开始单一的文本模型到现在的多模态模型等等。随着一起进步的不仅仅是模型的多样化，还有模型的使用方式。大模型使用的门槛越来越低，甚至现在每个人都可以在自己的电脑上运行模型。今天我们要说的就是大模型工具中的佼佼者`Ollama`,并演示如何通过`C#`来使用`Ollama`。


### Ollama


    `Ollama`是一个开源的大语言模型（LLM）服务工具，它允许用户在本地PC环境快速实验、管理和部署大型语言模型。它支持多种流行的开源大型语言模型，如 `Llama 3.1`、`Phi 3`、`Qwen 2`、`GLM 4`等，并且可以通过命令行界面轻松下载、运行和管理这些模型。`Ollama`的出现是为了降低使用大型语言模型的门槛，是让大型语言模型更加普及和易于访问。一言以蔽之就是`Ollama让使用模型更简单`。无论是`CPU`或是`GPU`都可以，算力高的话推理速度更快，算力不足的话推理的慢，而且容易胡言乱语。


#### 安装


`Ollama`的安装方式常用的有两种，一种是去官网下载，另一种是去GitHub下载，可以选择对应的系统版本进行下载


* 官网首页直接下载 [https://ollama.com/](https://github.com)
* Github Relase下载 [https://github.com/ollama/ollama/releases](https://github.com)


我的是Windows操作系统，所以直接下载一路Next就可以，默认安装在`C盘`无法更改，强迫症的话可以通过`mklink`做链接，但是自动更新之后还是在`C盘`。自动升级这一块不用太担心，联网的情况，如果有新版本`Ollama`会推送更新。


安装完成之后可以修改常用的环境变量


* 通过`OLLAMA_MODELS`环境变量设置模型下载的位置，默认是在`C盘`，可以换成其他地址。
* 通过`OLLAMA_HOST`设置`Ollama`服务监听的端口，默认的是`11434`。


安装完成之后通过`version`查看，如果显示版本号则安装成功。



```
ollama --version

```

比较常用的指令不多，也很简单


* `ollama list`列出本地下载的模型
* `ollama ps`查看正在运行的模型
* `ollama pull 模型标识`下载模型到本地，比如我要下载`qwen2 7b`则使用`ollama pull qwen2:7b`
* `ollama run 模型标识`运行模型，如果已下载则直接运行，如果没下载则先下载再运行。比如我要运行`qwen2 7b`可以直接运行`ollama run qwen2:7b`


也可以将本地已有的`GGUF`模型导入到`Ollama`中去，操作也很简单。


1. 编写一个名为`Modelfile`的文件，写入以下内容



```
FROM 模型路径/qwen2-0_5b-instruct-q8_0.gguf

```

2. 通过`Ollama`创建模型



```
ollama create qwen2:0.5b -f Modelfile

```

3. 运行刚创建的模型



```
ollama run qwen2:0.5b

```

需要注意的是运行`7B`至少需要`8GB`的内存或显存，运行`13B`至少需要`16GB`内存或显存。我电脑的配置信息如下



```
型号: 小新Pro16 AI元启
CPU: AMD Ryzen 7 8845H
内存: 32.0 GB

```

`AMD Ryzen 7 8845H`内置`NPU`，整体算力还可以, 运行运行`13B`及以下的模型没太大问题。当然这种级别参数大小的模型不会是一个无所不能的模型，这种量级的模型运行成本相对较低，适合做一些特定场景的推理任务。如果需要无所不能的模型建议还是直接使用`ChatGPT`这种商业模型。


#### 命令启动


下载模型完成之后可以测试运行，通过`cmd`运行指令，比如我运行起来`qwen2:7b`模型
[![](https://img2024.cnblogs.com/blog/2042116/202409/2042116-20240904102931244-1275660501.png)](https://github.com)
这种方式比较简单，只能是文字对话的方式而且没有样式，简单粗暴。


#### 接口访问


`Ollama`提供服务的本质还是`http`接口，我们可以通过http接口的方式来调用`/api/generate`接口



```
curl http://localhost:11434/api/generate -d '{
  "model": "qwen2:7b",
  "prompt": "请你告诉我你知道的天气有哪些？用json格式输出",
  "stream": false
}'

```

* `model`设置模型的名称
* `prompt`提示词
* `stream`设置为`false`要求不要流式返回


因为是一次性返回所有内容，所以需要等待一会，如果需要流式输出可以设置为`true`。等待一会后接口返回的信息如下所示



```
{
    "model": "qwen2:7b",
    "created_at": "2024-09-04T06:13:53.1082355Z",
    "response": "```json\n{\n  \"常见天气\": [\n    {\n      \"类型\": \"晴\",\n      \"描述\": \"天空无云或有少量高薄云，日间阳光充足。\",\n      \"符号\": \"☀️\"\n    },\n    {\n      \"类型\": \"多云\",\n      \"描述\": \"大部分天空被云层覆盖，但能见蓝天，太阳时隐时现。\",\n      \"符号\": \"🌤️\"\n    },\n    {\n      \"类型\": \"阴天\",\n      \"描述\": \"全天或大部分时间云量较多，几乎看不到阳光，光线较暗。\",\n      \"符号\": \"☁️\"\n    },\n    {\n      \"类型\": \"雨\",\n      \"子类型\": [\n        {\n          \"类型\": \"小雨\",\n          \"描述\": \"降水量不大，通常不会形成积水。\",\n          \"符号\": \"🌦️\"\n        },\n        {\n          \"类型\": \"中雨\",\n          \"描述\": \"降水量适中，可能会有局部积水。\",\n          \"符号\": \"🌧️\"\n        },\n        {\n          \"类型\": \"大雨\",\n          \"描述\": \"降水量大，可能伴有雷电和强风。\",\n          \"符号\": \"⛈️\"\n        }\n      ]\n    },\n    {\n      \"类型\": \"雪\",\n      \"子类型\": [\n        {\n          \"类型\": \"小雪\",\n          \"描述\": \"积雪较轻，地面可能仅局部有薄雪覆盖。\",\n          \"符号\": \"❄️\"\n        },\n        {\n          \"类型\": \"中雪\",\n          \"描述\": \"降雪量中等，地面和部分植被可能有积雪。\",\n          \"符号\": \"🌨️\"\n        },\n        {\n          \"类型\": \"大雪\",\n          \"描述\": \"降雪量很大，地面积雪深厚，交通和生活受严重影响。\",\n          \"符号\": \"❄️💨\"\n        }\n      ]\n    },\n    {\n      \"类型\": \"雾\",\n      \"描述\": \"大气中的水汽在地面或近地面凝结形成大量悬浮的微小水滴或冰晶的现象。\",\n      \"符号\": \"🌫️\"\n    },\n    {\n      \"类型\": \"雷阵雨\",\n      \"描述\": \"突然而短暂的强降雨伴有闪电和雷鸣，通常持续时间较短。\",\n      \"符号\": \"⚡🌧️\"\n    }\n  ]\n}\n```",
    "done": true,
    "done_reason": "stop",
    "context": [
        151644,
        872,
        198,
        //...省略...
        73594
    ],
    "total_duration": 70172634700,
    "load_duration": 22311300,
    "prompt_eval_count": 19,
    "prompt_eval_duration": 151255000,
    "eval_count": 495,
    "eval_duration": 69997676000
}

```

还有一种比较常用的操作就是大家比较关注的`嵌入模型`，通俗点就是对文本或者图片、视频等信息进行特征提取转换成向量的方式，这时候需要使用`/api/embed`接口,请求格式如下所示，这里使用的向量化模型是`nomic-embed-text`大家可以自行去用`ollama pull`这个模型



```
curl http://localhost:11434/api/embed -d '{
  "model": "nomic-embed-text:latest",
  "input": "我是中国人，我爱我的祖国"
}'

```

嵌入接口返回的数据格式如下所示



```
{
    "model": "nomic-embed-text:latest",
    "embeddings": [
        [
            0.012869273,
            0.015905218,
            -0.13998738,
            //...省略很多...
            -0.035138983,
            -0.03351391
        ]
    ],
    "total_duration": 619728100,
    "load_duration": 572422600,
    "prompt_eval_count": 12
}

```

当然`Ollama`提供的接口还有很多，比如对话、模型管理等待，这里我们就不一一介绍了，有需要的同学可以自行查阅接口文档地址[https://github.com/ollama/ollama/blob/main/docs/api.md](https://github.com)


#### 可视化UI


上面我们提到了两种方式访问`Ollama`服务，一种是命令行的方式，另一种是接口的方式。这两种虽然方式原始，但是并没有界面操作显得直观，如果你想通过界面的方式通过`Ollama`完成对话服务，官方`Github`推荐的也比较多，有兴趣的同学可以自行查看文档[https://github.com/ollama/ollama?tab\=readme\-ov\-file\#web\-\-desktop](https://github.com)，我选用的是第一个[Open WebUI](https://github.com)，简单的方式是通过`Docker`直接运行



```
docker run -d -p 3000:8080 -e OLLAMA_BASE_URL=https://你的ollama服务ip:11434 -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main

```

亦或者可以通过构建源码的方式构建启动，按照下面的命令一步一步来


1. git clone [https://github.com/open\-webui/open\-webui.git](https://github.com)
2. cd open\-webui/
3. cp \-RPp .env.example .env（复制一份`.env.example`文件重命名为`.env`。Windows系统的话使用copy .env.example .env）
4. npm install
5. npm run build
6. cd ./backend
7. conda create \-\-name open\-webui\-env python\=3\.11（用conda创建一个名为pen\-webui\-env的虚拟环境）
8. conda activate open\-webui\-env（激活虚拟环境）
9. pip install \-r requirements.txt \-U（安装python依赖）
10. bash start.sh（ Windows操作系统的话直接启动start\_windows）


如你所见，它是依赖`NodeJs`和`Python`的，还需要安装`Conda`


* 🐰 Node.js \>\= 20\.10
* 🐍 Python \>\= 3\.11
* conda我用的是24\.5\.0


启动成功后，在浏览器上输入`http://localhost:8080/`,注册一个用户名登陆进来之后界面如下所示
[![](https://img2024.cnblogs.com/blog/2042116/202409/2042116-20240904151109805-1320033524.png)](https://github.com)
可以直接选择模型进行对话，类似`ChatGPT`那种对话风格。


### C\#整合Ollama


上面我们了解到了`Ollama`的基本安装和使用，明白了它的调用是基于`Http接口`来完成的。其实我也可以参考接口文档自行封装一套调用，但是没必要, 因为有很多现成的SDK可以直接使用。


#### 使用Ollama Sdk


这里使用的C\#的SDK就叫0llama，它的Github地址是[https://github.com/tryAGI/Ollama](https://github.com), 为什么选择它呢，其实也很简单，因为它支持`function call`,这方便我们更早的体验新功能。安装它非常简单，相信同学们都会



```
dotnet add package Ollama --version 1.9.0

```

##### 简单对话


简单的对话功能上手也没什么难度，都是简单代码



```
string modelName = "qwen2:7b";
using var ollama = new OllamaApiClient(baseUri: new Uri("http://127.0.0.1:11434/api"));

Console.WriteLine("开始对话！！！");
string userInput = "";
do
{
    Console.WriteLine("User:");
    userInput = Console.ReadLine()!;
    var enumerable = ollama.Completions.GenerateCompletionAsync(modelName, userInput);
    Console.WriteLine("Agent:");
    await foreach (var response in enumerable)
    {
        Console.Write($"{response.Response}");
    }
    Console.WriteLine();

} while (!string.Equals(userInput, "exit", StringComparison.OrdinalIgnoreCase));
Console.WriteLine("对话结束！！！");

```

模型名称是必须要传递的，而且默认的是`流式输出`,如果想一次返回同样的是设置`stream为false`。示例使用的是`qwen2:7b`模型。执行起来之后便可以直接对话，如下所示
[![](https://img2024.cnblogs.com/blog/2042116/202409/2042116-20240905104244799-945119164.png)](https://github.com):[wgetCloud机场](https://tabijibiyori.org)
整体来说国产模型里面`qwen2:7b`整体的效果还是不错的，至少还不是扭曲事实。


##### 多轮对话


如果需要进行分角色的多轮对话，要换一个方式使用，使用提供的`Chat`方式，如下所示



```
string modelName = "glm4:9b";
using var ollama = new OllamaApiClient(baseUri: new Uri("http://127.0.0.1:11434/api"));
Console.WriteLine("开始对话！！！");
string userInput = "";
List messages = [];
do
{
    //只取最新的五条消息
    messages = messages.TakeLast(5).ToList();
    Console.WriteLine("User:");
    userInput = Console.ReadLine()!;
    //加入用户消息
    messages.Add(new Message(MessageRole.User, userInput));
    var enumerable = ollama.Chat.GenerateChatCompletionAsync(modelName, messages, stream: true);
    Console.WriteLine("Agent:");
    StringBuilder builder = new();
    await foreach (var response in enumerable)
    {
        string content = response.Message.Content;
        builder.AppendLine(content);
        Console.Write(content);
    }
    //加入机器消息
    messages.Add(new Message(MessageRole.Assistant, builder.ToString()));
    Console.WriteLine();

} while (!string.Equals(userInput, "exit", StringComparison.OrdinalIgnoreCase));
Console.WriteLine("对话结束！！！");

```

这次换了另一个国产模型`glm4:9b`, 多轮对话和完全对话使用的对象不同。


* 完全对话使用的是Completions对象，多轮对话使用的是`Chat`对象。
* 多轮对话需要用`List`存储之前的对话记录，这里模型才能捕获上下文。


运行起来，执行效果如下所示
[![](https://img2024.cnblogs.com/blog/2042116/202409/2042116-20240905105611315-1456276216.png)](https://github.com)
第一次我问他会c\#吗，它说了一堆表示会。第二句我让它写一个简单的示例，但是我并没有说写c\#示例，但是它可以通过上面的对话了解到意图，所以直接用c\#给我写了一个示例。


##### function call


高版本的`Ollama`支持`function call`，当然这也要求模型也必须支持，如果模型本身不支持，那也是没有效果的，其中`llama3.1`支持的比较好，美中不足是`llama3.1`对中文支持的不太好，所以我们简单的演示一下，这里使用的是`llama3.1:8b`模型，首先需要定义方法，这样和模型对话的时候，框架会把方法的元信息抽出来发给模型，让模型判断调用哪个，这里我简单定义了一个计算增删改查的接口，并实现这个接口。



```
//定义一个接口，提供元信息
[OllamaTools]
public interface IMathFunctions
{
    [Description("Add two numbers")]
    int Add(int a, int b);
    [Description("Subtract two numbers")]
    int Subtract(int a, int b);
    [Description("Multiply two numbers")]
    int Multiply(int a, int b);
    [Description("Divide two numbers")]
    int Divide(int a, int b);
}

//实现上面的接口提供具体的操作方法
public class MathService : IMathFunctions
{
    public int Add(int a, int b) => a + b;
    public int Subtract(int a, int b) => a - b;
    public int Multiply(int a, int b) => a * b;
    public int Divide(int a, int b) => a / b;
}

```

有了上面的接口和实现类之后，我们就可以通过`Ollama`使用它们了，使用方式如下



```
string modelName = "llama3.1:8b";
using var ollama = new OllamaApiClient(baseUri: new Uri("http://127.0.0.1:11434/api"));
var chat = ollama.Chat(
    model: modelName,
    systemMessage: "You are a helpful assistant.",
    autoCallTools: true);

//给Ollama注册刚才定义的类
var mathService = new MathService();
chat.AddToolService(mathService.AsTools(), mathService.AsCalls());

while (true)
{
    try
    {
        Console.WriteLine("User>");
        var newMessage = Console.ReadLine();
        var msg = await chat.SendAsync(newMessage);
        Console.WriteLine("Agent> " + msg.Content);
    }
    finally
    {
        //打印本次对话的所有消息
        Console.WriteLine(chat.PrintMessages());
    }
}

```

这里需要设置`autoCallTools`为`true`才能自动调用方法，`PrintMessages()`方法用来打印本轮会话中所有的消息, 一般自动调用`function call`的时候会产生多次请求，但是我们使用的时候是无感知的，因为框架已将帮我自动处理了，比如我的提示词是一个数学计算公式`(12+8)*4/2=?`，如下所所示
[![](https://img2024.cnblogs.com/blog/2042116/202409/2042116-20240905133400093-159717952.png)](https://github.com)
通过`PrintMessages()`方法打印的对话消息可知，虽然我只提供了一句提示词，但是`Ollama SDK`因为支持自动调用工具，`llama3.1:8b`将提示词算式`(12+8)*4/2)`进行了拆分，计算步骤如下所示


* 先拆分了括号里的逻辑`12+8`并调用`Add`方法得到结果20
* 然后第二步用上一步得到的结果调用`Multiply`计算`20*4`得到80
* 再用上一步的结果调用`Divide`计算`80/2`得到结果40
* 最后把Tools调用的步骤及结果一起在通过对话发送给`llama3.1`模型，模型得到最终的输出


如果我们不打印过程日志的话，模型只会输出



```
Assistant:
The correct calculation is:
(12+8)=20
20*4=80
80/2=40
Therefore,the answer is:40.

```

##### 嵌入模型


上面我们提到过`Ollama`不仅可以使用对话模型还可以使用`嵌入模型`的功能，`嵌入模型`简单的来说就是对文本、图片、语音等利用模型进行特征提起，得到向量数据的过程。通过`Ollama SDK`可以使用`Ollama`的嵌入功能，代码如下所示



```
 string modelName = "nomic-embed-text:latest";
 HttpClient client = new HttpClient();
 client.BaseAddress = new Uri("http://127.0.0.1:11434/api");
 client.Timeout = TimeSpan.FromSeconds(3000);
 using var ollama = new OllamaApiClient(client);
 var embeddingResp =  await ollama.Embeddings.GenerateEmbeddingAsync(modelName, "c#是一门不错的编程语言");
 Console.WriteLine($"[{string.Join(",", embeddingResp.Embedding!)}]");

```

得到的就是如下所示的向量信息
[![](https://img2024.cnblogs.com/blog/2042116/202409/2042116-20240905142033490-575462444.png)](https://github.com)
向量数据是可以计算相似度的，利用`余弦夹角`的概念可以计算向量的空间距离，空间距离越近，两个向量的相似度便越高。如果大家了解颜色表`RGB`的话就比较容易理解，举个例子`(255, 0, 0)`就是纯红色，`(255, 10, 10)`也是红色，但是不是纯红色。如果把`(255, 0, 0)`和`(255, 10, 10)`映射到一个三维的空间坐标图上它们的距离就很近，但是它们和纯蓝色`(0, 0, 255)`的空间距离就很远，因为一个贴近`X`轴，一个贴近`Z`轴。现在大家锁熟知的向量数据库，大概采用的就是类似的原理。也是现在流行的`RAG`检索增强生成的基础。


比如我把下面两句话嵌入模型得到向量值，然后通过计算`余弦夹角`来比较它们的相似度



```
var embeddingResp =  await ollama.Embeddings.GenerateEmbeddingAsync(modelName, "c#是一门不错的编程语言");
var embeddingResp2 = await ollama.Embeddings.GenerateEmbeddingAsync(modelName, "c#是很好的语言");
Console.WriteLine("相似度：" + CosineSimilarity([.. embeddingResp.Embedding!], [.. embeddingResp2!.Embedding]));

//计算余弦夹角
public static double CosineSimilarity(double[] vector1, double[] vector2)
{
    if (vector1.Length != vector2.Length)
        throw new ArgumentException("向量长度必须相同");

    double dotProduct = 0.0;
    double magnitude1 = 0.0;
    double magnitude2 = 0.0;

    for (int i = 0; i < vector1.Length; i++)
    {
        dotProduct += vector1[i] * vector2[i];
        magnitude1 += vector1[i] * vector1[i];
        magnitude2 += vector2[i] * vector2[i];
    }

    magnitude1 = Math.Sqrt(magnitude1);
    magnitude2 = Math.Sqrt(magnitude2);

    if (magnitude1 == 0.0 || magnitude2 == 0.0)
        return 0.0; // 避免除以零

    return dotProduct / (magnitude1 * magnitude2);
}

```

上面的得到的相似度结果是



```
相似度：0.9413230998586363

```

因为它们两句话表达的含义差不多，所以相似度很高。但是如果我要计算下面的两句话的相似度



```
var embeddingResp =  await ollama.Embeddings.GenerateEmbeddingAsync(modelName, "c#是一门不错的编程语言");
var embeddingResp2 = await ollama.Embeddings.GenerateEmbeddingAsync(modelName, "我喜欢吃芒果和草莓");

```

那么利用余弦值计算出来它们的相似度只有`0.59`，因为这两句话几乎没有任何关联。



```
相似度：0.5948448463206064

```

##### 多模态模型


刚开始的对话模型都比较单一，都是简单的文本对话，随着不断的升级，有些模型已经支持多种格式的输入输出而不仅仅是单一的文本，比如支持图片、视频、语音等等，这些模型被称为多模态模型。使用`Ollama`整合`llava`模型体验一把，这里我是用的是`llava:13b`。我在网上随便找了一张图片存放本地[![](https://img2024.cnblogs.com/blog/2042116/202409/2042116-20240905145135636-2080743229.jpg)](https://github.com)
用这张图片对模型进行提问，代码如下所示



```
HttpClient client = new HttpClient();
client.BaseAddress = new Uri("http://127.0.0.1:11434/api");
client.Timeout = TimeSpan.FromSeconds(3000);
using var ollama = new OllamaApiClient(client);
string modelName = "llava:13b";
string prompt = "What is in this picture?";
System.Drawing.Image image = System.Drawing.Image.FromFile("1120.jpg");
var enumerable = ollama.Completions.GenerateCompletionAsync(modelName, prompt, images: [BitmapToBase64(image)], stream: true);
await foreach (var response in enumerable)
{
    Console.Write($"{response.Response}");
}

//Image转base64
public static string BitmapToBase64(System.Drawing.Image bitmap)
{
    MemoryStream ms1 = new MemoryStream();
    bitmap.Save(ms1, System.Drawing.Imaging.ImageFormat.Jpeg);
    byte[] arr1 = new byte[ms1.Length];
    ms1.Position = 0;
    ms1.Read(arr1, 0, (int)ms1.Length);
    ms1.Close();
    return Convert.ToBase64String(arr1);
}

```

我用提示词让模型描述图片里面的内容，然后把这张图片转换成`base64`编码格式一起发送给模型，模型返回的内容如下所示
[![](https://img2024.cnblogs.com/blog/2042116/202409/2042116-20240905145851802-1222105384.png)](https://github.com)
确实够强大，描述的信息很准确，措词也相当不错，如果让人去描述图片中的内容，相信大部分人描述的也没这么好，不得不说模型越来越强大了。


#### 使用SemanticKernel


除了整合`Ollama SDK`以外,你还可以用`Semantic Kernel`来整合`Ollama`，我们知道默认情况下`Semantic Kernel`只能使用`OpenAI`和`Azure OpenAI`的接口格式，但是其他模型接口并不一定和`OpenAI`接口格式做兼容，有时候甚至可以通过`one-api`这样的服务来适配一下。不过不用担心`Ollama`兼容了`OpenAI`接口的格式，即使不需要任何的适配服务也可以直接使用，我们只需要重新适配一下请求地址即可。



```
using HttpClient httpClient = new HttpClient(new RedirectingHandler());
httpClient.Timeout = TimeSpan.FromSeconds(120);

var kernelBuilder = Kernel.CreateBuilder()
    .AddOpenAIChatCompletion(
       modelId: "glm4:9b",
       apiKey: "ollama",
       httpClient: httpClient);
Kernel kernel = kernelBuilder.Build();

var chatCompletionService = kernel.GetRequiredService();
OpenAIPromptExecutionSettings openAIPromptExecutionSettings = new()
{
    ToolCallBehavior = ToolCallBehavior.AutoInvokeKernelFunctions
};

var history = new ChatHistory();
string? userInput;
do
{
    Console.Write("User > ");
    userInput = Console.ReadLine();
    history.AddUserMessage(userInput!);

    var result = chatCompletionService.GetStreamingChatMessageContentsAsync(
        history,
        executionSettings: openAIPromptExecutionSettings,
        kernel: kernel);
    string fullMessage = "";
    System.Console.Write("Assistant > ");
    await foreach (var content in result)
    {
        System.Console.Write(content.Content);
        fullMessage += content.Content;
    }
    System.Console.WriteLine();

    history.AddAssistantMessage(fullMessage);
} while (userInput is not null);


public class RedirectingHandler : HttpClientHandler
{
    protected override Task SendAsync(
        HttpRequestMessage request, CancellationToken cancellationToken)
    {
        var uriBuilder = new UriBuilder(request.RequestUri!) { Scheme = "http", Host = "localhost", Port = 11434 };
        //对话模型
        if (request!.RequestUri!.PathAndQuery.Contains("v1/chat/completions"))
        {
            uriBuilder.Path = "/v1/chat/completions";
            request.RequestUri = uriBuilder.Uri;
        }
        //嵌入模型
        if (request!.RequestUri!.PathAndQuery.Contains("v1/embeddings"))
        {
            uriBuilder.Path = "/v1/embeddings";
            request.RequestUri = uriBuilder.Uri;
        }            
        return base.SendAsync(request, cancellationToken);
    }
}

```

这里我们使用的是国产模型`glm4:9b`,需要注意的是因为这里我们使用的是本地服务，所以需要适配一下服务的地址，通过编写`RedirectingHandler`类，并用其构造一个`HttpClient`实例传递给`Kernel`。细心的同学可能已经发现了，这里我转发的`Ollama`服务的路径也变成了和`OpenAI`服务一样的路径，但是上面我调用`Ollama`服务用的是`/api/chat`和`/api/embed`这种地址的接口。这是因为`Ollama`为了兼容`OpenAI`的标准，专门开发了一套和`OpenAI`路径和参数都一样的接口，这一点是需要注意的。当然`Ollama`暂时还没有全部兼容`OpenAI`接口的全部特征，有兴趣的同学可以去看一下[https://github.com/ollama/ollama/blob/main/docs/openai.md](https://github.com)文档地址，了解更详细的内容。


上面的服务运行起来，我们同样可以进行对话，效果如下所示
[![](https://img2024.cnblogs.com/blog/2042116/202409/2042116-20240905160706423-26310012.png)](https://github.com)
同样的你可以通过`SemanticKernel`使用嵌入模型的功能，如下所示



```
using HttpClient httpClient = new HttpClient(new RedirectingHandler());
httpClient.Timeout = TimeSpan.FromSeconds(120);

var kernelBuilder = Kernel.CreateBuilder()
    .AddOpenAITextEmbeddingGeneration(
       modelId:"nomic-embed-text:latest",
       apiKey:"ollama",
       httpClient: httpClient);
Kernel kernel = kernelBuilder.Build();
var embeddingService  = kernel.GetRequiredService();
var embeddings = await embeddingService.GenerateEmbeddingsAsync(["我觉得c#是一门不错的编程语言"]);
Console.WriteLine($"[{string.Join(",", embeddings[0].ToArray())}]");

```

这里休要注意的是`AddOpenAITextEmbeddingGeneration`方法是评估方法，将来版本有可能会删除的，所以默认的用VS使用该方法会有错误提醒，可以在`csproj`的`PropertyGroup`标签中设置一下`NoWarn`来忽略这个提醒。



```
<PropertyGroup>
   <OutputType>ExeOutputType>
   <TargetFramework>net8.0TargetFramework>
   <NoWarn>SKEXP0010;SKEXP0001NoWarn>
PropertyGroup>

```

### 总结


    本文介绍了如何通过`C#`结合`Ollama`实现本地大语言模型的部署与调用，重点演示了在`C#`应用中集成该功能的具体步骤。通过详细的安装指南与代码示例，帮助开发者快速上手。


* 首先我们介绍了`Ollama`的安装及基本设置和命令的使用。
* 然后介绍了如何通过`Ollama`调用大模型,比如使用`命令行`、`Http接口服务`、`可视乎界面`。
* 再次我们我们通过`C#`使用了`Ollama SDK`来演示了`对话模式`、`文本嵌入`、`多模态模型`如何使用，顺便说了一下相似度计算相关。
* 最后，我们展示了通过`Semantic Kernel`调用`Ollama`服务，因为`Ollama`对`OpenAI`的接口数据格式做了兼容，虽然还有部分未兼容，但是日常使用问题不大。


    通过本文希望没有了解过大模型的同学可以入门或者大概了解一下相关的基础，毕竟这是近两年或者未来几年都比较火的一个方向。即使我们不能深入的研究他，但是我们也得知道它了解它的基本原理与使用。我们为什么要持续学习，因为这些东西很多时候确实是可以给我们提供方便。接触它，了解它，才能真正的知道它可以帮助我解决什么问题。




👇欢迎扫码关注我的公众号👇
[![](https://img2020.cnblogs.com/blog/2042116/202006/2042116-20200622133425514-1420050576.png)](https://github.com)

