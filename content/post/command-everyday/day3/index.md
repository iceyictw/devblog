---
title: 一天一指令 Day 3 - dd
date: 2025-06-11T22:57:47+08:00
draft: false
summary: "dd 這麼方便的小工具你一定要會！"
tags: ["code", "learning", "linux"]
---

# 基本介紹

今天要學習的指令是`dd`。這個工具的 man page 相當簡短，只有152行內容，但它是一個小而精美的工具：從一個檔案讀取內容，然後寫入到另一個檔案，byte to byte，就是這樣一個很 low-level 的操作，筆者推薦各位一定要親自閱讀一下 man page。它的基本用法是：

```
dd if=/path/to/input_file of=/path/to/output_file
```

這個指令讓`dd`從 `/path/to/input_file`讀出 bytes，然後寫入到`/path/to/output_file`。這兩個選項`if`跟`of`是最常被指定的選項，若不指定，則`dd`會分別從`stdout`跟`stdin`來讀寫。

我們可以看到在`dd`指令的格式中，都是以`arg=value`的形式在傳遞參數的，一些其他常用的參數有：
- `bs=BYTES`: 指定`dd`一次讀和寫多少 bytes，也就是所謂的 block size，預設是 512。
- `ibs=BYTES`, obs=BYTES: 和 bs 類似，但分開指定輸入的 block size 和 輸出的 block size。
- `count=N`: 指定最多讀寫`N`個 block (總共是`N * block size`個 bytes)。
- `status=LEVEL`: 資訊輸出，例如我們經常指定值為`progess`讓他顯示目前進度。
- `conv=CONV`: 有關`dd`的輸入輸出轉換行為都在這邊可以調整，多個值用逗號分隔。常見的值有：
  - `nocreat`: `dd`預設若找不到輸出檔案，則會自行創建，使用這個選項可以讓`dd`只在輸出檔案存在時才執行，避免不必要的檔案複製。
  - `noerror`: 即使發生讀取錯誤也繼續執行。
  - `sync`: 根據 man page，這個選項可以使當 input block 讀進來大小不滿 ibs (input block size) 的時候，在尾端 padding NUL，讓內容可以實際上和 block 對齊。 (註：我不太理解為什麼這個選項叫做`sync`，不過這在硬碟複製的時候蠻常用的，參考：[這篇StackExchange](https://superuser.com/questions/622541/what-does-dd-conv-sync-noerror-do))
- `iflag=FLAG, oflag=FLAG`: 指定在`dd` open file 的時候 (記得讀寫檔案之前都需要先 `open()` 它們嗎？) 要傳入什麼相關的 flag。

# 一些使用情境

## 磁碟複製
假設我們想要將`/dev/sda`磁碟內容完整地複製到`/dev/sdb`中，我們可以使用以下指令。
```
dd if=/dev/sda of=/dev/sdb bs=64K conv=noerror,sync status=progress
```
這裡的 sync，根據 Archlinux Wiki 的說法，可以想成是 data offsets stay in sync。

注意由於我們完整複製了硬碟的內容，這兩個硬碟的UUID也會相同，因此若後續要做使用，則應該先透過`tune2fs`或`mkswap`等工具隨機生成UUID。

## 備份 MBR (Master Boot Record)

MBR 位於硬碟中的前 512 bytes，我們可以透過類似的指令將 MBR 存成檔案：
```
dd if=/dev/sdX of=/path/to/mbr_file.img bs=512 count=1
```

## 將 iso 燒錄到 USB 碟上

製作開機碟也是`dd`可以做到的事：
```
dd if=ubuntu.iso of=/dev/sdX bs=4M conv=fsync oflag=direct status=progress
```
這裡的`fsync`選項讓`dd`確保內容的確物理地寫入到磁碟上再結束，而非可能只寫入到 kernel buffer 中。`direct`則與 syscall `open()`的 flag `O_DIRECT`有關。(這個flag是一個我認為有點難解釋清楚的的 flag，可以詳見[這篇StackOverflow](https://stackoverflow.com/questions/41257656/what-does-o-direct-really-mean))

## 將硬碟上的內容全數寫為零

我們可以利用`/dev/null`來讀出無限量的0位元：
```
dd if=/dev/null of=/dev/sdX bs=4M status=progess
```
就能把整個磁碟寫滿 zero-bit 了。

# 小結
`dd`指令本身不太複雜，但學會使用它可以幫助你在很多資料簡單搬移的情境中省去使用其他特化工具的麻煩，另外關於 iflag、oflag 的部分，主要都跟 syscall `open()` 的 flag 設定有關，筆者目前對相關內容不太熟悉，有興趣的讀者可以去了解相關 flag 可能造成的影響。