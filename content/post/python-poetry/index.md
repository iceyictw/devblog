---
title: Python Poetry
date: 2025-01-04T16:13:27+08:00
draft: true
summary: "使用Poetry管理Python環境"
tags: ["learning", "python"]
---


# 前言
最近想研究如何將自己開發的Python package發佈到`PyPI`等Package Index上面，因此找上了`Poetry`這個工具。
實際上，`Poetry`本身就是一個套件管理工具，能夠彌補`pip`的諸多不足之處，尤其是dependency的管理。因此用這篇文章記錄一下`Poetry`的使用方法。

`Poetry`是一個很年輕的專案，目前的最新版本還是`1.8.5`(截至撰寫這篇文章的2025年1月5日)，因此它所遵循的規範和環境也是較新的。
故本文除了`Poetry`本身外，也會一併介紹其他相關的工具或概念。


# pipx
點進`Poetry`的官網，第一推薦的安裝方式就是使用`pipx`，這也是我第一次接觸`pipx`，這個東西跟`pip`有甚麼不同，又要怎麼使用呢？

`pipx`其實有點類似`npx`。`PyPI`上有一些工具是可以直接當作script來執行的，例如，`autopep8`、`flake8`、`black`等formatter & linter。
很多時候你只是想要把它當作一個command line tool使用而已，而不想汙染全域環境。`pipx`會將你安裝的工具個別放置在獨立的virtual environment中，方便管理。

首先我們要安裝`pipx`。以我使用的`Ubuntu`為例，使用`apt`即可安裝：
```bash
$ sudo apt install pipx
```
接著根據官方文件，要再使用`pipx ensurepath`讓安裝的套件能夠被執行。
(實際上，這個指令只是在你的PATH中加入`~/.local/bin`而已，屆時安裝的套件的執行檔案都會放在這裡。)
```bash
$ pipx ensurepath

# This line will be added to your shell's rc file
export PATH="$PATH:/home/iceyic/.local/bin"
```

接下來我們就試著安裝一些東西吧！以`cowsay`為例：
```bash
$ pipx install cowsay
$ cowsay -t mooo
  ____
| mooo |
  ====
    \
     \
       ^__^
       (oo)\_______
       (__)\       )\/\
           ||----w |
           ||     ||
```
這是怎麼做到的呢？如果使用`which`指令查看`cowsay`是從哪裡被invoke的話：
```bash
$ which cowsay
/home/iceyic/.local/bin/cowsay

$ ls -al ~/.local/bin
...
lrwxrwxrwx 1 iceyic iceyic   54 Jan  5 23:33 cowsay -> /home/iceyic/.local/share/pipx/venvs/cowsay/bin/cowsay
...
```
可以發現，`pipx`實際上為`cowsay`建立了一個虛擬環境，位於`~/.local/share/pipx/venvs/`中。

實際上，每一個`pipx install`都會為該package建立一個完全獨立的環境。

若要移除該package的話，只需要使用`uninstall`即可。
```bash
$ pipx uninstall cowsay
```

大致上這就是`pipx`的原理與使用方法。除此之外，`pipx`也提供像是 **「直接從git repo安裝」、「為一個已經安裝的套件，其環境中加入額外的套件(通常是需要當作plugin時)」** 等功能，
詳情可以參考[pipx Documentation](https://pipx.pypa.io/stable/)。


# Poetry

`poetry`的優點在於能夠解決`pip`無法解決的package相依性問題。在`pip`中，當你uninstall一個package時，它就只會移除該指定的package，卻無法將那些因為相依性而一併被下載的packages一同移除。
另外，`poetry`本身也是一個build frontend，可以幫助library developer發佈distributions。本文介紹`poetry`作為依賴管理工具的常用功能，並且實際發佈測試用的distribution至testpypi上。

我們可以透過`pipx`安裝`Poetry`:
```bash
$ pipx install poetry
```

# Poetry 建立專案

使用`poetry new`會在當前目錄裡起草一個新的專案。
```bash
$ poetry new poetry-demo-iceyic
$ cd poetry-demo-iceyic
```
如果是已經存在的專案，可以在裡面使用`poetry init`。`poetry`會prompt一些關於版本、作者相關的提問，然後產生`pyproject.toml`。

# Poetry 設定與虛擬環境

對於每一個專案，`poetry`都會使用一個虛擬環境來管理專案底下使用到的package，建立這個虛擬環境有三個方法：
  1. 使用`poetry add`加入新的package時自動產生
  2. 使用`poetry env use /path/to/python`指定python執行檔，產生對應版本的python virtual env
  3. 自行使用`venv`、`virtualenv`等工具在工作目錄中產生
一般我會使用`2.`或`3.`，搭配`pyenv`選擇python版本，就能自行控制virtual env的版本。

要注意的是，`Poetry`預設自動產生的virtual env並不是放在專案目錄中，而是放在`~/.cache/pypoetry/`底下統一管理。
個人覺得這不太直覺，所以我通常是把virtual env放在專案目錄底下。

若要修改`poetry`產生virtuelenv的位置，可以使用`poetry config`:
```bash
$ poetry config virtualenvs.in-project true
```

假設我們現在想要使用3.12.8作為虛擬環境的版本，就可以先用`pyenv`切換版本，再使用`poetry`產生環境:
```bash
$ pyenv global 3.12.8
$ poetry env use python 
```

最後我們可以用`poetry env info`確認`poetry`是否偵測到了工作目錄中的虛擬環境:
```bash
$ poetry env info

Virtualenv
Python:         3.12.8
Implementation: CPython
Path:           /home/iceyic/poetry-demo-iceyic/.venv
Executable:     /home/iceyic/poetry-demo-iceyic/.venv/bin/python
Valid:          True
...
```

# Poetry 管理套件

`poetry add`: 在環境中加入套件
`poetry install`: 根據`pyproject.toml`安裝套件

# 後記
Poetry也有提供方便打包python package並上傳的功能，詳情可以參考Poetry的文件。

# Reference
## Python Packaging
https://packaging.python.org/en/latest/guides/distributing-packages-using-setuptools/
https://betterscientificsoftware.github.io/python-for-hpc/tutorials/python-pypi-packaging/
https://bernat.tech/posts/growing-pain/

## pipx
https://pipx.pypa.io/stable/

## Poetry
https://github.com/python-poetry/poetry
https://python-poetry.org/docs/
https://blog.kyomind.tw/python-poetry/