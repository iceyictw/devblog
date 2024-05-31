---
title: 台大資工NASA修課心得(2)-第5~8週
date: 2024-04-10T02:48:54+08:00
draft: false
summary: "關於NASA(網路管理及系統管理)課程的內容摘要、筆記與心得，之二"
tags: ["course", "learning"]
---

# 前情提要
大一還懵懂無知的iceyic，在各種推薦文與學長姊的介紹之下，誤打誤撞進入了這門名為NASA的神祕課程。
雖然課程內容繁重，但iceyic仍然很高興能夠學到如此實用又多樣化的電腦管理技能。
殊不知，一場暴風雨即將改變安寧，iceyic接下來要面對的，是一場殘酷的腥風血雨...

以上...全部都是瞎掰的。想知道前幾週發生什麼事的可以參考我上一篇文章。

# 第5週：虛擬機器、Docker和Kubernetes
本週的教學內容就是虛擬化技術了。入門最常接觸到的我想大概就是[VirtualBox](https://www.virtualbox.org/)了，
不過在實務環境中，通常使用Linux作業系統，這次的作業就使用QEMU-KVM，然後再使用virsh指令工具，透過libvirt API來進行管理。
[libvirt](https://libvirt.org/)是一個C Library，因為Hypervisor的種類繁多，例如：KVM、IOS的Hypervisor.framework、Xen等等，
每個工具的參數眾多，難以使用，因此，libvirt就為了方便管理而誕生。

什麼是Hypervisor呢？QEMU這類的模擬器，是運用程式來模擬電腦中CPU的運算、暫存器和記憶體的狀態、網路卡的狀態等等手法來達到虛擬機的效果，實際上，我們電腦的CPU沒有真正「執行」到虛擬機裡的CPU instruction。可以預料到，這樣的效能會相當差。
Hypervisor是一種特別的軟體，能夠讓CPU直接「執行」虛擬機中的instruction，因此得以實際上讓虛擬機能夠享用CPU的運算資源。

課程的後半段，就是在討論近段時間崛起的容器化技術了，Docker就是一個非常廣泛且普遍的容器化服務。Container的優點就是體積小，建置方便，很多常用的伺服器在Docker Hub上面都找得到可用的image，例如：反向代理伺服器nginx、資料庫伺服器MySQL server，甚至是[minecraft server](https://hub.docker.com/r/itzg/minecraft-server)！

而更進階的container管理工具就是Kubernetes了，又簡稱K8s。他提供的功能強大，包括：使用複製容器(replicas)讓上線容器在出狀況時能快速替換、負載平衡、CI/CD等等。Kubernetes是在企業環境中廣泛使用的工具，因此在這邊不多加贅述。

本次的作業就是試著使用Virsh在系工作站開啟一個虛擬機器，並且在這個虛擬機器中用docker以及kubernetes啟動一些服務。
照著各工具的Docs提供的選項來設定應該不會過於困難。

# 第6週：Network Layer
接續第3週的Link Layer，這週我們的上課內容是討論Network Layer。我們先來複習一下Ethernet Frame的結構：

{{<figure src="hugo-logo.svg" attr="Logo of [Hugo](https://gohugo.io/)" align="center">}}

其中，Payload的部分，正是Ethernet Frame所要攜帶的資訊，除了像是之前介紹的ARP就是放在這裡面以外，還有大名鼎鼎的IP也是。

IP packet中包含了許多資訊，重要的幾個有：
- 來源IP及目的IP
- 上層協議，例如：UDP、TCP
- checksum
- TTL
- IP packet fragmentation所使用的 3 bit flags + 13 bit offset
- etc...

其中注意到Fragmentation，這是IP在data太大時，為了分割內容所使用的技巧，
這16個bit可以告訴其他網路設備是否有其他片段、可否進行fragmentation、這個片段是從data的第幾個byte開始。

IP位址分為兩個部分：網路位址跟主機位址，使用設定好的子網路遮罩便可知道IP位址中到哪個bit的範圍是網路位址，剩下的部分就是主機位址。
網路位址跟主機位址的關係就好像是一個巷子裡會有好幾號的住戶，這些住戶住得近，可以直接彼此交流。

當主機想要發送IP封包時，它得要透過乙太網路來發送嘛，這時就要先決定目的MAC address，這要怎麼做呢？
首先，他會檢查目的IP位址是否跟自己主機網卡的IP位址擁有同樣的網路位址，同樣的網路位址代表它們在Link Layer中是同一個network的，
此時，就可以使用ARP來取得目標MAC address，最後成功將封包送出。
如果網路位址不同，我們就會嘗試將封包送到預先設定好的「default gateway」這個IP位址，通常會是你路由器的IP位址，讓路由器替你轉送，
此時，目標MAC address就會是路由器的MAC address。

以上就是IP大致上運作的方式。除此之外，課程中還討論了ICMP、DHCP、NAT等協定，不過它們不複雜，在理解IP之後都可以快速理解，這裡不多贅述。

作業時間！本週的作業是[OPNSense](https://opnsense.org/)，一款開源的防火牆軟體，我們會在自己的電腦上架設一個OPNSense和兩台虛擬主機，
用來模擬一個區域網路中防火牆控制流量的功能。OPNSense有提供Web GUI讓我們操作設定，所以不花太多時間就能習慣操作。

# 第7週：Transport Layer、DNS
有了網路協定(IP)後，我們能夠傳遞資訊給任何一台已知IP的主機了，接下來呢？假設A主機想傳送3個bytes給B主機，他要怎麼做？把資料放進IP packet中送出嗎？
如果這3個bytes非常重要，幾乎不容許出錯呢？A主機要多送幾次嗎？要送幾次？10次？50次？這樣會造成網路壅塞嗎？

從上面的假想問題可知，我們的確需要幾套可以讓各個主機遵循的固定流程，來進行有效通訊。
而Transport Layer最具代表性的協定就是UDP與TCP了。

UDP的核心概念就是「不管怎樣把資料送過去就對了」。UDP frame中只有四個欄位：

- source port
- destination port
- data
- checksum

UDP是IP之上的協定，所以實際上這個UDP frame是放在IP packet中的data欄位來進行傳輸的。
UDP frame中提供的資訊，大概就只能讓對方主機知道封包是給哪個程式、以及資料是否有誤。這可以幹嘛呢？
舉例來說，語音通訊時，因為聲音是連續的資訊，即使丟失了一部分的資訊也不會造成嚴重的影響，因此通常使效率較高(與TCP相比之下)的UDP。

TCP更為複雜，簡明扼要地說明就是，TCP一開始會進行一個名為「三方交握(Three-way Handshaking)」的流程，讓雙方都知道這個TCP連線建立了。
接著，每次傳送資料時，接收方都要回應一個對應的封包來表明自己的確有收到，否則傳送方就必須要把那個可能缺失的資料重新傳一次。

DNS最主要的功能就是將域名轉換成IP，這可以透過名為DNS resolver的主機、以及name server的主機做到。電腦想查詢一個domain name的IP時，
會發送DNS請求到一台DNS resolver，要使用哪一台resolver可以由使用者自行設定，或是取得IP時DHCP伺服器指派。
DNS resolver接著會「遞迴地」跟各個層級的name server詢問，最後得到答案後，resolver可以將結果cache起來，以便之後重複拜訪時可以快速回應。

作業時間！儘管業界目前大部分都使用Bind 9架設DNS為主，不過這次我們要的是使用[PowerDNS](https://www.powerdns.com/)，並且練習使用dig指令來進行DNS查詢。
由於PowerDNS有提供Web GUI，因此設定起來相對簡單。

# 第8週：期中考
期中考之前會先以三人一組事先分好，每個人的考試成績就是該小組的分數。

期中考以上機操作的形式進行，題目基本上涵蓋了第1到7週的各個主題，其中也包含需要操作額外設備的題目(例如:一台無法開機的電腦，需要在考試中找出問題並修復)。
考試時間只有3小時，題目卻有8題，因此事前的分配是很重要的，通常會3個人下去分配自己要負責哪些主題，然後考前可以專心研究該主題的一些指令、觀念和cheatsheet。
滿分100分，最高分的組別拿到了43分，可知考試的難度非同小可。不過根據其他網站上的消息，通常期末的分數會好拿很多，希望到時可以彌補期中的分數漏洞。

(待續...)