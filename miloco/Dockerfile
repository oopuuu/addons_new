# 直接使用你打包好的镜像作为基础
FROM ghcr.io/oopuuu/miloco-backend:latest

# 定义参数
ARG GITHUB_DOMAIN=ghrp.hacs.vip
ARG BASHIO_VERSION=v0.17.0

# ----------------------------------------------------------------
# [新增] 硬件加速环境变量默认配置
# ----------------------------------------------------------------
# MILOCO_HW_ACCEL 可选值: cpu, intel, nvidia, mac, rpi
# MILOCO_HW_DEVICE: 硬件设备路径 (Intel/AMD 核显通常是 /dev/dri/renderD128)
ENV MILOCO_HW_ACCEL=cpu \
    MILOCO_HW_DEVICE=/dev/dri/renderD128 \
    \
    # 原有环境变量
    BACKEND_PORT=28800 \
    RTSP_PORT=36674 \
    LITE_MODE=true \
    BACKEND_LOG_LEVEL=warning \
    MILOCO_SERVER_STORAGE_DIR=/config/miloco_server \
    TZ=Asia/Shanghai

# 设置时区
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 拷贝 HA Add-on 的启动脚本和配置文件
COPY rootfs /

# ----------------------------------------------------------------
# 安装依赖与硬件驱动
# ----------------------------------------------------------------
RUN set -eux; \
    chmod a+x /run.sh; \
    \
    # 替换源以加速安装 (根据你的基础镜像实际情况，Debian/Ubuntu 适用)
    sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list.d/debian.sources 2>/dev/null || \
    sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list || true; \
    \
    apt update; \
    \
    # 1. 安装 HA Add-on 运行基础工具
    apt install -y --no-install-recommends \
        bash \
        curl \
        wget \
        tar \
        jq \
        nginx \
        netcat-openbsd \
        \
    # 2. [新增] 安装硬件加速驱动 (针对 N5105/Intel/AMD)
    # intel-media-va-driver: Intel iHD driver (适用于较新的 CPU 如 N5105, 8代+)
    # i965-va-driver: Intel 旧版驱动
    # mesa-va-drivers: AMD 显卡驱动
    # libva-drm2, libva2: VAAPI 运行时库
    # vainfo: 用于调试硬件加速是否开启 (命令: vainfo)
        libva-drm2 \
        libva2 \
        i965-va-driver \
        intel-media-va-driver \
        mesa-va-drivers \
        vainfo \
        ; \
    \
    # 3. 安装 Bashio
    mkdir -p bashio; \
    wget https://${GITHUB_DOMAIN}/hassio-addons/bashio/archive/${BASHIO_VERSION}.tar.gz -O- | tar zxvf - --strip 1 -C bashio; \
    mv bashio/lib /usr/lib/bashio; \
    ln -s /usr/lib/bashio/bashio /usr/bin/bashio; \
    rm -rf bashio; \
    \
    # 4. 清理缓存减小体积
    apt-get clean; \
    rm -rf /var/lib/apt/lists/*

# 设置入口
ENTRYPOINT ["/run.sh"]

# 启动命令
CMD ["python3", "start_server.py"]

# 健康检查
HEALTHCHECK --interval=1m --start-period=30s CMD nc -zn 0.0.0.0 ${BACKEND_PORT:-8000} || exit 1
