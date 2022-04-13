---
title: "ブロウちゃんをデスクトップにわいわいさせる(moviepy, ffmpeg, Übersicht)"
date: 2020-12-29T21:20:02+09:00
tags: ["other"]
---

[弊高専アドカレ](https://qiita.com/advent-calendar/2021/snct)埋めます

## 背景

SteamのセールでMuse Dash買いました. ブロウちゃん(アイドル)かわいすぎるなということで, ブロウちゃんをデスクトップでいつでも見れるようにしよう, という企画です.

元素材はキャラ選択画面だとUIが被って見切れるのでYoutubeから拾ってきた.

## 透過

moviepyでframeに分割していい感じにフィルタ処理, アルファチャンネルを追加してpngに叩き込む.

```py
from moviepy.editor import *
import numpy as np
import cv2

clip = VideoFileClip("idle.mp4")

loop_clip = VideoFileClip("idolloop.mp4")
loop_frames = loop_clip.iter_frames()
max_cnt = 0
for frame in loop_frames:
    max_cnt += 1
print(max_cnt)

clip = clip.subclip(2.0, 6.0)
print(clip.fps)
frames = clip.iter_frames()

new_frames = []
cnt = 0
for frame in frames:
    f = np.array(frame)
    #fb = [tuple(a) for a in f[:,1800,:]]
    # f = np.where(f == [93, 0, 82], [0, 0, 255], f)
    fil = np.sum(np.abs(f - [93, 0, 82]), axis=2) < 11
    (h, w, _) = f.shape
    res = np.zeros((h, w, 4))
    # BGR <- RGB
    res[:,:,2] = np.where(fil, 0, f[:,:,0])
    res[:,:,1] = np.where(fil, 0, f[:,:,1])
    res[:,:,0] = np.where(fil, 0, f[:,:,2])
    res[:,:,3] = np.where(fil, 0, 255)
    print(cnt)
    cv2.imwrite("%03d_idle.png" % cnt, res)
    cnt = cnt + 1
    if cnt == max_cnt:
        break
    #ff = [tuple(a) for a in f[:,1880,:]]
```

## 透過動画

[How to use transparent videos on the web in 2021](https://rotato.app/blog/transparent-videos-for-the-web)によると, 

- ChromeではVP9コーディックのwebm形式
- SafariではHEVCコーディックのmov形式

なら透過動画が扱えるらしい. ffmpegを使ってpngの列をwebm, movに変換してみる.

### Chrome

こっちは簡単, コーディックに`libvpx-vp9`を指定すればOK

```
ffmpeg -framerate 60 -f image2 -i %03d_idle.png -c:v libvpx-vp9 output.webm
```

### Safari

こっちがマジで面倒臭い. コーディックに`hevc`を指定するとデフォルトでは`libx265`が選択される. 詳細を`ffmpeg -h encoder=hevc`で表示すると
```
    Supported pixel formats: yuv420p yuvj420p yuv422p yuvj422p yuv444p yuvj444p gbrp yuv420p10le yuv422p10le yuv444p10le gbrp10le yuv420p12le yuv422p12le yuv444p12le gbrp12le gray gray10le gray121e
```

`yuva`のような`a`が含まれていないとアルファチャンネルを加味してエンコードしてくれないらしい.

`hevc_videotoolbox`を用いれば`bgra`が含まれていてOK

```
ffmpeg -framerate 60 -i %03d_idle.png -c:v hevc_videotoolbox -allow_sw 1 -alpha_quality 1 -tag:v hvc1 output.mov
```

## Übersicht

[Übersicht](http://tracesof.net/uebersicht/). これすごい. Reactの要素とか投げられる.

[paulbhartzog/ubersicht-video-widget](https://github.com/paulbhartzog/ubersicht-video-widget)を参考にして, 動画を再生させる. 以下のcoffee scriptをWidget Folderに含めると動く. output.movは, Widget Folderに含めればOK.

```coffee
refreshFrequency: false

render: ->"""
  <video width="240" loop autoplay playsinline>
    <source src="output.mov" type="video/mp4">
    Your browser does not support the video tag.
  </video>
"""

style: """
  top: calc(100% - 135px)
  left: calc(50% - 110px)
  border: 0px solid #fff
  z-index: 1000
  background-color: rgba(255, 255, 255, 0)
  /* kluge to erase space at bottom of video inside container */
  video
    margin-bottom: 0px
"""
```

## 動いてる感じ

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">MuseDashのブロウちゃんわいわいさせてたら時間溶けてた <a href="https://t.co/oA7ny2iAp9">pic.twitter.com/oA7ny2iAp9</a></p>&mdash; Niuez (@xiuez) <a href="https://twitter.com/xiuez/status/1475104564651368448?ref_src=twsrc%5Etfw">December 26, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## 〆

かわいい
