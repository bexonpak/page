Mac 中使用 Windows 版 Steam 的方法 （通过 Sikarugir）

即使想在 Steam 上玩的遊戲有 Windows 版但沒有 Mac 版的情況並不少見。因此，我試著利用 Sikarugir 在 Mac 上執行 Windows 版的遊戲。

※ Mac 版 Steam 已安裝且 Steam 帳號已建立。

先安裝：

[Sikarugir-App/Sikarugir](https://github.com/Sikarugir-App/Sikarugir)

自己的情況下，我透過下述方法成功啟動了 Luma Island

在「應用程式資料夾」中雙擊「Sikarugir Creator.app」，

點選「+」下載「WS12WineCX24.0.7_6」引擎

<img width="516" alt="Bildschirmfoto 2025-11-18 um 18.11.27" src="Bildschirmfoto 2025-11-18 um 18.11.27.png">

點選「Create New Blank Wrapper」

<img width="410" alt="Bildschirmfoto 2025-11-18 um 18.11.54" src="Bildschirmfoto 2025-11-18 um 18.11.54.png">

安裝成功後在 User 的 Applications 檔案夾找到 app 開啟

```/Users/{user_name}/Applications/Sikarugir/SteamWindows.app```

點選 Winetricks ，勾選

<img width="915" alt="Bildschirmfoto 2025-11-18 um 18.20.13" src="Bildschirmfoto 2025-11-18 um 18.20.13.png">

* apps
  * steam
* dlls
  * d3dx11_42
  * d3dx11_43
* fonts
  * fakechinese

點選 Run，等待安裝完成

點選 Windows app 後面的 Broswe 找到 Steam.exe

<img width="797" alt="Bildschirmfoto 2025-11-18 um 18.16.16" src="Bildschirmfoto 2025-11-18 um 18.16.16.png">

在路徑後面新增

```
-udpforce -allosarches -cef-force-32bit
```

最後大概是這個樣子

```
"C:\Program Files (x86)\Steam\Steam.exe" -udpforce -allosarches -cef-force-32bit
```

點選 「Test Run」啟動。

---

啟動測試了一下 Luma Island 

<img width="1392" alt="89a7e786a3e52c57fd2fd0a38c644afe" src="89a7e786a3e52c57fd2fd0a38c644afe.png">