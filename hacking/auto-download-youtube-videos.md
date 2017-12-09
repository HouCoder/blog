# 树莓派自动下载 YouTube 视频

在 YouTube 上面看到有趣的视频想要下载下来保存到手机上观看，非 YouTube Red 会员，仔细琢磨了一下，用一个简单的脚本部署在树莓派上实现了这个功能。 这种方法无需客户端，在 YouTube 网站以及 YouTube 各平台的 App 下都适用。

## 1. 创建下载播放列表

按照 [YouTube 官方说明](https://support.google.com/youtube/answer/57792)创建一个播放列表，⚠️ 注意隐私选项不要设置为 Private，建议设置为 Unlisted。

## 2. 安装并配置 youtube-dl、shadowsocks

[youtube-dl](https://github.com/rg3/youtube-dl) 是一个开源的 cli 视频下载工具。

```bash
$ pip install youtube-dl shadowsocks
```

安装完成后配置 Shadowsocks 并在后台运行，建议使用 supervisord 运行 Shadowsocks。

## 3. 配置下载脚本

脚本代码：https://gist.github.com/HouCoder/453f53fedaf6b729b92b2fce369c5e41 ，保存脚本到指定的目录，注意阅读注释，根据自己的需求来更改对应的脚本参数。

这里说明下 youtube-dl 用到的参数：

`--output "${base_folder_video}${2}/${current_date}__%(uploader)s__%(title)s.%(ext)s"`

这里是下载的文件路径模版，注意保存的文件名是以日期为开头的，这样方便浏览视频文件时排序。

`--download-archive ${base_folder_archive}download_arachive_${2}`

这里会把已经下载的文件记录里面，下次运行时就不会重复下载了。

`--ignore-errors`

忽略错误，出错时不会终止，会跳过错误继续下载。

`--write-sub`

视频提供字幕时下载字幕文件。

`--format best`

会自动获取最佳格式的视频，如果不设置会下载最佳质量的视频文件，以视频 [Portrait Mode: Explained!](https://www.youtube.com/watch?v=xhybjeRciYg) 为例，使用该参数下载的视频文件大小为 52.01MiB，如果不使用下载的视频则会占用 921.40MiB 的空间！

`--proxy socks5://127.0.0.1:1080`

非大陆用户可以删除此选项。


## 配置 cron

使用 `$ crontab -e` 命令添加定时任务，我设置的每 5 分钟下载一次：

```
*/5 * * * * /home/tonni/youtube-dl/youtube-dl.sh
```

## 结语

下次看到想要保存的视频只需要将视频加入到指定的播放列表即可，树莓派会定时更新并下载播放列表的视频，简单、安全、无需第三方应用。

除此之外可以在树莓派上开启 Samba 服务，将下载的视频保存到共享目录里，手机和 iPad 上安装 nPlayer 这样就能在家里多设备同步播放了，而且可以保存到手机上外出时观看。
