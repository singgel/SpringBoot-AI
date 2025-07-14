# SpringBoot-AI
# 后续系列文章在
https://blog.csdn.net/singgel/category_13004992.html?spm=1001.2014.3001.5482  
https://juejin.cn/column/7525700630524100617  
https://www.zhihu.com/column/c_1928107609761743771  
【AI】专栏。  

# spring-ai-mcp-demo
SpringAI MCP demo 结合通义千问大模型

## 环境要求
- JDK 17+
- Maven 3.8.6+
- npm 10.9.2+
- python3.12.3+
- 需要去申请一个自己的千问大模型key
- SpringAI 1.0.0-M5 + SpringAI 1.0.0-M6

## 模块功能
- mcp-client模块：基于SpringAI 1.0.0-M5 版本，展示了如何使用FunctionCall的方式与MCP服务端对接
- call-mcp-server模块：基于SpringAI 1.0.0-M6 版本，展示了如何使用ToolCall的方式与MCP服务端对接
- mcp-server模块：简单的MCP服务端Demo（已经去除了数据库依赖，具体功能可以自己实现）

## call-mcp-server模块
call-mcp-server模块可以充当cursor或者Claude的角色，直接调用各种开源MCP服务例如百度地图服务  
修改call-mcp-server/src/main/resources/mcp-server.json中的内容即可  
详情参考：[掘金技术社区 10分钟带你集成百度地图MCP服务](https://juejin.cn/post/7485758756913266707)

## 帮助文档
- [掘金技术社区 SpringAI-MCP技术初探](https://juejin.cn/post/7483127098352877579)



# spring-boot-ollama
SpringAI ollama demo 结合本地大模型
## ollama介绍与安装

[ollama](https://ollama.com/)作为一个工具(且开源)，让大模型的安装与运行变得非常简单。

ollama支持多种操作系统，为了方便可以直接使用Docker运行。

下载命令一行搞定：

docker

```
sudo docker pull ollama/ollama:latest
```

brew

```
brew install ollama
```

ollama上手

ollama下载好后，直接运行

#运行ollama，并指定挂载目录和映射端口

docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama

#进入ollama容器

docker exec -it ollama bash

## ollama安装大模型

#运行ollama命令pull一个大模型，这里拉取对话大模型的llama3

```
ollama pull llama3
```

ollama支持的大模型非常多，如google的gemma、facebook的llama、阿里的qwen通通都有，按需所取。

模型仓库地址为：https://ollama.com/library

大模型下载好了后，就可以使用ollama run命令运行对应的模型，并可以进行命令行的文本交互

```
ollama run llama3
```

## open-webui安装与使用

为了能获得更好的体验，可以使用开源的open-webui进行来访问离线大模型，UI界面和ChatGPT的非常像。

docker下拉取命令：

```
sudo docker pull ghcr.io/open-webui/open-webui:main
```

拉取好后直接运行：

```
docker run -d --network=host -v open-webui:/app/backend/data -e OLLAMA_BASE_URL=http://127.0.0.1:11434 --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

## spring-ai与ollama集成

前面介绍了如何使用ollama与open-webui快速搭建一个离线大模型平台，并体验了AI的相关功能。

但在实际业务场景中，前端程序往往与后端平台进行对接，与大模型的交互由后端程序来负责接入则更为合适。

站在这一维度考虑，使用Spring AI来接入大模型则是一个不错的选择。

关于Spring AI的详细介绍可参考：https://spring.io/projects/spring-ai

Spring AI is an application framework for AI engineering. Its goal is to apply to the AI domain Spring ecosystem design principles such as portability and modular design and promote using POJOs as the building blocks of an application to the AI domain.

翻译一下：Spring AI是一个AI工程框架。它的目标是将Spring生态的设计原则应用到AI领域，比如可移植性和模块化，并推广使用POJO来构建AI生态。

换句话讲：Spring AI不生产AI，只是AI的搬运工

#### pom依赖

新建一个maven项目，其pom.xml内容如下：

```
    <dependencies>
        <dependency>
            <groupId>com.vaadin</groupId>
            <artifactId>vaadin-core</artifactId>
            <version>${vaadin.version}</version>
        </dependency>
        <dependency>
            <groupId>com.vaadin</groupId>
            <artifactId>vaadin-spring-boot-starter</artifactId>
            <version>${vaadin.version}</version>
        </dependency>
        <dependency>
            <groupId>in.virit</groupId>
            <artifactId>viritin</artifactId>
            <version>${viritin.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-core</artifactId>
            <version>${spring-ai.version}</version> <!-- 使用属性变量 -->
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-ollama-spring-boot-starter</artifactId>
            <version>${spring-ai.version}</version> <!-- 使用属性变量 -->
        </dependency>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-tika-document-reader</artifactId>
            <version>${spring-ai.version}</version> <!-- 使用属性变量 -->
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
```

#### 大模型接口配置

在SpringBoot项目的配置文件中添加配置项，在application.properties中新增：

```
spring.application.name=spring-boot-ollama
server.port=8088
#配置ollama接口地址
spring.ai.ollama.base-url=http://127.0.0.1:11434/
#配置使用的ollama模型
spring.ai.ollama.chat.options.model=llama3:latest
#spring.ai.ollama.chat.options.model=qwen2.5:latest
spring.ai.ollama.chat.options.temperature=0.7
spring.ai.ollama.embedding.model=nomic-embed-text:latest
spring.ai.ollama.embedding.options.top-k=1
spring.ai.ollama.chat.options.top-k=1
```

## 帮助文档
- [掘金技术社区 大模型应用开发](https://juejin.cn/post/7525614850199011368)
- [掘金技术社区 LangChain构建RAG应用](https://juejin.cn/post/7525370788360323124)
- [掘金技术社区 LLM应用系统评估与优化](https://juejin.cn/post/7525614850199027752)
