# 無線連接設備

> 編寫:[acenodie](https://github.com/acenodie) - 原文:<http://developer.android.com/training/connect-devices-wirelessly/index.html>

除了能夠在雲端通信，Android 的無線 API 也允許同一局域網中的設備進行通信，甚至沒有連接到網絡上，而是物理上隔得很近，也可以相互通信。此外，網絡服務發現（Network Service Discovery，簡稱NSD）可以進一步通過允許應用程序運行能相互通信的服務去尋找附近運行相同服務的設備。把這個功能整合到我們的應用中，可以提供許多功能，如在同一個房間，用戶玩遊戲，可以利用 NSD 實現從一個網絡攝像頭獲取圖像，或遠程登錄到在同一網絡中的其他機器。

本節課介紹了一些使我們的應用程序能夠尋找和連接其他設備的主要 API。具體地說，它介紹了用於發現可用服務的 NSD API 和能實現點對點無線連接的無線點對點（the Wi-Fi Peer-to-Peer，簡稱 Wi-Fi P2P）API。本節課也將告訴我們怎樣將 NSD 和 Wi-Fi P2P 結合起來去檢測其他設備所提供的服務。當檢測到時，連接到相應的設備上。即使設備都沒有連接到一個網絡中。

## Lessons

[**使用網絡服務發現**](nsd.html)

  學習如何廣播由我們自己的應用程序提供的服務，如何發現在本地網絡上提供的服務，並用 NSD 獲取我們將要連接的服務的詳細信息。


[**使用 WiFi 建立 P2P 連接**](wifi-direct.html)

  學習如何獲取附近的對等設備，如何創建一個設備接入點，如何連接到其他具有 Wi-Fi P2P 連接功能的設備。


[**使用 WiFi P2P 發現服務**](nsd-wifi-index.html)

  學習如何使用 WiFi P2P 服務去發現附近的不在同一個網絡的服務。
