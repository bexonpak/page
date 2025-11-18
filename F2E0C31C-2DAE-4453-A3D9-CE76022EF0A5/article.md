Xcode Swift Package Manager 無法下載 GitHub 包問題

<img width="128" alt="xcode-s-128x128_2x" src="xcode-s-128x128_2x.png">

因為中國大陸網路複雜度的原因，GitHub 在不同時間，不同寬頻運營商，不同省份的訪問速度和可到情況都不同。

而 GitHub 承擔了絕大部分的 swift package 的分發。

但 Xcode 的網路請求並不會透過系統 proxy，這時看著 fetching... 就會幹著急。

Xcode 雖然不經過系統的 proxy，但是 Terminal 可以呀！

1. 使用終端進入專案 root 目錄

2. 複製並在終端執行終端代理命令（這裡以 clash 為演示）
   <img width="212" alt="Bildschirmfoto 2025-09-18 um 18.10.29" src="Bildschirmfoto 2025-09-18 um 18.10.29.png">

   ``` shell
   export https_proxy=http://127.0.0.1:7897 http_proxy=http://127.0.0.1:7897 all_proxy=socks5://127.0.0.1:7897
   ```

3. 再執行 xcodebuild 的 fetch swift package 的命令就好啦~

   ``` shell
   xcodebuild -resolvePackageDependencies -scmProvider system
   ```

4. 當你看到 `resolved source packages` 就大功告成！

祝程式設計愉悅！