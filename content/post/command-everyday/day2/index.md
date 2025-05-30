---
title: 一天一指令 Day 2 - htop
date: 2025-05-30T20:35:20+08:00
draft: false
summary: "關於 htop 的便捷小技巧"
tags: ["code", "learning", "linux"]
---

今天要來介紹的是`htop`。`htop`可以用於監控主機資源和 process 並進行一定程度的管理，例如可以向 process 發送 signal、改變 nice value、設定 CPU Affinity 等等功能。

這個工具可以說是`top`的進化版。`top`隨 Linux distro 預先安裝，但`htop`相較`top`有更良好的介面、功能及可自訂性，支援滑鼠操作，因此大部分機器上也會安裝`htop`。`htop`有使用`ncurses`這個 library 來編寫，因此它提供了相當完整的 TUI 介面。

# 主要介面

輸入`htop`之後，會進入到主頁面：

{{<figure src="htop.png" attr="htop 主要的介面" align="center">}}

先來介紹這個頁面上有什麼東西。最上面的區域稱為 <mark>Meters</mark> ，顯示主機資源的使用狀況，例如 CPU 使用百分比、Memory & Swap 使用量、Task 跟 Thread 的數量、平均 CPU 負載量 (過去1min、過去5mins、過去15mins)。

下面的大視窗是一個 <mark>Screen</mark>，可以使用`Tab`來切換各個 Screen，例如圖中有兩個 Screen：main 和 I/O。

每個 Screen 都會有一系列的 <mark>Columns</mark>，`htop`會找出主機上的所有 process/thread，並且列在 Screen 中，而這些 Columns 則決定了要顯示關於 process/thread 的哪些資訊。

Meters 上面要顯示的資訊，以及 Screen 顯示的 Column，都可以自行做設定，只要按`F2`進行 Setup 即可。操作方法直觀，因此這邊不多贅述。

# 快捷鍵與你會想知道的功能
`htop`的優秀之處在於它非常容易上手。頁面最下方大部分時間都會顯示可以使用的按鍵以及對應的操作。但為了方便，我們這邊介紹幾個可以幫助提升效率的快捷鍵。

## h for help: 顯示幫助頁面
在主頁按下`h`可以顯示幫助頁面，功能跟`F1`一樣。幫助頁面有彙整了精簡的有用資訊，例如 CPU usage / Memory usage 的條狀圖中各個顏色代表什麼樣的使用、process state 的字母縮寫分別代表什麼、各快捷鍵的功能等。

{{<figure src="htop-help.png" attr="htop 幫助頁面" align="center">}}

## t for tree: 將 process/thread 以樹狀層級表示
在主頁按下`t`便可以讓 Command 欄位的內容以 process parent/child 或是 process / thread 以他們的父子關係排列並顯示出來。可以進一步用`-`來摺疊。

## 讓 thread 以不同顏色顯示
實際上在`htop`中，每一筆 record 其實是代表一個 thread，而非單一的 process。我在這裡推薦把 `Setup > Display Options > Display threads in a different color`打開，這樣如果是一個某 process 的子 thread 的話，他的 Command 就會以<span style="color: green;">綠色</span>顯示，而如果是 process 本身 (或者也可以叫他 main thread)，則會維持白色。

## H for Hide: 隱藏 user threads
有時候 threads 數量太多了，而且我們其實比較需要 process 本身相關資訊的時候，可以按`H` (大寫，也就是`shift + h`)，來隱藏 threads。

## u for user: 顯示特定 uesr 的 process
當你想只顯示特定使用者的 process 時，按下`u`便會在左側顯示一個選單，讓你選擇要顯示哪一個使用者。如果使用者太多，你可以在按下`u`之後直接用鍵盤輸入 username，`htop`會幫你跳到符合的選項。

## 更快速地 sorting
我們知道`F6`的 SortBy 功能可以選擇一個 Column 並根據那個 Column 進行排序 (或是用滑鼠點 Column 項目也可以)，但如果我們想要快速地用鍵盤做到這件事呢？
某幾個常用項目中，`htop`其實有提供快捷鍵來讓我們排序：
- `N` (`shift + n`): 以 PID 進行排序
- `M` (`shift + m`): 以 MEM% (memory usage) 進行排序
- `P` (`shift + p`): 以 CPU% (processor usage) 進行排序
- `T` (`shift + t`): 以 TIME 進行排序

另外，`I` (`shift + i`) 可以改變排序的升降。

## F for Focus: 跟隨一個 process
將游標移動到你想關切的 process 上，按下`F` (`shift + f`) 之後，該 process 會變為黃色底，這時候`htop`會保持這個 process 持續地出現在畫面中，直到你移動游標，取消 focus 狀態。

## 對多個 process 同時進行操作
我們可以對 process 發送 signal，或是調整 nice value，但如果我想一次對很多 process 做一樣的操作該怎麼辦？

使用 `space`，你會發現游標當前指向的 process 的文字會變為<span style="color: yellow;">黃色</span>，這代表這個 process 被 "tag" 了。使用`c`的話，則可以 tag 該 process 以及它所有的 children。

tag 完畢後，進行的操作便會全部套用在選擇的 processes 上。若要取消，則使用`U`進行 Untag。

{{<figure src="htop-tagging.png" attr="tagging 完之後的樣子" align="center">}}

## s for strace, l for lsof
`s`可以利用`strace`查看 process 的 live syscalls；`l`可以裡用`lsof`查看 process 的 open files。

# 後記
在寫這邊的時候我突然想到：「如果`htop`的每一個 record 是代表一個 thread，為什麼屬於同一個 process 的 thread 卻有不同的 PID 呢？」這個問題的答案，簡單來說，是因為在 Linux 中，PID 實際上是用來 identify `task_struct`結構的：每一個`task_struct`都有自己獨一無二的 PID。而 TPID (Thread Group ID) 其實才是真正意義上的 "Process ID"。在 Linux 中，main thread 的 TPID 會與它的 PID 相同，而它所產生的其他 threads，則會使用相同的 TPID，但由於不同的 thread 實際上就是不同的`task_struct`，因此就會有不同的PID了。

關於這些 "___ID"，有機會之後可能可以另外寫一篇來了解了解。