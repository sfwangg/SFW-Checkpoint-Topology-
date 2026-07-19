# Check Point 網路拓樸工具｜開發計畫

## 目標

建立一個完全在瀏覽器本機執行的 Check Point 網路拓樸分析工具。使用者匯入 Gaia `show configuration` 與可選的 `cphaprob -a if` 輸出 `.log`／`.txt` 後，系統解析設備、介面、Bond、VLAN、靜態路由、Next Hop、HA Cluster 與 Heartbeat，並以可互動 SVG 拓樸呈現。

## 已完成

- 支援一次匯入 1–7 份 `.log`／`.txt` 設定檔；不提供 XML 上傳或 XML 解析。
- 解析 `set interface`、IPv4／CIDR、`set static-route`、Next Hop 與路由註解。
- 解析 Bond 成員、VLAN 子介面、VIP／Cluster IP、Heartbeat／HB 介面與介面註解；Cluster IP 優先取自 `cphaprob -a if` 的 `Virtual cluster interfaces`。
- 依嚴格 HA 條件合併同一 Cluster 的兩份設定：Cluster mode、同一 `/24` 管理網段與相鄰管理 IP。
- Cluster 以單一 `CLUSTER ROOT` 顯示，成員主機名稱與管理 IP 分行呈現。
- 建立 `ROOT → Interface/Bond → VLAN → Next Hop` 的拓樸階層；VLAN 以父 Bond 為圓心環繞排列。
- Cluster 介面顯示 Cluster IP，兩台成員的介面 IP 以主機名稱／末段 IP 呈現；非 Cluster 介面逐台顯示 IP 與註解。
- 支援 Next Hop 對端顯示切換、相關路由顯示切換、主機篩選、縮放、平移、PNG 匯出及節點拖曳。
- 移動 ROOT、Bond 或介面時，遞迴帶動其子節點；位置與縮放在同一次拓樸操作中保留。
- 支援 TXT 補充設備描述（IP／主機／備註），以及 SMS 管理 IP 輸入與「確定」按鈕。
- SMS 節點先判斷 ROOT 的直接連線介面，再依最長前綴 Static Route 與 Next Hop 建立連線；Next Hop 可複製 `ping <Next Hop IP>`。
- 管理 IP 依 `set management interface <介面>` 指向的介面 IPv4 取得；查無命令、介面或 IPv4 時顯示「未定義」。
- `cphaprob -a if` 的介面行出現 `(S)` 時才判定為 Heartbeat；沒有 `(S)` 時，介面註解不會自行推導為 Heartbeat。
- 移除設定檔會立即重新解析剩餘檔案；移除最後一份時清空舊拓樸與統計。
- 跨設備「相關路由」僅在雙方路由皆涵蓋對方設備 IP 時顯示，不再只因目的網段相同而連線。
- 拖曳拓樸時以 `requestAnimationFrame` 合併邊線更新，並快取 SVG 邊線元素以降低重繪成本。
- 匯入區已明確說明 `show configuration` 與 `cphaprob -a if` 輸出；補充設備描述標示為非必需。

## 後續計畫

1. 定義並實作「`show configuration` 與獨立 `cphaprob -a if` 檔案」的設備對應規則；目前兩者需位於同一份匯入檔才能可靠套用 Heartbeat。
2. 補強 HA Cluster 判定，避免僅憑 Cluster mode、同 `/24` 與相鄰管理 IP 誤合併設備。
3. 以 `Checkpoint_CONFIG` 及人工案例建立 parser 回歸測試：單機、HA、VLAN、Bond、無管理介面、SMS 直連／靜態路由與 `cphaprob -a if`。
4. 補強 parser 對不同 Gaia 版本、路由屬性、註解格式及多種 Bond／VLAN 寫法的容錯。
5. 將解析、資料模型、SVG 繪圖與互動控制拆成模組，降低單一 `index.html` 的維護成本。

## 開發原則

- 所有設定檔只在瀏覽器本機解析，不上傳伺服器。
- 以 IPv4 嚴格驗證與最長前綴匹配作為路由判斷基礎。
- Heartbeat 只能以 `cphaprob -a if` 的 `(S)` 標記判定；介面註解僅作一般備註，不得推導 Heartbeat。
- 修改互動或版面後，確認拓樸、表格、篩選器與清除狀態同步。
- 每次修改至少執行 JavaScript 語法檢查，涉及 parser 時以驗證設定檔實測。
