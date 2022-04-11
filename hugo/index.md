# Hugo使用技巧


<!--more-->

**LoveIt** 主题在 Hugo 内置的 shortcode 的基础上提供多个扩展的 shortcode.

## 1 image {#image}

{{< version 0.2.0 changed >}}

`image` shortcode 是 [`figure` shortcode](../theme-documentation-built-in-shortcodes#figure) 的替代. `image` shortcode 可以充分利用 [lazysizes](https://github.com/aFarkas/lazysizes) 和 [lightgallery.js](https://github.com/sachinchoolur/lightgallery.js) 两个依赖库.

{{< version 0.2.10 >}} 支持[本地资源引用](../theme-documentation-content#contents-organization)的完整用法.

`image` shortcode 有以下命名参数:

- **src** *[必需]* (**第一个**位置参数)
  
  图片的 URL.

- **alt** *[可选]* (**第二个**位置参数)
  
  图片无法显示时的替代文本, 默认值是 **src** 参数的值.
  
  *支持 Markdown 或者 HTML 格式.*

- **caption** *[可选]* (**第三个**位置参数)
  
  图片标题.
  
  *支持 Markdown 或者 HTML 格式.*

- **title** *[可选]*
  
  当悬停在图片上会显示的提示.

- **class** *[可选]*
  
  HTML `figure` 标签的 `class` 属性.

- **src_s** *[可选]*
  
  图片缩略图的 URL, 用在画廊模式中, 默认值是 **src** 参数的值.

- **src_l** *[可选]*
  
  高清图片的 URL, 用在画廊模式中, 默认值是 **src** 参数的值.

- **height** *[可选]*
  
  图片的 `height` 属性.

- **width** *[可选]*
  
  图片的 `width` 属性.

- **linked** *[可选]*
  
  图片是否需要被链接, 默认值是 `true`.

- **rel** *[可选]*
  
  HTML `a` 标签 的 `rel` 补充属性, 仅在 **linked** 属性设置成 `true` 时有效.

一个 `image` 示例:

```markdown
{{</* image src="/images/avatar.png" caption="Lighthouse (`image`)"  */>}}
```

呈现的输出效果如下:

{{< image src="/images/avatar.png" caption="Lighthouse (`image`)" src_s="/images/lighthouse-small.jpg" src_l="/images/lighthouse-large.jpg" >}}

## 2 admonition

`admonition` shortcode 支持 **12** 种 帮助你在页面中插入提示的横幅.

*支持 Markdown 或者 HTML 格式.*

{{< admonition >}}
一个 **注意** 横幅
{{< /admonition >}}

{{< admonition abstract >}}
一个 **摘要** 横幅
{{< /admonition >}}

{{< admonition info >}}
一个 **信息** 横幅
{{< /admonition >}}

{{< admonition tip >}}
一个 **技巧** 横幅
{{< /admonition >}}

{{< admonition success >}}
一个 **成功** 横幅
{{< /admonition >}}

{{< admonition question >}}
一个 **问题** 横幅
{{< /admonition >}}

{{< admonition warning >}}
一个 **警告** 横幅
{{< /admonition >}}

{{< admonition failure >}}
一个 **失败** 横幅
{{< /admonition >}}

{{< admonition danger >}}
一个 **危险** 横幅
{{< /admonition >}}

{{< admonition bug >}}
一个 **Bug** 横幅
{{< /admonition >}}

{{< admonition example >}}
一个 **示例** 横幅
{{< /admonition >}}

{{< admonition quote >}}
一个 **引用** 横幅
{{< /admonition >}}

`admonition` shortcode 有以下命名参数:

- **type** *[必需]* (**第一个**位置参数)
  
  `admonition` 横幅的类型, 默认值是 `note`.

- **title** *[可选]* (**第二个**位置参数)
  
  `admonition` 横幅的标题, 默认值是 **type** 参数的值.

- **open** *[可选]* (**第三个**位置参数) {{< version 0.2.0 changed >}}
  
  横幅内容是否默认展开, 默认值是 `true`.

一个 `admonition` 示例:

```markdown
{{</* admonition type=tip title="This is a tip" open=false */>}}
一个 **技巧** 横幅
{{</* /admonition */>}}
或者
{{</* admonition tip "This is a tip" false */>}}
一个 **技巧** 横幅
{{</* /admonition */>}}
```

## 3 music

`music` shortcode 基于 [APlayer](https://github.com/MoePlayer/APlayer) 和 [MetingJS](https://github.com/metowolf/MetingJS) 提供了一个内嵌的响应式音乐播放器.

有三种方式使用 `music` shortcode.

### 8.1 自定义音乐 URL {#custom-music-url}

{{< version 0.2.10 >}} 支持[本地资源引用](../theme-documentation-content#contents-organization)的完整用法.

`music` shortcode 有以下命名参数来使用自定义音乐 URL:

- **server** *[必需]*
  
  音乐的链接.

- **type** *[可选]*
  
  音乐的名称.

- **artist** *[可选]*
  
  音乐的创作者.

- **cover** *[可选]*
  
  音乐的封面链接.

一个使用自定义音乐 URL 的 `music` 示例:

```markdown
{{</* music url="/music/Wavelength.mp3" name=Wavelength artist=oldmanyoung cover="/images/Wavelength.jpg" */>}}
```

呈现的输出效果如下:

{{< music url="/music/Wavelength.mp3" name=Wavelength artist=oldmanyoung cover="/images/Wavelength.jpg" >}}

### 8.2 音乐平台 URL 的自动识别 {#automatic-identification}

`music` shortcode 有一个命名参数来使用音乐平台 URL 的自动识别:

- **auto** *[必需]]* (**第一个**位置参数)
  
  用来自动识别的音乐平台 URL, 支持 `netease`, `tencent` 和 `xiami` 平台.

一个使用音乐平台 URL 的自动识别的 `music` 示例:

```markdown
{{</* music auto="https://music.163.com/#/playlist?id=60198" */>}}
或者
{{</* music "https://music.163.com/#/playlist?id=60198" */>}}
```

呈现的输出效果如下:

### 8.3 自定义音乐平台, 类型和 ID {#custom-server}

`music` shortcode 有以下命名参数来使用自定义音乐平台:

- **server** *[必需]* (**第一个**位置参数)
  
  [`netease`, `tencent`, `kugou`, `xiami`, `baidu`]
  
  音乐平台.

- **type** *[必需]* (**第二个**位置参数)
  
  [`song`, `playlist`, `album`, `search`, `artist`]
  
  音乐类型.

- **id** *[必需]* (**第三个**位置参数)
  
  歌曲 ID, 或者播放列表 ID, 或者专辑 ID, 或者搜索关键词, 或者创作者 ID.

一个使用自定义音乐平台的 `music` 示例:

```markdown
{{</* music server="netease" type="song" id="1868553" */>}}
或者
{{</* music netease song 1868553 */>}}
```

呈现的输出效果如下:

{{< music netease song 1868553 >}}

### 8.4 其它参数 {#other-parameters}

`music` shortcode 有一些可以应用于以上三种方式的其它命名参数:

- **theme** *[可选]*
  
  {{< version 0.2.0 changed >}} 音乐播放器的主题色, 默认值是 `#448aff`.

- **fixed** *[可选]*
  
  是否开启固定模式, 默认值是 `false`.

- **mini** *[可选]*
  
  是否开启迷你模式, 默认值是 `false`.

- **autoplay** *[可选]*
  
  是否自动播放音乐, 默认值是 `false`.

- **volume** *[可选]*
  
  第一次打开播放器时的默认音量, 会被保存在浏览器缓存中, 默认值是 `0.7`.

- **mutex** *[可选]*
  
  是否自动暂停其它播放器, 默认值是 `true`.

`music` shortcode 还有一些只适用于音乐列表方式的其它命名参数:

- **loop** *[可选]*
  
  [`all`, `one`, `none`]
  
  音乐列表的循环模式, 默认值是 `none`.

- **order** *[可选]*
  
  [`list`, `random`]
  
  音乐列表的播放顺序, 默认值是 `list`.

- **list-folded** *[可选]*
  
  初次打开的时候音乐列表是否折叠, 默认值是 `false`.

- **list-max-height** *[可选]*
  
  音乐列表的最大高度, 默认值是 `340px`.

## 4 bilibili

{{< version 0.2.0 changed >}}

`bilibili` shortcode 提供了一个内嵌的用来播放 bilibili 视频的响应式播放器.

如果视频只有一个部分, 则仅需要视频的 BV `id`, 例如:

```code
https://www.bilibili.com/video/BV1Sx411T7QQ
```

一个 `bilibili` 示例:

```markdown
{{</* bilibili BV1Sx411T7QQ */>}}
或者
{{</* bilibili id=BV1Sx411T7QQ */>}}
```

呈现的输出效果如下:

{{< bilibili id=BV1Sx411T7QQ >}}

如果视频包含多个部分, 则除了视频的 BV `id` 之外, 还需要 `p`, 默认值为 `1`, 例如:

```code
https://www.bilibili.com/video/BV1TJ411C7An?p=3
```

一个带有 `p` 参数的 `bilibili` 示例:

```markdown
{{</* bilibili BV1TJ411C7An 3 */>}}
或者
{{</* bilibili id=BV1TJ411C7An p=3 */>}}
```

呈现的输出效果如下:

{{< bilibili id=BV1TJ411C7An p=3 >}}

