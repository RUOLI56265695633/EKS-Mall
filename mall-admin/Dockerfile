# 第一阶段：构建应用
FROM maven:3.8.6-openjdk-8-slim AS builder

# 设置工作目录
WORKDIR /app

# 复制各个模块的 pom.xml 并下载依赖（利用 Docker 缓存机制）
COPY pom.xml .
COPY ../mall-common/pom.xml mall-common/
COPY ../mall-mbg/pom.xml mall-mbg/
COPY ../mall-security/pom.xml mall-security/

# 离线下载所有依赖，-B 表示批处理模式
RUN mvn dependency:go-offline -B

# 复制所有模块的源代码
COPY ../mall-common/src mall-common/src
COPY ../mall-mbg/src mall-mbg/src
COPY ../mall-security/src mall-security/src
COPY src ./src

# 打包应用（跳过测试），并将最终的 jar 包重命名为 app.jar
RUN mvn clean package -DskipTests -B && \
    mv target/mall-admin-*.jar app.jar

# 第二阶段：运行应用（使用轻量级的 alpine 镜像）
FROM openjdk:8-jre-alpine

# 安装并配置时区为上海（解决容器时区偏移问题）
RUN apk add --no-cache tzdata && \
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    apk del tzdata

# 设置运行时工作目录
WORKDIR /app

# 从构建阶段复制打包好的 jar 包
COPY --from=builder /app/app.jar app.jar

# 设置 JVM 启动参数（初始堆 512M，最大堆 1024M）
ENV JAVA_OPTS="-Xms512m -Xmx1024m"

# 暴露应用端口
EXPOSE 8080

# 健康检查配置（每 30 秒检查一次，超时 3 秒，启动后 60 秒开始检查，失败 3 次判定为不健康）
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
    CMD wget --quiet --tries=1 --spider http://localhost:8080/actuator/health || exit 1

# 启动应用（支持通过环境变量动态调整 JVM 参数）
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]