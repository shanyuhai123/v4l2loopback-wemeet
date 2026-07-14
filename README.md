# v4l2loopback-wemeet

腾讯会议 Linux 客户端的 `v4l2loopback` 兼容分支。

- 上游项目：[v4l2loopback/v4l2loopback](https://github.com/v4l2loopback/v4l2loopback)
- 上游基线：`v0.15.4`
- 兼容分支：`wemeet-compat`
- 兼容标签：`v0.15.4-wemeet.1`

## 解决的问题

腾讯会议选择 `v4l2loopback` 虚拟摄像头后，视频设置页显示黑屏，日志持续出现：

```text
Not enough buffer on device:/dev/video10
Could not requeue buffer on device:/dev/video10
```

腾讯会议已经通过 `VIDIOC_REQBUFS` 协商 MMAP 缓冲区，但后续调用
`VIDIOC_DQBUF` / `VIDIOC_QBUF` 时会将 `v4l2_buffer.memory` 留为 `0`。
上游 `v0.15.4` 严格要求该字段为 `V4L2_MEMORY_MMAP`，因此直接返回
`EINVAL`，最终导致预览黑屏。

本分支仅在当前 opener 已经选择 `V4L2L_IO_MMAP` 时，将未指定的
`memory == 0` 归一化为 `V4L2_MEMORY_MMAP`。其他内存类型、缓冲区索引、
分配状态和流所有权检查均保持上游行为。

## 已验证环境

- Arch Linux，内核 `7.1.3-arch1-2`
- 腾讯会议 `wemeet-bin 3.26.10.401-2`
- `v4l2loopback 0.15.4`
- FFmpeg `8.1.2`
- DroidCam 手机视频流，经 FFmpeg 转为 `YUYV 640x480 @ 25fps`，写入
  `/dev/video10`

验证结果：腾讯会议 **设置 → 视频** 可以持续显示真实画面；模拟腾讯会议
传入 `memory == 0` 的最小程序也能成功执行 `DQBUF` 并抓取完整视频帧。

构建、DKMS 安装和模块参数请参考[上游文档](https://github.com/v4l2loopback/v4l2loopback#readme)。
