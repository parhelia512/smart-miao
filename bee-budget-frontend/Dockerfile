# ============ 阶段1: 构建 ============
FROM node:20-alpine AS builder

WORKDIR /app

# 先复制依赖文件，利用 Docker 缓存层
COPY package.json package-lock.json ./
RUN npm ci --registry=https://registry.npmmirror.com

# 复制源码并构建
COPY . .
RUN npm run build:prod

# ============ 阶段2: 运行 ============
FROM nginx:1.27-alpine

# 移除 nginx 默认配置，避免与我们的模板冲突
RUN rm -f /etc/nginx/conf.d/default.conf

# 复制 nginx 配置模板（Docker 启动时会自动用 envsubst 替换环境变量）
COPY nginx.conf /etc/nginx/templates/default.conf.template

# 复制构建产物
COPY --from=builder /app/dist /usr/share/nginx/html/frontend

# 环境变量默认值（可在 docker-compose 或 docker run 时覆盖）
# BACKEND_URL: 后端 API 地址
#ENV BACKEND_URL=http://backend:8080
ENV BACKEND_URL=http://192.168.0.88:10003

# 关键：只替换 BACKEND_URL，防止 envsubst 误替换 nginx 内置变量（$host, $uri 等）
ENV envsubst='${BACKEND_URL}'

EXPOSE 8000

CMD ["nginx", "-g", "daemon off;"]
