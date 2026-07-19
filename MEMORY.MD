# Check Point 網路拓樸工具｜長期知識與需求

## 專案定位

- 專案是 Check Point Gaia 網路拓樸工具，不是 Palo Alto／XML 工具。
- 慣用語言為繁體中文。
- 主要輸入是 Gaia `show configuration` 與可選的 `cphaprob -a if` 輸出 `.log`／`.txt`；不可重新加入 XML 功能或 XML 文字描述。
- 系統必須維持純本機解析，不上傳設定檔。

## 長期需求

- 支援最多 7 份 Check Point 設定檔並可比較／整合。
- 介面需顯示名稱、IP／CIDR、註解、Bond 成員、VIP／Cluster IP 與 HB 狀態。
- Bond 後有 VLAN tag 時，必須呈現 `ROOT → Bond → VLAN → Next Hop`，VLAN 名稱為 `BondX.tag`。
- HA 兩份設定判斷為同一 Cluster 時合併成一個 ROOT，但 ROOT 內仍要保留兩台成員的主機名稱與 IP。
- Cluster 介面顯示 Cluster IP 及兩台成員 IP；一般介面逐台顯示主機名稱、IP 與註解。
- Mgmt 若介面名稱就是 `Mgmt`，不可因註解而改名；Heartbeat 註解另行顯示。
- Next Hop 必須顯示來源主機；來源主機文字使用黃色以提高對比。
- Next Hop Ping 操作提示為「複製 ping <Next Hop IP>」，命令不可帶 source。
- SMS 管理 IP 由使用者輸入並按「確定」後才生效；依 ROOT 最長前綴路由連到 SMS 節點。
- SMS IP 若位於 ROOT 的直接連線介面網段，必須直接由該介面連到 SMS；只有非直連時才使用 Static Route 與 Next Hop。
- 補充 TXT 為 IP／主機名稱／備註三欄；相同主機名稱應合併為補充 ROOT。
- 管理 IP 必須由 `set management interface <介面>` 指向的介面 IPv4 取得；查無設定、介面或 IPv4 時顯示「未定義」，不可改用檔名或 `Mgmt` 介面猜測。
- Heartbeat 必須且只能以 `cphaprob -a if` 介面行的 `(S)` 標記判定；`Beat`、`Heartbeat` 等介面註解不可自行推導 HB。

## 視覺與互動慣例

- ROOT 位於群組中心，介面環繞 ROOT；兩組相近管理 IP 的拓樸應採上下鏡像／對齊方向。
- 移動父節點時，所有子節點必須一起移動；移動子節點本身不應拖動其他分支。
- VLAN 採深玫紅色，Cluster 介面採較深綠色，Next Hop 採紫色，ROOT 採藍色。
- SVG 寬度使用瀏覽器可用寬度；畫布允許內部平移與縮放。
- 文字不可依註解取代介面名稱；節點資訊應刪除不必要空白行。
- 相關路由線只表示「雙向路由覆蓋」的邏輯關聯，不代表實體連線；相同目的網段不足以建立關聯。

## 解析與判斷原則

- IPv4 與 CIDR 必須使用嚴格驗證。
- 路由先比對直接連線，再使用最長前綴 Static Route。
- HA Cluster 不可只依主機名稱或單一 IP 推測，需有明確 Cluster mode、同 /24 且管理 IP 相鄰。
- Cluster IP 以 `cphaprob -a if` 的 `Virtual cluster interfaces` 列出的介面與 IP 為主要來源；相容設定檔明確 VIP／virtual IP。不得以成員 IP 相鄰或減一規則推測。
- VLAN 父 Bond 的位置要由全部 VLAN 的幾何分布計算，不得取第一個 VLAN 作為唯一依據。
- 路由註解若存在必須在介面／Next Hop 或路由表適當位置顯示。
- `cphaprob -a if` 若和 `show configuration` 分成不同檔案，未建立可靠設備配對前不可任意合併 Heartbeat 資訊。

## 維護注意事項

- 目前所有程式集中於 `index.html`，修改時需同時檢查 HTML、CSS 與 JavaScript。
- `state.renderVersion` 不可移除，避免非同步解析競態。
- 清除功能需重設檔案、補充 TXT、SMS IP、節點位置、縮放、選取與篩選狀態。
- 任何新功能完成後，至少執行 Node.js JavaScript 語法檢查，並以 `Checkpoint_CONFIG` 實測。
- 拖曳互動的邊線更新應維持逐影格合併與 SVG 元素快取，避免大量節點時退化。
