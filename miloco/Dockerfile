# 直接使用你打包好的镜像作为基础
FROM ghcr.io/oopuuu/miloco-backend:latest

# 定义参数 (保留原逻辑中的参数，防止构建脚本报错，虽然有些可能用不到了)
ARG GITHUB_DOMAIN=ghrp.hacs.vip
ARG BASHIO_VERSION=v0.17.0

# 环境变量设置 (根据你的镜像实际情况调整，这里保留了原有的路径配置)
# 注意：确保你的镜像里这些路径是有效的，或者在这里覆盖它们
ENV BACKEND_PORT=28800 \
    RTSP_PORT=36674
    BACKEND_LOG_LEVEL=warning \
    MILOCO_SERVER_STORAGE_DIR=/config/miloco_server \
    TZ=Asia/Shanghai

# 设置时区 (如果你的基础镜像没设，这里设一下比较保险)
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# ----------------------------------------------------------------
# 下面是 Home Assistant Add-on 必须的“胶水层”
# 你的基础镜像可能没有 bashio 和 nginx，需要装上
# ----------------------------------------------------------------

# 拷贝 HA Add-on 的启动脚本和配置文件
# 假设你的目录结构里有 rootfs 文件夹包含 run.sh 等
COPY rootfs /

RUN set -eux; \
    chmod a+x /run.sh; \
    \
    # 替换源以加速安装 (假设你的镜像是基于 Debian 的)
    sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list.d/debian.sources; \
    apt update; \
    \
    # 安装 HA Add-on 运行所需的工具
    # nginx: 通常用于 HA Add-on 的 Ingress 访问
    # netcat-openbsd: 用于健康检查
    # jq, curl, bash: bashio 依赖
    apt install -y bash curl wget tar jq nginx netcat-openbsd; \
    \
    # 安装 Bashio (HA Add-on 的核心辅助库)
    mkdir -p bashio; \
    wget https://${GITHUB_DOMAIN}/hassio-addons/bashio/archive/${BASHIO_VERSION}.tar.gz -O- | tar zxvf - --strip 1 -C bashio; \
    mv bashio/lib /usr/lib/bashio; \
    ln -s /usr/lib/bashio/bashio /usr/bin/bashio; \
    rm -rf bashio; \
    \
    # 清理缓存减小体积
    rm -rf /var/lib/apt/lists/*

# 设置入口
# 这是一个关键点：原 Dockerfile 用 /run.sh 启动
# 你的镜像可能有自己的 ENTRYPOINT，这里需要覆盖它以适应 HA Add-on 的启动逻辑
ENTRYPOINT ["/run.sh"]

# 这里的 CMD 是传给 /run.sh 的参数，最终通常会执行 python 启动命令
CMD ["python3", "start_server.py"]

# 健康检查
HEALTHCHECK --interval=1m --start-period=30s CMD nc -zn 0.0.0.0 ${BACKEND_PORT:-8000} || exit 1
