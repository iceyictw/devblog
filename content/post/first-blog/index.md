---
title: 使用Hugo架設個人Blog
date: 2024-03-20T03:41:31+08:00
draft: false
tags: ["frontend", "code", "learning"]
---

# 前言
**升上大學**之後，開始覺得想要紀錄一下自己學習的軌跡，另一方面也可以將習得的知識寫成文章，日後便可以自己回頭複習檢視，
甚至讓其他和我一樣遇到相同瓶頸的人得到解決方法。再者，撰寫自己的Blog大概也算是某種品牌經營吧？
諸如此類，現在這個網站才會誕生啦！

# Framework選擇
到2024年3月為止，比較主流的靜態網頁生成框架有：
- Jekyll
- Hexo
- Hugo
- Pelican

我最後選擇使用<mark>Hugo</mark>來架設這個Blog，主要是它在Github上面的Stars蠻多的，而且很多人都因它在速度上的優勢推薦它。

{{<figure src="hugo-logo.svg" attr="Logo of [Hugo](https://gohugo.io/)" align="center">}}

# Theme選擇
我選擇使用[PaperMod](https://github.com/adityatelange/hugo-PaperMod)這個Theme，我個人偏好這種簡潔的版面設計，
而且調整起來相當簡單，不需要花費太多心力就能了解它提供的[Features](https://github.com/adityatelange/hugo-PaperMod/wiki/Features)。

# 設定Markup Parser
Hugo預設使用[Goldmark](https://github.com/yuin/goldmark/)這個Parser來將以Markdown寫成的文章在生成網頁時轉換為適當的html。
其中我想修改的是這一項：

```yaml
# config/_default/markup.yaml
goldmark:
  renderer:
    unsafe: true
```

當unsafe選項改為true時，markdown中的html標記會直接被保留，才能使用像是`<mark>`之類的html tag。

# $\LaTeX$
作為一個devblog，需要用到數學式子的日子肯定不會少，就算沒有用到也沒關係，我仍然會加入$\LaTeX$的功能。

這邊我使用[$\KaTeX$](https://katex.org/docs/autorender.html#usage)這個Library來達成，將一些script放在我們的html head中，它會自動尋找內文中所有使用delimiter的部分，並將其替換成適當的元素。
KaTeX使用的js function可以自行傳入想要使用的delimiter，這樣我們就可以根據自己的偏好進行設定了。

```html
<!-- layouts/partials/entend_head.html -->

{{ if or .Params.math .Site.Params.math }}
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.12.0/dist/katex.min.css" integrity="sha384-AfEj0r4/OFrOo5t7NnNe46zW/tFgW6x/bCJG8FqQCEo3+Aro6EYUG4+cU+KJWu/X" crossorigin="anonymous">
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.12.0/dist/katex.min.js" integrity="sha384-g7c+Jr9ZivxKLnZTDUhnkOnsh30B4H0rpLUpJ4jAIKs4fnJI+sEnkvrMWph2EDg4" crossorigin="anonymous"></script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.12.0/dist/contrib/auto-render.min.js" integrity="sha384-mll67QQFJfxn0IYznZYonOWZ644AWYC+Pt2cHqMaRhXVrursRwvLnLaebdGIlYNa" crossorigin="anonymous"></script>
<script>
    document.addEventListener("DOMContentLoaded", function() {
        renderMathInElement(document.body, {
          // customised options
          // • auto-render specific keys, e.g.:
          delimiters: [
              {left: '$$', right: '$$', display: true},
              {left: '$', right: '$', display: false},
              {left: '\\(', right: '\\)', display: false},
              {left: '\\[', right: '\\]', display: true}
          ],
          // • rendering keys, e.g.:
          throwOnError : false
        });
    });
</script>
{{ end }}
```

將這段html放置在`layouts/partials/entend_head.html`中，PaperMod會引用這個partial，將它嵌入在所有生成網頁的head中，我們就可以盡情使用$\LaTeX$了：
$$
e^{i\pi} + 1 = 0
$$

# 設定字體樣式
經過一番挑選之後，中文字體我使用<mark>Noto Sans TC</mark>，英文字體則使用<mark>Ubuntu</mark>，兩者都可以在[Google Fonts](https://fonts.google.com/)上找到，
這邊一樣透過google fonts提供的CDN，將對應的代碼放入html header中即可。

```html
<!-- layouts/partials/entend_head.html -->

<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@100..900&family=Roboto:ital,wght@0,100;0,300;0,400;0,500;0,700;0,900;1,100;1,300;1,400;1,500;1,700;1,900&family=Ubuntu:ital,wght@0,300;0,400;0,500;0,700;1,300;1,400;1,500;1,700&display=swap" rel="stylesheet">
```

然後再透過css設定樣式。除了字體之外，我也加入了文內超連結的hover特效。
```css
body {
  font-family: 'Ubuntu', 'Noto Sans TC', '微軟正黑體', sans-serif;
}

mark {
  background-color: #edff4c;
}

/* hyperlink hover transition */
.toc a, .post-content a {
  transition: all .15s ease-out;
}
.toc a:hover, .post-content a:hover {
  background-color: white;
  color: black;
  transition: all .15s ease-out;
}
```

# Google Analytics & Google Search Console
這個兩項服務是我參考學長的Blog時得知的，所以我也試著幫我的Blog完成設定。
設定Google Search Console後，Blog就可以出現在Chrome的搜尋引擎當中了，而Google Analytics則能蒐集網站拜訪者的相關資訊，簡單來說就有點像是數據後台啦。

# 後記
從我開始看Hugo Docs到寫完這篇文章為止，過了大概五天的時間。其實Hugo的Documentation我覺得有點小凌亂，參考了不少教學影片跟網路文章才把整體架構搞清楚。
之後大概會連載一些在大學端的修課心得，以及課餘時間摸索的東西。