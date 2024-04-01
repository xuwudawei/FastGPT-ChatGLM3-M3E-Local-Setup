# FastGPT接入ChatGLM3和M3E向量模型 - 本地开发搭建私有化知识库教程

- Docker用来安装其他的程序
- One API程序用来管理和分发ChatGLM3模型
- ChatGLM3和M3E向量模型文件
- fastGPT程序用来作为提问和知识库管理的平台

以下为个人简化教程，详细教程建议从官网上查看

官网本地开发部署教程链接： https://doc.fastai.site/docs/development/intro/

官网Docker部署教程链接：https://doc.fastai.site/docs/development/docker/

## 个人简化教程：

创建fastgpt目录：`mkdir fastgpt`

切换到fastgpt目录：`cd fastgpt`

下载docker-compose文件：`curl -O https://raw.githubusercontent.com/labring/FastGPT/main/files/deploy/fastgpt/docker-compose.yml`

下载config文件：`curl -O https://raw.githubusercontent.com/labring/FastGPT/main/projects/app/data/config.json`

docker-compose文件github：https://github.com/labring/FastGPT/blob/main/files/deploy/fastgpt/docker-compose.yml

config文件github：https://github.com/labring/FastGPT/blob/main/projects/app/data/config.json

拉取镜像：`docker-compose pull`

在后台运行容器：`docker-compose up -d`

FastGPT的页面：http://localhost:3000/

登录用户名为root，密码为docker-compose文件里DEFAULT_ROOT_PSW，默认密码1234

### 安装数据库：

建议使用Docker Compose快速部署，安装操作请见：https://doc.fastai.site/docs/development/docker/

### 初始配置
以下文件均在 `projects/app` 路径下。

#### 环境变量

复制`.env.template`文件，在同级目录下生成一个`.env.local` 文件，修改`.env.local` 里内容才是有效的变量。变量说明见 `.env.template`

#### config 配置文件

复制 `data/config.json` 文件，生成一个 `data/config.local.json` 配置文件，具体配置参数说明，可参考 config 配置说明

### 注意以下几点：

#### docker-compose文件

1. 修改`OPENAI_BASE_URL`：http://<本地ip地址>:3002/v1		localhost记得替换为你的本地ip地址
2. 修改`MONGO_URI`, `mongodb://myusername:mypassword@<本地ip地址>:27017?authSource=admin` 	记得替换为你的本地ip地址
3. 修改`PG-URL`:`postgresql://username:password@<本地ip地址>:5432/postgres`

#### Mongo 副本集自动初始化

记得替换mongo为你的本地ip地址

`rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "<本地ip地址>:27017" }
  ]
})`

### 下载模型和运行文件

ChatGLM3模型：`git clone https://www.modelscope.cn/ZhipuAI/chatglm3-6b.git` (文件较大需时间)

ChatGLM3运行文件: `git clone https://github.com/xuwudawei/ChatGLM3`

在运行文件里: 

`cd openai_demo`

修改`Model_Path`和`Tokenizer_Path`，指向模型位置

确认port端口，默认8000。

`python api_server.py`

用docker部署m3e模型，默认CPU运行：
`docker run -d -p 6008:6008 --name=m3e-large-api registry.cn-hangzhou.aliyuncs.com/fastgpt_docker/m3e-large-api:latest`

使用GPU运行：
`docker run -d -p 6008:6008 --gpus all --name=m3e-large-api registry.cn-hangzhou.aliyuncs.com/fastgpt_docker/m3e-large-api:latest`

原镜像：
`docker run -d -p 6200:6008 --name=m3e-large-api stawky/m3e-large-api:latest`

OneAPI网站：http://localhost:3002/

OneAPI网页后，使用root用户和默认密码123456登录

chatglm3的Base URL：http://localhost:8000	localhost修改为你的本地ip地址

m3e的Base URL：http://localhost:6008		localhost记得修改为你的本地ip地址

m3e密钥：sk-aaabbbcccdddeeefffggghhhiiijjjkkk

docker-compose文件修改`OPENAI_BASE_URL`：http://<本地ip地址>:3002/v1		localhost记得替换为你的本地ip地址

docker-compose文件修改`CHAT_API_KEY`：填入从OneAPI令牌复制的key

修改config文件`llmModels`：
```json
{
  "model": "chatglm3",
  "name": "chatglm3",
  "maxContext": 16000,
  "maxResponse": 4000,
  "quoteMaxToken": 13000,
  "maxTemperature": 1.2,
  "vision": false,
  "defaultSystemChatPrompt": ""
}
```

可以在`defaultSystemChatPrompt`里设置你的系统默认提示词。

修改config文件`VectorModels`：
```json
{
  "model": "m3e",
  "name": "m3e",
  "price": 0.1,
  "defaultToken": 700,
  "maxToken": 3000
}
```

### 重新更新配置文件，依次输入命令：

1. 关闭容器：
   ```
   docker-compose down
   ```
2. 重新启动容器：
   ```
   docker-compose up -d
   ```

### 进入FastGPT网页：

- 访问：[http://localhost:3000/](http://localhost:3000/)
- 登录用户名为`root`，密码为docker-compose文件里`DEFAULT_ROOT_PSW`，默认密码`1234`。

### 网上推荐的部署视频链接：

- 观看YouTube教程：[部署视频教程](https://www.youtube.com/watch?v=yYdjBvhdYW8&t=1742s)

### 结语

有什么问题还请指正，谢谢！
