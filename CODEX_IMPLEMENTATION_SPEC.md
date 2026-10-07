# 《詭地圖 EP.01》CODEX IMPLEMENTATION SPEC

版本：2.0 · Evidence-first implementation contract

本文件依使用者提供的 Master Brief 設計，不新增歷史研究或認定。文中的「已確認」指該資料包所標示的狀態，不代表實作者已另行查證。目標是 10–15 分鐘、單人、Mobile-first 的網頁歷史鑑識遊戲。

## 0. 交付與不可突破的邊界

- 新版與現有 Demo 分開開發，先保存原版；完成驗收後才以新版替換桌面「鬼地圖/index.html」。
- 單一 HTML 內含 CSS、JS、集中研究設定、證物資料。合法大型影像可放同資料夾；未附影像時須完整支援 metadata／待研究模式。無第三方網路依賴，不自動下載圖片。
- 舊版偽公文、虛構剪報、猜測的 1904 地圖、模擬 1945 航照、固定 GPS 與古路「吻合」結論，不得轉入新版 ARCHIVE。舊版的三民路坐標不能作為第一代市场定位依據。
- 新調查範圍是桃園舊城、景福宮後方、舊永和市場一帶；第一代建物範圍仍待研究。不得拿第三代範圍補空缺。
- 沒有史料影像時，顯示「影像待提供」的中性空框及資料卡；不得用擬真生成圖、SVG 建物、假報紙排版冒充影像。
- 所有超自然訊息標為 FICTION，與 ARCHIVE 使用不同容器、資料型別及匯出欄位。此版在研究結果未完成時不啟用超自然終章。
- `VERDICT = PENDING` 是初始且目前唯一允許的正式研究結果。玩家操作、點擊次數、畫出候選範圍都不能改寫它。

## A. 完整 Game Flow

|順序|畫面|目標時長|操作與產出|
|---|---|---|---|
|01|PHOTO 開場|0:00–0:45|看影像／缺件卡，辨識哪些 metadata 是 UNKNOWN|
|02|PHOTO 觀察桌|0:45–2:15|放大、標記可見特徵；保存觀察或記錄無法觀察|
|03|ART 比對桌|2:15–3:45|配對影像特徵、判斷相似性與定位證明的差別|
|04|PRESS 時間桌|3:45–5:15|排列已知報刊 metadata，保留 1931／1932 衝突|
|05|1945 盲判 B|5:15–7:45|只找市場候選範圍，不看 1920 水文結果|
|06|命題封套|7:45–8:15|閱讀後世「填埤建市場」說法，標示 HYPOTHESIS|
|07|1920 盲判 A|8:15–10:45|分類地貌，不顯示市場候選範圍|
|08|最後疊圖|10:45–12:45|A/B 完成後解鎖；控制圖層、查看可算性及限制|
|09|結案／暫結|12:45–14:00|選擇證據所允許的結論，匯出調查筆記|

正式史料完整時以 10–15 分鐘為設計目標，不設強制計時。當前缺件版約 5–8 分鐘即可完成 metadata 判讀與缺件清單；不得用等待動畫或假謎題冒充完整體驗。UI 開始即顯示「研究預覽：影像與空間答案尚未齊備」。

雙盲的實作意義是 A/B 互不顯示對方成果、答案與預覽；玩家已看過命題與其他史料，不能宣稱這是嚴格科研意義的完全雙盲。B 先於 A 僅是出場順序，`A PROCESSED`、`B PROCESSED` 指完成處理流程，不代表範圍驗證成功。

## B. Screen-by-Screen 規格

### 01 — `screen-opening`

- **purpose**：用未知照片建立缺席感，讓玩家接受「不知道」是資訊。
- **visual_layout**：影像為主、右／下方 monospace metadata；底部唯一開始按鈕。缺影像則空框，禁止替代歷史圖。
- **display_text**：「照片裡的人沒有失蹤。失蹤的是他們身後的地方。」「CASE 001／移位之地」。DATE、PHOTOGRAPHER、CAMERA POSITION、ORIENTATION、ORIGINAL HOLDER 均 UNKNOWN。
- **assets**：PHOTO-01；來源說明使用提供資料包的描述，URL 未提供時留空。
- **player_action**：展開來源與未知欄位，開始勘驗。
- **success_condition**：已開啟來源卡；記錄為 VIEWED。
- **failure_condition**：無失敗；缺影像不顯示下載錯誤堆疊。
- **hint_logic**：開始按鈕下固定提示「先分清楚照片告訴你什麼、沒告訴你什麼」。
- **transition**：`screen-photo`。
- **audio**：預設靜音；可選非史料性的輕微紙張聲，不自動播放。
- **mobile_behavior**：影像寬度隨容器；metadata 可摺疊，操作列不遮住內容。

### 02 — `screen-photo`

- **purpose**：觀察，不做定位。
- **visual_layout**：影像檢視器＋特徵籤＋觀察筆記；原圖和標記分層。
- **display_text**：「先說你看見了什麼。不要替照片補上地址。」特徵：ARCH、COLUMN、ROOF、ENTRANCE、STREET EDGE、UNCERTAIN；「THIS EVIDENCE DOES NOT PROVE：拍攝日期、相機位置、朝向」。
- **assets**：PHOTO-01；不使用目前資料夾任意照片自動冒名。
- **player_action**：縮放平移、點選影像加觀察點、選特徵、填一句可見依據；可撤銷／刪除。提供表單式新增標記供鍵盤使用。
- **success_condition**：保存至少一筆有依據的觀察，或明確選擇「影像缺失／無法辨識」。後者標為 UNRESOLVED，不偽裝為 ANALYZED 成功辨認。
- **failure_condition**：空筆記、點在圖外；顯示就地表單提示，不扣分。
- **hint_logic**：主動點「提示」先提示看構件，再提示區分觀察和推測；不預放「正解」熱點。
- **transition**：`screen-art`。
- **audio**：靜音，標記可有可關閉的低音量 click。
- **mobile_behavior**：圖片單指平移僅在操作模式；另有方向與放大按鈕；完成標記退出操作模式以恢復頁面捲動。

### 03 — `screen-art`

- **purpose**：重演視覺對應，不把藝術研究轉成坐標證明。
- **visual_layout**：PHOTO-01／ART-01 並列；手機分兩個標示清楚的圖卡，下方配對清單。禁止缺圖時以自畫拱門代替原畫。
- **display_text**：「簡綽然《夜之素描》（夜のスケッチ）」「1939 夏／紙本水彩／36.5 × 47 cm／第二回府展・西洋畫部・入選／國立臺灣美術館」。判讀說明：「研究者推測為桃園街消費市場，不是定位證明。」
- **assets**：ART-01，預設 LICENSE_REQUIRED＋PLACEHOLDER；PHOTO-01。
- **player_action**：先選照片特徵，再選畫作觀察，組成配對，附一句理由；最後選擇「相似特徵支持身分推測」或「足以得出精確坐標」。
- **success_condition**：有經研究者審核的對應表時才可顯示 `ACADEMIC CORRESPONDENCE MATCHED`。無對應表則 `CANDIDATE CORRESPONDENCE / REVIEW PENDING`；缺任一圖則記錄無法比對。正確理解證據限制即可繼續。
- **failure_condition**：選精確定位，提示「相似構件無法提供絕對坐標」；不刪除觀察。
- **hint_logic**：列出研究資料提及的拱形入口、人影、腳踏車、招牌／旗幟、電線桿、電燈，明示這是研究摘要而非 UI 新發現。
- **transition**：`screen-press`。
- **audio**：無。
- **mobile_behavior**：以點選配對替代必須拖曳；支援返回上一張圖而不丟失配對。

匹配狀態的 UI 文案為「Identity confidence ↑（定性）」與「Coordinate confidence: 0（無新增定位支持）」。不得產生虛構百分比或將 0 解釋為地理統計誤差。

### 04 — `screen-press`

- **purpose**：建立時間錨，保留年代衝突。
- **visual_layout**：兩張乾淨 metadata 卡、時間軸、未解衝突欄；不排成仿真報紙頁面。
- **display_text**：PRESS-01《臺灣日日新報》1932-05-28 第4版〈桃園市場一日より開場〉；「研究引用支持 6月1日起開場」。PRESS-02《臺灣新民報》1932年5月〈桃園　祝市場落成　賣酒贈景品〉，日／版次未提供。另列「1931 遷至現址？／UNRESOLVED CHRONOLOGY」。
- **assets**：兩份報刊 metadata；`Press_FullText=null`，無全文就不提供偽逐字轉錄。
- **player_action**：將「報導日期」「開場日期」「1931說法」分類到時間軸／未解區；用移動按鈕或選單亦可完成。
- **success_condition**：1932-05-28 和 1932-06-01 區分正確；PRESS-02 保持月精度；1931 留在未解欄。
- **failure_condition**：自行選「1931施工、1932開幕」，顯示「這是可能解釋，資料尚不足支持」，允許重試。
- **hint_logic**：第一次提示報導和事件不是同一天；第二次提示月精度資料不能補日期。
- **transition**：`screen-aerial`，年代衝突仍維持 UNRESOLVED。
- **audio**：無。
- **mobile_behavior**：垂直時間軸；觸控不需拖曳小卡。

### 05 — `screen-aerial` / LAYER B

- **purpose**：找第一代市場候選 footprint，不找水。
- **visual_layout**：1945 原航照、獨立筆記及候選範圍工具。不得含 1920 圖、分類、polygon 或縮圖。
- **display_text**：「只處理1945。先看地標、道路、屋脊與建築群。」「航照日期：1945-07-11（研究資料所載）」「MARKET POLYGON: PENDING」。
- **assets**：AERIAL-01；地標坐標只可用研究包注入的資料。未提供的景福宮坐標不畫點。
- **player_action**：查看影像、逐點畫候選範圍、填定位依據與不確定性，或選「無法定位」。PHOTO／ART 筆記可側欄開啟；不顯示 A 的結果。
- **success_condition**：合法候選範圍＋理由，或有原因的未解紀錄；提交後封存為 `LAYER B PROCESSED`。
- **failure_condition**：少於3點、自交、零面積或缺理由時阻止提交；缺圖禁用繪圖但允許缺件提交。
- **hint_logic**：只問「你使用哪個固定地標？」；不揭露答案輪廓。無審核 polygon 時不判玩家正誤。
- **transition**：`screen-hypothesis`。
- **audio**：無。
- **mobile_behavior**：點擊加點、確認閉合、撤銷上一點，另提供數值座標表；操作模式之外可正常捲動。

### 06 — `screen-hypothesis`

- **purpose**：區分命題和答案。
- **visual_layout**：一張引用摘要卡，不偽造新聞原件。
- **display_text**：「後世地方報導：1932年，在大廟後方填平埤塘，興建桃園消費市場。」「HYPOTHESIS／目前尚無1932當年報紙文字證明『填平埤塘』。」
- **assets**：後世說法摘要；來源 URL 留待資料提供。
- **player_action**：選此刻可接受的狀態：已證實／待驗證／已否定。
- **success_condition**：待驗證。
- **failure_condition**：其他選項顯示證據缺口，可立即重選。
- **hint_logic**：「被報導的說法，不等於已完成空間查核。」
- **transition**：`screen-map`。
- **audio**：無。
- **mobile_behavior**：三個至少44px高按鈕。

### 07 — `screen-map` / LAYER A

- **purpose**：1920 地貌分類，不找市場。
- **visual_layout**：1920 原圖、研究包提供的圖例／說明、分類及筆記。完全不渲染 B polygon、候選或地標匹配結果。
- **display_text**：「《桃園水利組合區域地形圖》／1:2,500」「圖層存在：VERIFIED；本區 pixel 判讀：PENDING」。類別：CLOSED WATERBODY、IRRIGATION CHANNEL、PADDY、BUILT-UP AREA、OTHER、UNCERTAIN。
- **assets**：MAP-01；無圖例時標示「圖例待提供」，不猜圖例。
- **player_action**：圈選候選區、分類、記錄圖上可見符號與判讀理由；可選 UNCERTAIN。
- **success_condition**：合法候選及理由，或缺件／不確定紀錄；封存 `LAYER A PROCESSED`。
- **failure_condition**：幾何無效或無理由；不因選水田而加分，也不暗示必有埤塘。
- **hint_logic**：提示查圖例與比例尺；沒有影像就直接提示資料不足。
- **transition**：A/B 均 processed 後開放 `screen-overlay`。
- **audio**：無。
- **mobile_behavior**：同航照，點选分类代替 hover；提交前顯示縮小預覽，不能出現 B 的輪廓。

### 08 — `screen-overlay`

- **purpose**：最後才看重疊；分清玩家候選與研究判定。
- **visual_layout**：GIS 檢視器＋A/B 開關、透明度、圖例、坐標與資料有效性。兩層缺資料時各自為空，不画共同假輪廓。
- **display_text**：「兩份獨立判讀，現在才能相遇。」「候選疊圖不等於研究定論。」缺件顯示 `INTERSECTION: NOT COMPUTABLE / VERDICT: PENDING`。
- **assets**：前兩關合法影像、transform、候選圖層、經審核研究範圍；不同來源用不同線型與文字標籤。
- **player_action**：先按「疊合已封存圖層」，調整透明度、查看研究前提清單，選擇目前能否下結論。
- **success_condition**：PENDING 時選 EVIDENCE INSUFFICIENT；資料完整時閱讀動態結果與限制。
- **failure_condition**：資料缺失仍選肯定結論，顯示缺少哪個條件；不捏造 intersection。
- **hint_logic**：指出缺 transform、影像、polygon 或研究簽核中的具體項目。
- **transition**：`screen-ending`。
- **audio**：可選輕微非敘事確認聲，不用恐怖突發音效。
- **mobile_behavior**：操作模式可平移／雙指縮放；外部工具列和頁面維持正常捲動；備有縮放、方向及重設按鈕。

### 09 — `screen-ending`

- **purpose**：結論與研究完成程度相符。
- **visual_layout**：結論、支持／不能證明的事項、缺件清單、玩家筆記匯出及繼續按鈕。
- **display_text**：PENDING：「你完成了調查程序，但還沒有足夠證據替歷史下結論。」「CASE STATUS: INSUFFICIENT EVIDENCE」。共同結尾：「你一直在找一個消失的地方。但城市很少突然消失。道路改了。地景變了。市場出現了。市場又搬走了。城市只是一層一層，把以前的自己蓋在下面。」
- **assets**：可選已授權 PHOTO REVEAL 03；缺圖不替代生成拆除照片。系列標語可放在片尾，但不能被當成本案未解空間命題的證明。
- **player_action**：匯出 JSON 筆記、返回工作桌、重新開始。
- **success_condition**：紀錄研究狀態與玩家理解；不以超自然劇情獎勵錯誤斷言。
- **failure_condition**：無；匯出失敗提供可複製文字。
- **hint_logic**：列出下一步需要什麼史料，而不是催玩家解鎖鬼故事。
- **transition**：繼續可回已解鎖畫面；重新開始需確認且先提供匯出。
- **audio**：可選淡出環境聲，必須有靜音及文字替代。
- **mobile_behavior**：原生下載不可用時用複製／分享文字，不清除進度。

## C. State Machine

將進度、證據狀態、研究判定分開。不要用一個 `verified=true` 同時代表玩過與史料查證。

|state|意義|允許轉換|
|---|---|---|
|LOCKED|前置步驟未完成|依 prerequisites → AVAILABLE|
|AVAILABLE|可調閱|開啟 → VIEWED|
|VIEWED|已看資料|保存觀察 → ANALYZED；缺件 → UNRESOLVED|
|ANALYZED|完成玩家分析|候選配對 → MATCHED；發現衝突 → DISPUTED|
|MATCHED|玩家記錄了對應|未獲研究審核仍不得升 VERIFIED；衝突 → DISPUTED|
|DISPUTED|存在證据冲突|保存待研究問題 → UNRESOLVED|
|UNRESOLVED|缺件或無法判定|資料更新後 → AVAILABLE 或 VIEWED，保留先前版本紀錄|
|VERIFIED|研究資料特定主張已審核|只能由資料更新及 researcher review 設定；撤回／變更後 → DISPUTED|

`playerState` 與 `claimStatus` 是不同欄位。研究層可標「圖層存在 VERIFIED」同時保留「目標地貌 PENDING」。

關卡完成用 `processed=true`、`outcome=OBSERVED|CANDIDATE|UNCERTAIN|DATA_PENDING`。A/B processed 不等於 A/B polygon 存在。疊圖解鎖公式為 `processed.A && processed.B`，計算允許性另外判斷。

## D. Evidence JSON Schema 與集中設定

核心資料可直接嵌在 `<script type="application/json" id="research-data">`；用 `JSON.parse(textContent)` 讀取，所有資料文字以 `textContent` 呈現。不得 `eval`、不得將資料中的 HTML 當程式。

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Evidence",
  "type": "object",
  "required": ["id","type","title","date","source","source_url","provenance_status","rights_status","visible_features","player_actions","supports","does_not_support","spatial_status","confidence","unlocks","assetMode"],
  "properties": {
    "id": {"type":"string"},
    "type": {"enum":["PHOTO","PRESS","ART","AERIAL","MAP","FICTION"]},
    "title": {"type":"string"},
    "date": {"type":["string","null"]},
    "date_precision": {"enum":["DAY","MONTH","YEAR","SEASON","UNKNOWN"]},
    "source": {"type":["string","null"]},
    "source_url": {"type":["string","null"]},
    "provenance_status": {"enum":["BRIEF_REPORTED","VERIFIED","CROSS_CHECK_NEEDED","UNKNOWN","FICTION"]},
    "rights_status": {"enum":["LICENSE_REQUIRED","LICENSED","UNKNOWN","NOT_APPLICABLE"]},
    "visible_features": {"type":"array","items":{"type":"string"}},
    "player_actions": {"type":"array","items":{"type":"string"}},
    "supports": {"type":"array","items":{"type":"string"}},
    "does_not_support": {"type":"array","items":{"type":"string"}},
    "spatial_status": {"enum":["UNKNOWN","CANDIDATE","VERIFIED"]},
    "confidence": {"enum":["PRIMARY","CONTEMPORARY","SECONDARY","INTERPRETIVE","PENDING"]},
    "unlocks": {"type":"array","items":{"type":"string"}},
    "assetMode": {"enum":["LOCAL_LICENSED","EXTERNAL_REFERENCE","PLACEHOLDER"]},
    "assetPath": {"type":["string","null"]}
  }
}
```

Schema URI 僅為版本識別字，不需連網解析。程式另須驗證：LOCAL_LICENSED 必須 rights_status=LICENSED、有合法相對路徑及來源紀錄；禁止絕對路徑穿越、remote image URL、javascript URL。EXTERNAL_REFERENCE 只顯示來源連結，不自動嵌入或抓取。

```json
{
  "schemaVersion": 1,
  "researchVersion": "brief-v1.0-pending",
  "Verdict": "PENDING",
  "Polygon_1920": null,
  "Polygon_1945": null,
  "Intersection": null,
  "registration": null,
  "review": {"status":"DATA_PENDING","reviewer":null,"reviewedAt":null},
  "Photo_Metadata": {
    "date":null,"photographer":null,"cameraPosition":null,
    "orientation":null,"originalHolder":null,"focalLength":null
  },
  "Press_FullText": {"PRESS-01":null,"PRESS-02":null},
  "fictionEnabled": false,
  "evidence": [
    {
      "id":"PHOTO-01","type":"PHOTO",
      "title":"第一代桃園消費市場迎賓門與人群",
      "date":null,"date_precision":"UNKNOWN",
      "source":"資料包記載：2019市場記憶展相關新聞翻攝",
      "source_url":null,"provenance_status":"BRIEF_REPORTED",
      "rights_status":"UNKNOWN","assetMode":"PLACEHOLDER","assetPath":null,
      "visible_features":["迎賓門","人群"],
      "player_actions":["ZOOM","ANNOTATE","RECORD_UNKNOWN"],
      "supports":["第一代市場建築外觀的比較"],
      "does_not_support":["精確拍攝日期","相機位置與朝向","第一代市場地理範圍"],
      "spatial_status":"UNKNOWN","confidence":"PENDING","unlocks":["screen-art"]
    },
    {
      "id":"MAP-01","type":"MAP","title":"桃園水利組合區域地形圖",
      "date":"1920","date_precision":"YEAR",
      "source":"Taoyuan_topomap_2.5K_1920","source_url":null,
      "provenance_status":"BRIEF_REPORTED","rights_status":"LICENSE_REQUIRED",
      "assetMode":"PLACEHOLDER","assetPath":null,"visible_features":[],
      "player_actions":["CLASSIFY","ANNOTATE","RECORD_UNCERTAIN"],
      "supports":["圖層存在，比例尺1:2,500（資料包已確認）"],
      "does_not_support":["未判讀區必有池塘","與市場有重疊"],
      "spatial_status":"UNKNOWN","confidence":"PENDING","unlocks":["screen-overlay"]
    }
  ]
}
```

其餘證物依 Asset Manifest 建立相同欄位，未知填 null，不填猜測的 URL。時間資料須保留原文、精度與引用層次，不能只存一個 JavaScript Date。

## E. DATA_PENDING Architecture

三層資料嚴格分離：

1. `researchData`：唯讀研究包、史料來源、合法資產、簽核範圍、結果。
2. `playerSave`：觀察點、候選範圍、理由、畫面進度；一律不是史料答案。
3. `viewState`：縮放平移、透明度、選取點；不得回寫地理 transform。

目前完整缺件清單必須包含：1920 pixel 判讀、Polygon_1920、1945 footprint、Polygon_1945、交集、PHOTO 原始收藏者、日期、攝影位置／方向、1931／1932 衝突、1932填埤原文。

沒有圖或授權時仍能調閱 metadata、檢查不能證明的事、建立時間錨、登錄不確定、查看最後缺件清單。繪製工具禁用；提供「無足夠資料可供圈選」操作讓流程完成。

更新研究包時執行 schema／幾何／授權校驗，再建立新的 `researchVersion`。玩家候選若舊影像或 transform 已變更，標 `STALE_REQUIRES_REVIEW`，不默默套新圖。新增研究資料不能自動把玩家操作變成正確答案。

## F. Final Overlay Engine

### 坐標與對位

- 原始 pixel annotation 使用影像自身坐標；未提供 georeferencing 就僅為 pixel，不顯示假的經緯度。
- 同一工作區必須有明確 `crs`、影像尺寸、有效範圍、研究提供的 affine/projective transform、控制點與殘差資訊。不同 CRS 不直接相交。
- 無依賴 P0 只接受研究者已轉換到同一平面坐標系的資料，不假装能處理任意投影。
- `registration` 缺失時，允許並排檢視，不允許聲稱對位；結果 PENDING。
- 平移縮放是同一 viewport transform。玩家不能靠移動底圖讓候選「吻合」後生成官方結果。
- 可選「實驗對位」另開 sandbox，持續標 `PLAYER ALIGNMENT / NOT RESEARCH`，結果不能寫入 Verdict。

### 幾何契約

研究 Polygon 使用 GeoJSON Polygon／MultiPolygon、閉合 ring、平面坐標單位公尺；含 `sourceEvidenceId`、`featureClass`、`researchVersion`、`reviewStatus`、`positionalUncertaintyM`。UNSPECIFIED uncertainty 不能當零誤差。

P0 採研究流程離線計算的 `Intersection`：需附兩個輸入幾何版本／摘要、演算法版本、審核資訊。瀏覽器執行一致性驗證與顯示，缺任何必要欄位回 PENDING。這避免以簡單凸多邊形裁切誤處理凹形、多島、洞或拓撲錯誤。

若後續加入瀏覽器 runtime intersection，必须通過凹形、多洞、多島、僅接邊、僅接點、自交拒絕、極小片段等單元測試，與已知離線算例一致，才可啟用；否則保留離線接口。不以包圍盒相交或圖層肉眼透明重疊代替 polygon intersection。

### 互動與數值

- pan、zoom、兩層開關、0–100% opacity、fit-to-data、reset view。
- 研究範圍實線、玩家候選虛線、交集斜線填充，文字標籤同步，不只靠顏色。
- 顯示 `areaA`、`areaB`、`intersectionArea` 及 `intersection/areaA`、`intersection/areaB`；分母零不得除，顯示無法計算。
- 不自行訂「10%即填埤」等歷史門檻；meaningful overlap、误差容忍及結果敘述由研究設定提供。
- 幾何的零交集只代表指定時間、資料與誤差模型下未相交。

### 動態 Verdict

`resolveVerdict()` 先驗資料與審核，再讀經審核的資料驅動結果；不是從候選的面積自動推導史實。

|設定|必要研究條件|顯示與限制|結案狀態|
|---|---|---|---|
|PENDING|缺任一關鍵資料／未審核|尚不能驗證此空間命題|INSUFFICIENT EVIDENCE|
|A / SPATIAL_OVERLAP_CONFIRMED|封閉水體辨識、第一代footprint、對位、實質重疊均審核|後世填埤說法獲空間旁證；不是施工過程的一手文字證明|CLOSED（本輪空間查核）|
|B / HYDRAULIC_LANDSCAPE_OVERLAP|水田／圳路／水利地貌與市場重合已審核|支持農業／水利地貌；不支持封閉埤塘精確描述|CLOSED（本輪空間查核）|
|C / NO_CARTOGRAPHIC_CORROBORATION|資料覆蓋與判讀充分，未辨識相關水體，已審核|1920未提供空間旁證；不能證明從未有埤塘|CLOSED（本輪空間查核）|
|D / SPATIAL_OFFSET_DETECTED|水體與市場各自確定，缺充分重疊，已審核|顯示偏移及誤差限制；不推論後代市場蓋入池塘|SUSPENDED（待跨期資料）|

幾何交集不決定「1931施工」或「地下暗道」等旁支敘事。資料矛盾時 INVALID_RESEARCH_PACKAGE，UI 回 PENDING 並列出需修正事項。

## G. Asset Manifest

狀態是多軸，不可將 VERIFIED 與 LICENSE_REQUIRED 視為互斥。每個原始資產保存來源、權利說明、版本及檔案校驗資訊。

|ID|資產／主張|認識狀態|權利／出廠顯示|
|---|---|---|---|
|PHOTO-01|第一代迎賓門及人群|資料包有影像描述；拍攝及來源鏈多項 UNKNOWN|UNKNOWN／PLACEHOLDER|
|ART-01|簡綽然《夜之素描》|作品資訊依資料包；市場身分 INTERPRETIVE|LICENSE_REQUIRED／PLACEHOLDER|
|PRESS-01|1932-05-28第4版報導metadata|一手報刊支持經研究引用轉述；全文 DATA_PENDING|metadata可顯示／影像權利待確認|
|PRESS-02|1932年5月落成報導metadata|條目存在；日、版次、全文 DATA_PENDING|同上|
|MAP-01|1920、1:2,500圖層存在|VERIFIED（依資料包）；目標區判讀 DATA_PENDING|LICENSE_REQUIRED／PLACEHOLDER|
|AERIAL-01|1945航照、研究記載07-11|存在；市場footprint DATA_PENDING|LICENSE_REQUIRED／PLACEHOLDER|
|PHOTO-02|邱垂龍2000年代攤商作品|選圖、來源鏈待整理|LICENSE_REQUIRED／可選、未授權不顯示|
|PHOTO-03|2021拆除影像|具體檔案、日期、來源待配對|DATA_PENDING／可選|
|FICTION-01|虛構人物訊息／筆記|FICTION，不是歷史证據|P2原創；預設關閉|
|BONUS-01|蘇清海家族資料|FAMILY ARCHIVE / CROSS-CHECK NEEDED|P3待提供；不製作假命盤|

本機既有 `4.jpg` 等檔名不具有證物身分；必須經 manifest 配對及授權確認才能指定為 PHOTO-01。保留原圖比例，可縮放但不修補建築內容；所有 UI 標記可隱藏，且不修改原影像。

## H. Responsive UX

- Desktop ≥1100px：最大1280px的鑑識桌；左證物索引、中圖像、右筆記。維持一步一任務。
- Tablet 768–1099px：圖像＋可切換側欄；不可讓縮圖過小取代主圖。
- Mobile <768px：單欄，主操作最少44×44px，建議48px；圖像工具列可換行，安全區留白。所有 hover 功能都有可點按對應。
- 全頁使用內容流；不以子容器 `height:100vh` 垂直置中。僅外層可設 min-height，地圖容器用 aspect-ratio 與最大高度。
- 進入地圖操作模式後僅 map surface 設 `touch-action:none`；離開後恢復 `pan-y`。工具列觸控不攔截頁面捲動；touchmove 只在 active gesture 時 preventDefault，passive:false。
- 旋轉或 resize 以 normalized viewport 保存焦點，不變更 annotation 原始坐標。

## I. Accessibility

- 以 button、fieldset、label、dialog、heading 等語意元素實作；畫面切換聚焦新 h1/h2，隱藏區使用 hidden／display:none，不留下可 Tab 元素。
- 所有拖曳均有點按／鍵盤等價；地圖方向鍵平移、+/-缩放、Home fit，且不攔截文字框輸入。標記編輯提供坐標表及刪除按鈕。
- aria-live=polite 公告保存、判讀結果及錯誤；不在每次 pan/zoom 廣播大量坐標。
- 一般文字對比 ≥4.5:1，介面界線／焦點 ≥3:1，狀態以文字＋形狀而非色彩區別。
- reduced-motion 停用印章彈跳、淡入、平滑自動移動；無閃爍、無突然強音。
- alt 描述可見內容與來源限制，不能把 PENDING 結論寫進 alt。缺圖 alt 明確說影像待提供。
- 音訊可全關，提供完整 transcript，標記 FICTION；靜音不影響過關。純視覺比對有「無法觀察／需替代資料」路徑，不能因此卡住。

## J. Save / Resume

- 存檔 key：`guiditu.ep01.save.v1`，保存 schemaVersion、researchVersion、updatedAt、screenId、evidenceStates、processedA/B、annotations、候選範圍、配對、時間軸、note、viewState、settings。
- 每次重要操作立即更新記憶體並 debounce 300ms 寫 localStorage；提交、畫面切換、visibilitychange(hidden)、pagehide 時 flush。不要只依賴 beforeunload。
- 不把圖片或大型研究 JSON 塞進 localStorage；筆記／點數設合理上限。Quota、安全例外、損毀 JSON 都 catch，維持可玩並顯示「此瀏覽器無法自動儲存，請匯出進度」。
- file:// 的 storage 行為依瀏覽器而異；提供進度 JSON 匯入／匯出作可靠備援，不承諾跨檔案／跨瀏覽器自動續玩。
- 重載顯示「繼續上次勘驗」和時間；只恢復合法且已解鎖 screenId。LINE切回触发 pageshow 检查，不重建或清空 session。
- 研究版本不一致時保留原筆記、標示待重審，不自動重算成已確認；匯入只接受玩家資料，不得覆寫研究 Verdict。
- 至少保存前一次有效備份；reset 前確認且給匯出選項。存檔完全留本機，不傳送追蹤或玩家坐標。

## K. Debug Mode

`?debug=1` 在 file:// 與 http 均可讀取，顯示可收合的唯讀面板：

- researchVersion／save version、current screen。
- 每件 evidence 的 playerState／claimStatus、關卡 unlock 理由。
- requested Verdict、effective Verdict、被降為 PENDING 的原因。
- 缺件及權利狀態、asset load/error、來源欄位完整性。
- 研究 polygon 和候選 polygon 分開列，含 CRS、版本、bounding box、點數、驗證錯誤及坐標 JSON。
- `Intersection` 資料版本／來源是否一致；目前全部 null 應正常顯示。

Debug 不提供「強制 VERIFIED」；開發者跳關只能在明確標為 TEST SESSION 的獨立存檔，不能污染正式進度或輸出研究結論。

## L. 施工優先順序與驗收

### P0 — 核心可玩

1. 研究 JSON、證物 model、guard functions、狀態機、版本與存檔。
2. 9畫面內容流、五種證物操作與可及性替代。
3. 缺件／未知流程及完整 PENDING 結局。
4. A/B互不洩露、processed gate、候選與研究資料分離。
5. 唯讀 debug、匯入／匯出、離線運行。
6. Overlay 檢視與經審核離線交集接口；未齊備時正確不計算。

### P1 — 鑑識質感

合法真實圖像、來源抽屜、原圖檢視、特徵標記、座標顯示與精準幾何驗證、桌面鑑識桌布局、錯誤復原。

### P2 — 動畫／聲音

輕微過場及紙張音效；prefers-reduced-motion與靜音同等體驗。研究結果未完成前不解鎖超自然空間吻合敘事。

### P3 — Bonus Archive

蘇清海家族支線，保留 CROSS-CHECK NEEDED、來源與權利資訊；最多三次大型照片展示，皆須真實資產授權。

### 必測驗收案例

|案例|預期|
|---|---|
|所有圖像未提供、polygon全null|全程不造圖，可到 INSUFFICIENT EVIDENCE|
|玩家把兩候選畫成完全重合|只能顯示玩家候選交集，正式 Verdict仍PENDING|
|只完成A或只完成B|不能開研究疊圖|
|A編輯時開DOM／debug以外一般介面|不得渲染B輪廓；B亦同|
|缺transform、混合CRS、未審核polygon|不計算研究交集、明列原因|
|A/B資料簽章或版本不合|拒絕舊Intersection，回PENDING|
|1931施工／1932開幕選項|不可被當成確認答案|
|ART特徵相似|不能解鎖地理坐標|
|C結果|結語不得出現「從來沒有埤塘」|
|D結果|不得推論第三代市場蓋入池塘|
|資料更新／損毀存檔／storage拒絕|可復原／匯出，不白屏、不丟掉已載入筆記|
|手機切去LINE、重載、旋轉|續玩；不改變幾何坐標|
|鍵盤完整流程／缺圖／reduced motion|均可完成，不需hover或聲音|
|離線＋file://|無網路依賴；外部參考連結不自動請求|

交付報告必須區分「流程已可玩」「研究資料已齊備」「歷史命題已驗證」三件事。不得因程式測試通過就宣稱後兩者完成。
