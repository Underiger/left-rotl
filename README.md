# 左旋（簡報版）

計算機組織第十五號題目 ROTL：替 RISC-V 設計一條自訂指令「循環左移」。

這個分支是**上台報告、放進作業用的白話版**。它用中共改革開放時期的口號，把 CP1–CP3 的技術重點串起來。完整的架空歷史小說版在 [`main` 分支](https://github.com/Underiger/left-rotl/tree/main)。

- 簡報：[`簡報.md`](簡報.md)
- 報告：[`report/CP1.md`](report/CP1.md)、[`report/CP2.md`](report/CP2.md)、[`report/CP3.md`](report/CP3.md)

## 為什麼是中共

ROTL 是 rotate **left**。鄧小平 1992 年南巡講話說：「中國要警惕右，但主要是防止『左』。」一條叫「左」的指令，剛好碰上一條「主要防左」的路線。所以簡報裡，每一個技術決定都配了一句中共的口號：

| 技術決定 | 配的口號 |
|---|---|
| 以測資判斷對錯，n = 0 要先判斷 | 實踐是檢驗真理的唯一標準 |
| 不管程式是人寫還是 AI 寫，過測資就好 | 不管黑貓白貓 |
| 無分支版本 5 條，比含分支版平均 7.84 條少 | 貧窮不是社會主義 → 分支不是精簡指令集 |
| 兩個組合語言版本都留，一個當對照 | 兩手抓，兩手都要硬 |
| 用 custom-0 自訂編碼，不直接用 Zbb 的 `rol` | 韜光養晦，決不當頭 |
| 動態指令數少 18.9%–42.3% | 發展才是硬道理 |

完整對照表在簡報最後一頁。

## 指令

```
rotl rd, rs1, rs2      n = rs2 & 31;  rd = n == 0 ? rs1 : (rs1 << n) | (rs1 >> (32 − n))
```

R-type，opcode 用 custom-0（`0001011`）。funct3、funct7 等教師分配，現在是暫定值（見 [`spec/rotl_isa.h`](spec/rotl_isa.h)）。

## 怎麼跑

需要 `cc`、`python3`（3.10 以上）、`riscv64-elf-gcc`。在 repo 根目錄執行：

```bash
make test      # C 測試 + UBSan + 22 項 Python 測試（含模擬器）
make counts    # 重新量指令數，寫到 report/figures/
make encode    # 印出 rotl 的機器碼範例
make asm       # 產生編譯器輸出的組合語言
make ub-demo   # 反例：沒處理 n = 0 的寫法（預期會報錯）
make verify    # 效能驗證（ARM64 實機量測只在 ARM64 電腦上跑）
```

## 結果

- C 參考實作：16 組測資、69,000 次交叉比對、UBSan 全部通過
- RV32IM：手寫無分支版 4 條（不含 `ret`），GCC `-O2` 5 條，自訂指令 1 條
- 動態指令數（模擬器實測，GCC 16.2.0 `-O2`）：

| 核心 | RV32IM | ROTL | IC 比 |
|---|---:|---:|---:|
| ChaCha20 區塊 | 1919 | 1284 | 1.49x |
| RC5 金鑰排程 | 2467 | 2000 | 1.23x |
| RC5 加密一個區塊 | 227 | 131 | 1.73x |

- 效能驗證（[`experiments/`](experiments/README.md)）：IC 用反組譯手算交叉驗證一致；5 級 pipeline 模型的 cycle 比 1.21x–1.67x；ARM64 實機用 `ror` 快 1.29x–1.52x（只算旁證）。前提是 ROTL 不拉長 clock period，要到 CP4 才能確認

## 目錄

```
c_baseline/    CP1：C 參考實作與測試
tests/         16 組測資 CSV
rv32im/        CP2：RV32IM 組合語言、密碼學核心
spec/          CP3：語意規格、編碼；rotl_isa.h 是編碼常數唯一的設定來源
tools/         黃金模型、RV32IM 模擬器、量測腳本、Python 測試
experiments/   效能驗證：IC 交叉驗證、5 級 pipeline 模型、ARM64 實機量測
references/    參考資料
report/        CP1–CP3 報告與數據表
簡報.md        上台用的精華版
```

## 簡報怎麼匯出

`簡報.md` 是 [Marp](https://marp.app/) 格式，用 `---` 分頁。可以用 VS Code 的 Marp 擴充套件預覽和匯出，或在命令列執行：

```bash
npx @marp-team/marp-cli 簡報.md --pptx
```

## 說明

- 改編自 [Underiger/rotl](https://github.com/Underiger/rotl)。報告第六節的組員姓名已經拿掉，交作業前請自己補上
- AI 使用說明：程式與報告初稿由 Claude Code（Claude Opus 5.5）產生；這個分支的 README 和簡報也是 Claude Code 依原專案改寫的
