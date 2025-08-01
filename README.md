# realesrgan-ncnn-vulkan

> [!NOTE]
>
> 我的设备信息
>
> - 电脑：MacBook Air（2022）
> - 操作系统：macOS
> - 芯片：Apple M2
> - 内存：16GB
>
> 处理一个 40 秒/60 帧的视频，大约需要 50 分钟。

模型使用建议：

| 模型名称                  | 说明                                                                                                                                         |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `realesrgan-x4plus`       | **通用模型**，适合真实场景、人物、街景等。<br>对视频中的人像、运动细节保留效果较好。<br>如果输出的都是马赛克，可以调整参数 `-s` 将比例放大。 |
| `realesrgan-x4plus-anime` | 面向**二次元动画/漫画**风格，不适合真实人物视频。                                                                                            |
| `realesr-animevideov3`    | 更适合动漫视频（连贯性更好），**不推荐用于真人视频**。                                                                                       |

## 提升图片画质

```shell
./realesrgan-ncnn-vulkan -i [原图片路径] -o [新图片路径] -n realesrgan-x4plus -s 2
```

## 提升视频画质

### 将视频转为关键帧

```shell
# 新建关键帧目录
touch tmp_frames

# 转为关键帧
ffmpeg -i [视频路径] -qscale:v 1 -qmin 1 -qmax 1 -vsync 0 tmp_frames/frame%08d.jpg
```

### 处理每一个关键帧

```shell
# 新建处理后关键帧目录
touch out_frames

# 处理关键帧
./realesrgan-ncnn-vulkan -i tmp_frames -o out_frames -n realesr-animevideov3 -s 2 -f jpg
```

### 将处理后的关键帧合成新视频

```shell
ffmpeg -i out_frames/frame%08d.jpg -i [原视频路径] -map 0:v:0 -map 1:a:0 -c:a copy -c:v libx264 -r 23.98 -pix_fmt yuv420p [新视频路径]
```

指定帧率为 60

```shell
ffmpeg -framerate 60 -i out_frames/frame%08d.jpg -i [原视频路径] -map 0:v:0 -map 1:a:0 -c:a copy -c:v libx264 -r 60 -pix_fmt yuv420p [新视频路径]
```
