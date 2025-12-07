# 小米Miloco

Xiaomi Local Copilot 以米家摄像机为视觉信息来源，以自研大模型为核心，打通全屋 IoT 设备。基于大模型的开发范式，让用户能够以自然语言定义家庭的各种需求和规则，实现更广泛、更具创意的智能联动。

在官方镜像的基础上修改:
- 支持arm64架构
- 默认获取音频流
- 默认获取高清视频流

现在无需micam配合就可以获取rtsp视频流，配置好miloco后访问 rtsp://{haos的ip}:{RTSP_PORT}/{相机ID}?video_quality={清晰度}即可
例如 rtsp://192.168.0.11:36674/1123456789?video_quality=2
<img src="IMG_0110.png">

