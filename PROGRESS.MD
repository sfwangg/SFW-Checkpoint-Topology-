# Check Point 網路拓樸工具｜工作紀錄

## 2026-07-19

### 解析與邏輯修正

- 管理 IP 改依 `set management interface <介面>` 指向的介面 IPv4 取得；查無設定、介面或 IPv4 時統一顯示「未定義」。
- Static Route 解析支援 `nexthop gateway address <IP> priority <數值> on/off` 格式，包含帶 priority 的 `default` 預設路由，並正規化為 `0.0.0.0/0`。
- SMS 路由新增直接連線優先判斷：SMS IP 落在 ROOT 介面網段時，直接由介面連至 SMS；非直連才走最長前綴 Static Route／Next Hop。
- Heartbeat 規則再收緊為只有 `cphaprob -a if` 的 `(S)` 標記才算 HB；`Beat` 等介面註解僅保留為備註，不再觸發 HB 判定。
- 移除依成員 IP 相鄰位置減一推測 Cluster IP 的邏輯；沒有明確 VIP／virtual IP 時不顯示 Cluster IP，也不將該介面標記為 Cluster 介面。
- Cluster IP 改解析 `cphaprob -a if` 的 `Virtual cluster interfaces`；未含遮罩的 IP 會依同介面的 IPv4 網段補上 CIDR，無可用網段時才使用 `/32`。
- 跨設備相關路由改為必須「雙方路由皆涵蓋對方設備 IP」才顯示；移除僅因目的網段相同而產生的關聯線。

### 互動、效能與介面

- 移除已匯入檔案後會自動重新解析；移除最後一份時清空舊拓樸、統計與表格。
- 拖曳節點時採 `requestAnimationFrame` 合併邊線重繪，並快取 SVG 邊線元素，降低大量節點時的拖曳負擔。
- 「符合畫面」改為依拓樸節點實際範圍與畫布可用寬高計算縮放倍率，不再只固定回到 100%。
- 拓樸產生後新增節點碰撞整理；節點重疊面積超過較小節點的 20% 時會自動推開，並優先固定 ROOT、補充 ROOT 與 Bond 父節點位置。
- 更新匯入提示為 `show configuration` 與 `cphaprob -a if` 的 `.log`／`.txt`；移除重複描述。
- 「匯入補充設備描述」標記為非必需，且「選取 TXT」按鈕樣式改為與 SMS「確定」一致。
- PNG 匯出檔名改為 `checkpoint-network-topology-<timestamp>.png`。

### 驗證

- 三段 JavaScript 語法檢查通過。
- 驗證 `set management interface bond1` 與 `bond1.11` 可正確找到管理 IP；無此設定時回傳「未定義」。
- 驗證 `set static-route default nexthop gateway address <IP> priority 1 on` 可正確解析為 `0.0.0.0/0`，並在 `Checkpoint_CONFIG` 案例中取得預設路由 Next Hop。
- 驗證 `Interface Name: eth1 (S)` 會判定 `eth1` 為 Heartbeat。
- 驗證雙向路由會產生關聯，單向路由不產生關聯。

## 2026-07-18

### 功能與需求實作

- 完成從 Palo Alto／XML 方向改為 Check Point Gaia `.log`／`.txt` 工作流程，移除 XML 上傳、解析與相關文字描述。
- 依 `Checkpoint_CONFIG` 驗證設定檔解析介面、路由、註解、Bond、VLAN、VIP、HB 與 HA 資訊。
- 增加 HA Cluster 判斷與兩台設備合併成單一 `CLUSTER ROOT`。
- Cluster ROOT 顯示成員主機名稱與管理 IP；Cluster 介面顯示兩台成員 IP 與 Cluster IP。
- 非 Cluster 介面逐台顯示「主機名稱：IP」及介面註解；Mgmt／Heartbeat 介面保留介面名稱。
- Bond 顯示成員組合；Bond 後方帶 VLAN tag 時建立 `Bond → VLAN` 階層，VLAN 顯示為 `BondX.tag`。
- 修正 VLAN 邊線生成、HA 判斷過寬與父 Bond 位置依第一個 VLAN 決定等問題。
- VLAN 改用深玫紅色；Cluster 介面使用較深綠色；Next Hop 來源主機改用黃色提高辨識度。

### 互動與版面

- ROOT、Bond、介面拖曳時遞迴帶動子節點。
- 新增「Next Hop 對端」顯示開關與相關路由顯示開關。
- 調整 SVG 寬度符合瀏覽器可用寬度；一般介面節點寬度縮短，Cluster 介面再縮短。
- 介面格式統一為 `介面 · ethX`／`介面 · Mgmt`；Bond 名稱與成員分兩行。
- IP 資訊、Cluster IP、介面名稱與 Cluster ROOT 字體依畫面可讀性調整。
- Ping 圖示提示改為「複製 ping ${Next Hop IP}」，實際複製內容為 `ping <Next Hop IP>`。
- 新增 SMS 管理 IP 輸入欄位與「確定」按鈕；SMS 節點依 ROOT 路由建立連線。
- 將匯入區標題改為「匯入補充設備描述」，補充 TXT 與 SMS 欄位置於拖曳檔案區下方；目前順序為補充 TXT 在上、SMS 在下。

### 驗證

- 每次程式調整後執行 Node.js JavaScript 語法檢查並通過。
- 以單機、雙機 HA、VLAN、Bond、Heartbeat、路由註解與 SMS 情境反覆確認畫面。

## 待辦

- 建立獨立 `cphaprob -a if` 檔案與設備設定檔的可靠對應機制。
- 增加自動化 parser 回歸測試與瀏覽器互動測試。
- 將單一 `index.html` 拆分為模組，並保留目前資料模型與視圖行為。
