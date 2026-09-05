# ================= 基础运行阶段 =================
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS base
WORKDIR /app
EXPOSE 8080

# 合并 RUN 指令减少镜像层数
RUN sed -i 's|http://archive.ubuntu.com/ubuntu|https://mirrors.aliyun.com/ubuntu|g' /etc/apt/sources.list.d/ubuntu.sources && \
    sed -i 's|http://security.ubuntu.com/ubuntu|https://mirrors.aliyun.com/ubuntu|g' /etc/apt/sources.list.d/ubuntu.sources && \
    apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/* && \
    mkdir -p /app/logs /app/wwwroot/uploads && \
    chown -R app:app /app/logs /app/wwwroot/uploads

# 明确指定用户，避免 $APP_UID 变量在某些构建环境中未定义的风险
USER app

# ================= 构建阶段 =================
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# 利用 Docker 缓存层，先还原依赖再拷贝源码
COPY ["Bedrock.WebApi/Bedrock.WebApi.csproj", "Bedrock.WebApi/"]
COPY ["Bedrock.Configuration/Bedrock.Configuration.csproj", "Bedrock.Configuration/"]
COPY ["Bedrock.Infrastructure/Bedrock.Infrastructure.csproj", "Bedrock.Infrastructure/"]
COPY ["Bedrock.Core/Bedrock.Core.csproj", "Bedrock.Core/"]
COPY ["Bedrock.Application/Bedrock.Application.csproj", "Bedrock.Application/"]
RUN dotnet restore "./Bedrock.WebApi/Bedrock.WebApi.csproj"

COPY . .
WORKDIR "/src/Bedrock.WebApi"
RUN dotnet build "./Bedrock.WebApi.csproj" -c $BUILD_CONFIGURATION -o /app/build --no-restore

# ================= 发布阶段 =================
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
ARG RUNTIME_ID=linux-x64
ARG SELF_CONTAINED=false
ARG USE_APP_HOST=false
ARG DEBUG_TYPE=none
ARG DEBUG_SYMBOLS=false

RUN dotnet publish "./Bedrock.WebApi.csproj" \
    -c $BUILD_CONFIGURATION \
    -r $RUNTIME_ID \
    -o /app/publish \
    /p:SelfContained=$SELF_CONTAINED \
    /p:UseAppHost=$USE_APP_HOST \
    /p:DebugType=$DEBUG_TYPE \
    /p:DebugSymbols=$DEBUG_SYMBOLS

# ================= 最终运行阶段 =================
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .

# 确保运行时文件权限正确
USER root
RUN chown -R app:app /app
USER app

# 添加健康检查（根据你的实际健康检查端点调整）
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD ["curl", "-sf", "http://localhost:8080/health"]

ENTRYPOINT ["dotnet", "Bedrock.WebApi.dll"]
