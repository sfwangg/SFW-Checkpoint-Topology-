# Check Point 網路拓樸工具｜系統架構與設計說明

## 系統型態

系統是單頁式、純前端靜態網頁，主要程式集中於 `index.html`：

- HTML：檔案匯入、補充描述、SMS 輸入、拓樸畫布、統計、介面表格與路由表格。
- CSS：響應式版面、SVG 節點顏色、Cluster／VLAN／Next Hop 狀態與互動樣式。
- JavaScript：File API 讀檔、Check Point 設定解析、資料模型、SVG 佈局、篩選、拖曳與匯出。

沒有後端、資料庫或外部 API；所有資料留在瀏覽器記憶體。

## 資料流程

```text
show configuration／cphaprob -a if ─┐
                                    ├─> parseGaiaConfiguration()
補充描述 TXT ────────────────────────┘             │
                                     ▼
                           設備／介面／路由資料
                                     │
                           buildDataModel()
                                     │
                    Cluster 合併、路由與關聯索引
                                     │
                         draw() 建立 SVG 拓樸
                                     │
               篩選、拖曳、縮放、表格與 PNG 匯出
```

## 核心狀態

- `state.files`：待解析的 `.log`／`.txt` 檔案。
- `state.data`：每個檔案解析出的原始設備資料。
- `state.model`：合併後的設備、介面、路由與索引。
- `state.supplements`：以 Next Hop IP 為鍵的補充主機描述陣列。
- `state.smsIp`：使用者按下確定後採用的 SMS 管理 IP。
- `state.renderVersion`：避免非同步解析完成順序造成舊結果覆蓋新結果。
- `treeUi`：節點位置、縮放、選取、主機／介面／路由篩選與視圖開關。

## 設定檔解析

`parseGaiaConfiguration()` 主要處理：

- 主機名稱、`set management interface <介面>` 與該介面的管理 IP；找不到時管理 IP 為「未定義」。
- `set interface` 的介面 IP／CIDR與介面註解。
- `cphaprob -a if` 中介面行的 `(S)` 標記；此標記優先作為 Heartbeat 判定，缺少該輸出時才以註解備援。
- Bond 介面及其成員；顯示格式為 `Bond1 (eth1-05 + eth5)`。
- `bondX.<tag>` VLAN 子介面；拓樸節點顯示為 `BondX.<tag>`。
- VIP／virtual IP／Cluster IP。
- `set static-route` 的目的網段、Next Hop、介面及註解。
- `CLUSTER_MODE`，供 HA Cluster 判斷使用。

目前 `cphaprob -a if` 與 `show configuration` 必須位於同一份匯入檔才能直接關聯；獨立檔案尚未有可靠的設備配對機制。

## HA Cluster 合併

只有同時符合以下條件才合併：

1. 兩份設定都明確出現 Cluster mode 設定。
2. 管理 IP 位於同一個 `/24` 網段。
3. 管理 IP 為相鄰位址。

合併後建立一個 `CLUSTER ROOT`，保留兩個成員的主機名稱、管理 IP、介面 IP、Cluster IP 與註解。若條件不符，維持獨立 ROOT。

## 拓樸佈局

- ROOT 位於設備群組中心，介面與 Next Hop 以放射狀排列。
- Bond 是父介面；VLAN 子節點以所有 VLAN 的平均角度計算父 Bond 位置，再以父 Bond 為圓心環繞排列，避免位置依第一個 VLAN 決定。
- 邊線分為設備連線、Next Hop 對端、雙向路由覆蓋、補充主機與 SMS 路由；相同目的網段本身不構成跨設備關聯。
- 父節點拖曳會遞迴平移所有 descendants，並以 `requestAnimationFrame` 合併邊線更新；SVG 邊線元素會快取，避免每次拖曳都重複查詢 DOM。

## 顯示規則

- 一般介面標題：`介面 · ethX` 或 `介面 · Mgmt`。
- Cluster 介面使用較深綠色，顯示 Cluster IP，並以 `主機名稱：IP` 呈現成員 IP。
- VLAN 使用深玫紅色，顯示 IP；父 Bond 顯示兩行名稱與成員。
- Next Hop 顯示路由數、目的網段、來源主機；來源主機使用黃色字體。
- TXT 補充主機使用黃色來源標示；SMS 使用獨立節點與路由線。

## 路由與 SMS

路由判斷先檢查直接連線網段，再以最長前綴選擇 Static Route。輸入 SMS IP 並按下「確定」後，若 SMS 位於 ROOT 的直接連線介面，介面直接連至 SMS；否則由對應 Next Hop 連至 SMS。

## 安全與限制

- 不接受 XML；檔案選擇器只接受 `.log`／`.txt`。
- IPv4 四段均須為 0–255，CIDR prefix 須為 0–32。
- SVG 及檔案內容皆在本機處理，沒有網路傳輸。
