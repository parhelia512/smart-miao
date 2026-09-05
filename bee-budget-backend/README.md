# BedrockWebapi

## 快速开始

1. 安装好必须的运行环境
2. 运行根目录下的`init.sql`文件，创建数据库、测试数据
3. F5 运行调试

## Docker Compose 调试、部署

1. 默认应该直接就是 Docker Compose 方式启动，按F5即可运行，如果不是就切换一下 “启动项目”

2. 部署时使用 docker-compose.prod.yml 文件，有详细说明

## 单 Dockerfile 方式启动

1. 想要修改镜像名称，需要修改 `Bedrock.WebApi` 项目文件里的 `ContainerRepository` 选项，否则默认镜像名称会一直为 `bedrockwebapi`。其他选项参考官方文档 [《容器项目的MSBuild属性》](https://learn.microsoft.com/zh-cn/visualstudio/containers/container-msbuild-properties?view=visualstudio)

```
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
  <ContainerRepository>bedrockwebapi-test</ContainerRepository>
    ......
```

2. 调试时默认镜像标签是`dev`，可以修改 `MSBuild-ContainerImageTag` 项目。使用VS自带的“发布”到 Docker 容器注册表时，记得修改镜像标记，否则默认一直是 `latest`
