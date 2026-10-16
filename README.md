# Syrtis-DSH（魔改版）— 已收尾，fork 停止維護

> DSH 用量解析在 2026-10-16 官方 [v1.2.0](https://github.com/Nanako0129/Syrtis-Windows/releases/tag/v1.2.0)
> 就內建了，新裝請直接用官方版。這個 repo 留作紀錄，不再發版。
> 原始碼 MIT，原作者 Nanako Tsai（[@Nanako0129](https://github.com/Nanako0129)），
> 這個 fork 只是在官方版本上補 DSH，感謝他願意合併引擎 PR 並自己完成 consumer 端註冊。

## 時間線

| 日期 | 事件 |
|---|---|
| 2026-10-11 | [tokscale-core#66](https://github.com/Nanako0129/tokscale-core/pull/66) 合併，merge commit `2d64ec4`，DSH client 進官方引擎（`ClientId::Dsh = 41`） |
| 2026-10-11 | 本 repo 發 `dsh-v1.0.0-magmod2`，給官方升 pin 前的使用者 |
| 2026-10-16 | 官方 [Syrtis-Windows#250](https://github.com/Nanako0129/Syrtis-Windows/pull/250) 合併，引擎 pin 推到 `2d64ec42`、註冊 DSH client |
| 2026-10-16 | 官方 **v1.2.0** 發布，release notes 寫 "Thanks @kizaki-R for the engine support (tokscale-core#66)" |

官方 v1.2.0 的分頁顏色是 `#001259`（深藍，跟 macOS 共用），icon 是 DSH 桌面版本體圖，
跟這個 fork 當初的 `#0f766e` 鯨魚標不同 ── 官方版是對的。

## 還留著的東西：影子價範本

`custom-pricing.json.TEMPLATE` 是唯一沒有進官方版的部分。引擎本身
（`src/pricing/custom.rs`）在官方 `2d64ec4` 就有讀 `custom-pricing.json` 的能力，
所以這個檔案**配官方 v1.2.0 也能用**，不必再裝這個 fork：

1. 複製 [`custom-pricing.json.TEMPLATE`](custom-pricing.json.TEMPLATE) 改名成 `custom-pricing.json`。
2. 放到 `%APPDATA%\tokscale\`（一般是 `C:\Users\<你>\AppData\Roaming\tokscale\`）。
3. 重啟 Syrtis。

31 個 `-free` 模型的 input / output / cache_read 都填了付費版影子價（$/1M），
免費模型就不會再顯示 `—`。
`north-mini-code-free`、`ring-2.6-1t-free`、`space-bunny-free` 找不到付費價，不在表裡，只看 tokens。

## 這份解析怎麼算出來的

讀的是 DSH 追加的 zstd 事件紀錄，不是 projcache 快照
（`~/.dsh/storages/session_projcache/`）。快照在不同安裝上帶的欄位不一致，
同一個 DSH 版本這台有 `contextTimeline` 那台沒有，沒法當基準。

每個 `assistant/message` 事件就是一個完成的 step，自帶 usage、provider / model 和自己的時間戳，
所以中途換模型或跨幾天會分開算。

四個在審查中實測出來、寫進 `UPSTREAM.md` 的處理：

- **seeded session 不重複計**：子 session 開頭是 parent 前 N 個事件的逐字複製，dedup key 不含 session id 也不含 `seq`，複製的 step 與原檔 key 相同，只算一次。
- **繼承的 step 歸 parent**：index 小於 `inheritedEventCount` 的事件改用 header 的 `parentSession` 當 session id，parent 的 log 不在也照樣算。
- **subagent 各自一個 session**：subagent 存在裸 UUID 目錄，格式 `session-*.jsonl.zstd` 掃不到，現在各算各的。
- **`compaction/summary` 有計**：DSH 自己的總數不含它，但那是真的花掉的 token。維護者同意保留。

per-event 的用量加總等於快照裡的 `tokenUsage.val.totals`（兩台機器、12 個 session 加 2 個 subagent 都對得上），
例外是上面那 2 次 compaction，會多出來。

## 授權

MIT，見 `LICENSE`。`THIRD-PARTY-NOTICES.md` 隨包發佈（官方打包腳本的要求）。