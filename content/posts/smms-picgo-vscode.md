---
title: "个人博客(一): SMMS + PICGO + VSCODE"           # 文章标题
description : "搭建基于SMMS图床，配合使用图片上传工具PICGO，以及VSCODE作为markdown编辑器撰写md文章"    # 文章描述信息
date: 2023-05-13            # 文章编写日期
lastmod: 2023-05-12         # 文章修改日期
tags : [                    # 文章所属标签
    "Blog"
]
featuredImage: "https://s2.loli.net/2023/05/14/SObmMxkNvaGrDRQ.png"
featuredImagePreview: "https://s2.loli.net/2023/05/14/SObmMxkNvaGrDRQ.png"
draft: false
---
<!--more-->
## SMMS
### 1. 注册  
https://smms.app
### 2. 生成token  
![](https://s2.loli.net/2023/05/13/uknzUDqFbZcSTsv.png)

## PicGo
### 1. 下载  
https://github.com/Molunerfinn/PicGo
<table>
<thead>
<tr>
<th>下载源</th>
<th>地址/安装方式</th>
<th>平台</th>
<th>备注</th>
</tr>
</thead>
<tbody>
<tr>
<td>GitHub Release</td>
<td><a href="https://github.com/Molunerfinn/PicGo/releases">https://github.com/Molunerfinn/PicGo/releases</a></td>
<td>All</td>
<td>国内下载速度可能会慢</td>
</tr>
<tr>
<td><a href="https://cloud.tencent.com/product/cos" rel="nofollow">腾讯云COS</a></td>
<td><a href="https://github.com/Molunerfinn/PicGo/releases">https://github.com/Molunerfinn/PicGo/releases</a> 附在更新日志结尾</td>
<td>All</td>
<td>感谢 <a href="https://cloud.tencent.com/product/cos" rel="nofollow">腾讯云COS</a> 提供的赞助支持</td>
</tr>
<tr>
<td><a href="https://mirrors.sdu.edu.cn/" rel="nofollow">山东大学镜像站</a></td>
<td><a href="https://mirrors.sdu.edu.cn/github-release/Molunerfinn_PicGo" rel="nofollow">https://mirrors.sdu.edu.cn/github-release/Molunerfinn_PicGo</a></td>
<td>All</td>
<td>感谢 <a href="https://mirrors.sdu.edu.cn/" rel="nofollow">山东大学镜像站</a> 提供的镜像支持</td>
</tr>
<tr>
<td><a href="https://scoop.sh/" rel="nofollow">Scoop</a></td>
<td><code>scoop bucket add helbing https://github.com/helbing/scoop-bucket</code> &amp; <code>scoop install picgo</code></td>
<td>Windows</td>
<td>感谢 @helbing 的贡献</td>
</tr>
<tr>
<td><a href="https://chocolatey.org/" rel="nofollow">Chocolatey</a></td>
<td><code>choco install picgo</code></td>
<td>Windows</td>
<td>感谢 @iYato 的贡献</td>
</tr>
<tr>
<td><a href="https://brew.sh/" rel="nofollow">Homebrew</a></td>
<td><code>brew install picgo --cask</code></td>
<td>macOS</td>
<td>感谢 @womeimingzi11 的贡献</td>
</tr>
<tr>
<td><a href="https://aur.archlinux.org/packages/yay" rel="nofollow">AUR</a></td>
<td><code>yay -S picgo-appimage</code></td>
<td>Arch-Linux</td>
<td>感谢 @houbaron 的贡献</td>
</tr>
</tbody>
</table>

### 2. 设置SMMS  
设置SMMS的token，以及备用上传域名为`smms.app`，中国区必须设置。  
![](https://s2.loli.net/2023/05/13/g3Dqwsx4khpfWHr.png)  
设置server  
![](https://s2.loli.net/2023/05/13/MkyXxG6qECoKNTO.png)  
可选设置，设置为时间戳命名  
![](https://s2.loli.net/2023/05/13/rQtj7zfX4HsndgV.png)

## VSCode
### 下载插件  
![](https://s2.loli.net/2023/05/13/JgzvPG6YTf9xVBq.png)  
VSCODE 虽然有PICGO的插件，但是我一直得到这个错误 `Error: read ECONNRESET`，我猜测是因为插件默认的请求域名是`sm.ms`，但是因为中国区需要设置为`smms.app`，而插件暂时没有更新到这个特性，所以失败了。我在Github上提交了[issue](https://github.com/PicGo/vs-picgo/issues/143)，再等一段时间看看能否支持。

