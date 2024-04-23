# 单独或批量把Webp动画转换为GIF，Webp静态图片转为Png
Individually or batch convert Webp animation to GIF, Webp image to Png  
WebpアニメーションをGifに、Webp静止画像をPngに個別または一括変換する

修复某些Webp动画没有无限循环动画的bug  
把不标准的GIF用ffmpeg修复为标准GIF（可用ACDSee3.2播放）

Fix the bug that some Webp animations don't have infinite loop animation.  
Repair non-standard GIF with ffmpeg to standard GIF (playable with ACDSee 3.2).

一部の Webp アニメーションが無限ループしないバグを修正。   
ffmpegで非標準GIFを標準GIF（ACDSee3.2で再生可能）に修復。

---

众所周知，Google搞出来的Webp格式难用得一逼——  
灾难性的兼容性，对动画和静态图片不做后缀名的区分，以及在老系统上无法正常打开等缺点不一而足。  
遂创立以下Python代码，以将其转换回GIF或PNG。  
The Webp format that Google came up with is hard to use as hell -    
Disastrous compatibility, no suffix distinction between animated and static images, and the inability to open it properly on older systems, to name a few drawbacks.    
The following Python code was created to convert it back to a GIF or PNG 
Googleが開発したWebpフォーマットが地獄のように使いにくいことはよく知られた事実だ。 
悲惨な互換性、アニメーション画像と静止画像の接尾辞の区別がない、古いシステムでは正しく開けない、などの欠点がある。 
次のPythonコードは、GIFまたはPNGに変換するために作成されました。

请先下载以下Windows程序，用于转换目的，使用其它平台的朋友请按照自身情况下载对应的代码包：    
Please download the following Windows program for conversion purpose, if you are using other platforms, please download the corresponding code package according to your own situation:  
他のプラットフォームを使用している場合は、各自の状況に応じて対応するコードパッケージをダウンロードしてください：  

1. [FbWebp2Gif](https://github.com/Nick-IronTower/FbWebp2Gif)
    Converts animated webp images to gif.
2. [libwebp-1.4.0-windows-x64](https://developers.google.com/speed/webp/docs/precompiled)
    `dwebp` decodes WebP back into PNG
3. [FFmpeg](https://www.gyan.dev/ffmpeg/builds/ffmpeg-git-full.7z) 
    Optional, to fix Gif code compatibility
4. [pillow 10.3.0](https://pypi.org/project/pillow/)
    Used to determine whether a Webp file is an animated or static image  

---
   
```Python
import os
import sys
from PIL import Image, ImageSequence

ROOT = r"D:\Workspace"  # Root Directory 根目录 ルートディレクトリ
ANIMATION = r"D:\FbWebp2Gif\FbWebp2Gif.exe"  # Animation Conversion Tool Path 动画转换工具路径 アニメーション変換ツールパス
STATIC = r"D:\libwebp-1.4.0-windows-x64\bin\dwebp.exe"  # Static WebP Conversion Tool Path 静态WebP转换工具路径 静的WebP変換ツールパス

def wash_gif(input):
"""
Repair non-standard Gif with ffmpeg to standard Gif (playable with ACDSee 3.2)
用 ffmpeg 修复非标准的 GIF，使其成为标准 GIF（可在 ACDSee 3.2 中播放）
ffmpegで非標準GIFを修正し、標準GIF（ACDSee 3.2で再生可能）にする
"""
    if ".gif" in input:
        output = 'temp.gif'
        # Execute ffmpeg command to repair GIF
        cmd = f'ffmpeg -i "{input}" "{output}"'
        print(cmd)
        os.system(cmd)
        # Fix the problem that GIFs can't be looped
        cmd = f'ffmpeg -y -i "{output}" -loop 0 "{output}"'
        print(cmd)
        os.system(cmd)


def is_animation(file):
"""
Estimate wether a WebP file is static or animation
估算一个 WebP 文件是静态的还是动画的
WebPファイルが静的かアニメーションかを推定する
"""
    image = Image.open(file)
    count = 0
    for frame in ImageSequence.Iterator(image):
        count += 1
    return count > 1


def main(webp_filepath):
"""
Main function to convert WebP files to GIF
主函数，用于将 WebP 文件转换为 GIF
WebPファイルをGIFに変換する主な機能
"""
    gif_filepath = webp_filepath.replace('webp', 'gif')

    if not is_animation(webp_filepath):
        # If it's a static WebP image, convert it to PNG
        # 如果是静态的 WebP 图片，将其转换为 PNG
        # 静的なWebP画像の場合は、PNGに変換します
        save_path = webp_filepath.replace('webp', 'png')
        cmd = f'{STATIC} "{webp_filepath}" -o "{save_path}"'
        os.system(cmd)
        os.remove(webp_filepath)
        sys.exit()

    # For WebP files, convert, or leave it alone if it's a GIF
    # 对于 WebP 文件，进行转换，如果是 GIF 则不进行处理
    # WebPファイルの場合、変換され、GIFの場合は処理されない
    if '.webp' in webp_filepath:
        # Convert to GIF 转换为 GIF GIFに変換する
        cmd = f"{ANIMATION} {webp_filepath}"
        os.system(cmd)

        # Delete previous WebP 删除原始的 WebP 文件 元のWebPファイルを削除する
        os.remove(webp_filepath)

    elif '.gif' in webp_filepath:
        # Copy original filename 复制原始文件名 元のファイル名をコピーする
        gif_filename = gif_filepath

        # Using FFmpeg to fix GIF 使用 FFmpeg 修复 GIF FFmpegでGIFを修正する
        wash_gif(gif_filepath)

        # Delete original File 删除原始文件 オリジナルファイルを削除する
        os.remove(gif_filepath)

        # Rename the temporary file 重命名临时文件 一時ファイルの名前を変更する
        os.rename('temp.gif', gif_filename)


if __name__ == '__main__':
    os.chdir(ROOT)
    for file in os.listdir(ROOT):    
        main(file)

```
