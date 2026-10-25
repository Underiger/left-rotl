# 附件三　RV32IM 實作與密碼學核心

- 對應：CP2；正文第二幕
- 規定：只使用 RV32IM 指令（`-march=rv32im -mabi=ilp32`）。自訂指令只出現在標示為 custom 的版本

| 檔案 | 內容 |
|---|---|
| `rotl.S` | 三個手寫版本：`rotl_branch`（版本 A，逐字翻譯，含分支）、`rotl_branchless`（版本 B，無分支）、`rotl_custom`（自訂指令） |
| `vectors_main.c` | 把測資送進 C 參考實作與三個手寫版本，結果存到全域陣列 |
| `kernels.c`／`kernels.h` | ChaCha20 區塊函式與 RC5-32/12/16，以 `USE_CUSTOM_ROTL` 切換標準版與自訂指令版 |
| `kernels_main.c` | 以 RFC 8439 與 RC5 論文的測試向量呼叫核心 |
| `start.S` | 設定堆疊、呼叫 `main`、`ebreak` |
| `link.ld` | 連結腳本：程式從位址 0 開始 |
| `runtime.c` | `memcpy`／`memset`（沒有標準函式庫時，GCC 仍可能呼叫） |

版本 A 與版本 B 同一次提交進倉庫：A 照第七頁逐字翻譯，作為對照；B 依 RISC-V 手冊 §2.4（`sll`／`srl` 只取 rs2 的低 5 位元），為採用版本。兩手抓，兩手都要硬。

建置與執行由 `tools/` 的 Python 腳本處理：

```bash
make test-py   # 在模擬器上驗證正確性與指令數（380 組輸入，四個版本）
make counts    # 產生 report/figures/instruction_counts.md
make asm       # 產生 build/rotl_O0.s、build/rotl_O2.s（編譯器輸出）
```

## 在 Ripes 中執行

`rotl.S` 的 `rotl_branch` 與 `rotl_branchless` 只用 RV32I 指令，可以把函式本體貼進 Ripes，在前面加上設定 `a0`、`a1` 的指令，觀察暫存器變化。

Ripes 不認得 custom-0，`rotl_custom` 無法在 Ripes 執行。系上 Ripes 加入 custom-0 的申請單已於九月二十六日送出，教學設備小組擬於下學期統一更新。申請單不撤；在此以前，自訂指令一律在本組模擬器（[`tools/rv32sim.py`](../tools/rv32sim.py)）上驗證。
