# 附件一　C 參考實作

- 對應：CP1；正文第一幕
- 依據：作業規定第六節第四款（第十五號題目，第七頁）；小組討論紀要（二〇二六年十月八日）第三條

| 檔案 | 內容 |
|---|---|
| `rotl.h`／`rotl.c` | C 參考實作 `uint32_t rotl_ref(uint32_t rs1, uint32_t rs2)`，與規格逐字對應，保留 `n == 0` 之分支 |
| `test_rotl.c` | 主機測試：CSV 測資、與逐位旋轉寫法交叉比對（以程式輸出為準，計 69,000 次）、性質檢查 |
| `ub_demo.c` | 反例：未處理 `n = 0` 的寫法。UBSan 下報出移位 32 的未定義行為；不開 UBSan 時，本機輸出「剛好算對」 |

以下指令在倉庫根目錄執行：

```bash
make test-c       # 編譯並執行 test_rotl
make test-ubsan   # 同一組測試加上 -fsanitize=undefined
make ub-demo      # 反例：n = 0 時預期報錯，make 的結束碼仍為 0
```

`rotl.c` 另由 RISC-V 工具鏈編譯，作為 CP2 的「編譯器基準」（見 [`../rv32im/vectors_main.c`](../rv32im/vectors_main.c)）。
