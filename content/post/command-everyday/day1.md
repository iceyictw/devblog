---
title: 一天一指令 Day 1 - grep 和他的兄弟：egrep、fgrep、rgrep
date: 2025-05-29T21:59:12+08:00
draft: true
summary: "關於grep的常用功能及常見問題"
tags: ["code", "learning", "linux"]
---

# 前言
之所以想要開啟這個系列，主要是因為發覺自己在使用很多工具程式，需要某種特定用法的時候，幾乎每一次都會打開 man page 或是 Chrome 去搜尋正確下 <ins>OPTION</ins> 的方法。雖然看似只是小問題，但我覺得這件事其實會影響整體工作效率。另一點就是因為有個系上同學都會知道很多「原來這個工具還有這樣的功能！」因此這個系列希望可以每天找一個常用的Linux指令，來學習它具有的各種功能，並嘗試記住然後使用在日常工作中。

今天要學習的是`grep`。

# grep

grep可以在檔案或是`stdin`中用regex搜尋內容。

首先來看SYNOPSIS：
```
grep [<ins><ins>OPTION</ins></ins>...] <ins>PATTERNS</ins> [<ins>FILE</ins>...]
grep [<ins>OPTION</ins>...] -e <ins>PATTERNS</ins> ... [<ins>FILE</ins>...]
grep [<ins>OPTION</ins>...] -f <ins>PATTERN_FILE</ins> ... [<ins>FILE</ins>...]
```

PATTERNS 是你要搜尋的regex表示式，FILE 則是要搜尋的檔案。
FILE 可以一次輸入多個，`grep` 會搜尋每一個檔案。
PATTERNS 也可以多個，但是要使用選項`-e`來重複指定。
```
$ grep -e pattern1 -e pattern2 file.txt
```

如果檔名為`-`，則會改為讀stdin。如果沒有給檔名，那麼會有兩種情況：
- nonrecursive search: 讀`stdin`
- recursive search: 讀 working directory

# 常用 Options

## r for recursive: 遞迴搜尋整個目錄
使用`-r`便可以遞迴地搜尋目錄內的所有檔案，這在尋找C code中的definition時會很好用。
使用`-r`時，也會預設開啟`-H`選項：顯示各行所屬的檔名。
`-R`與`-r`類似，但會 follow symbolic link。

```
grep -r "sys_open" xv6/kernel

$ grep -r "sys_open" xv6/kernel
xv6/kernel/syscall.c:extern uint64 sys_open(void);
xv6/kernel/syscall.c:[SYS_open]    sys_open,
xv6/kernel/kernel.asm:000000008000558e <sys_open>:
xv6/kernel/kernel.asm:sys_open(void)
xv6/kernel/kernel.asm:    800055b0:     0c054163                bltz    a0,80005672 <sys_open+0xe4>
xv6/kernel/kernel.asm:    800055c2:     0a054863                bltz    a0,80005672 <sys_open+0xe4>
xv6/kernel/kernel.asm:    800055d6:     cbdd                    beqz    a5,8000568c <sys_open+0xfe>
...
grep: xv6/kernel/sysfile.o: binary file matches
grep: xv6/kernel/syscall.o: binary file matches
grep: xv6/kernel/kernel: binary file matches
```

但是注意在上面的例子中，有些檔案不是文字檔，因此grep會回傳錯誤訊息。

## s for suppress: 不顯示錯誤訊息
使用`-s`可以讓grep不顯示檔案不存在，或是檔案無法讀取等錯誤。

## n for number: 顯示行數
使用`-n`可以顯示matching line在檔案中的行數。

## i for ignore: 忽略大小寫差異
使用`-i`可以在matching時忽略大小寫。

# egrep、fgrep 和 rgrep
有時候我們會看到`grep`的變種，這些變種實際上等價於`grep`加上一些選項：
- `egrep`: 等價於`grep -E`，使用 extended regular expression。
- `rgrep`: 等價於`grep -r`。
- `fgrep`: 等價於`grep -F`，將 PATTERNS 視為普通字串，而非 regex。

這些指令實際上為 deprecated 狀態，保留它們只因相容性問題，實務上時盡量避免使用。