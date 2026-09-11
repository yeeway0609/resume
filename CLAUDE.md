# 專案說明

我的履歷原始碼，中英文各一份，用 XeLaTeX 排版，各自輸出單頁 A4 PDF。

| 版本 | 原始檔 | 產物 |
| --- | --- | --- |
| 英文 | `Resume_Alex_SoftwareEngineer.tex` | `output/Resume_Alex_SoftwareEngineer.pdf` |
| 中文 | `履歷_蘇奕幃_軟體工程師.tex` | `output/履歷_蘇奕幃_軟體工程師.pdf` |

原始檔名刻意與產物同名，這樣不必用 `-jobname`，LaTeX Workshop 也能直接找到對應的 PDF。

```
layout.tex          版面定義（邊界、顏色、巨集），兩版共用
*.tex               各自的字型設定與內容
icons/              圖示，svg 與 pdf 都進版控
output/             產物；中間檔已在 .gitignore
```

## 規則

**改內容一定要兩版一起改。** 中英文是同一份履歷的兩個版本，任何條目的新增、刪除、
改寫都要同步。只改一邊會讓兩版內容不一致。

**`.tex` 和 `layout.tex` 裡不要寫註解。** 所有說明寫在這個檔案裡。

**改完要確認還是一頁。** LaTeX 超出一頁不會有任何警告，會直接流到第二頁：

```bash
pdfinfo "output/Resume_Alex_SoftwareEngineer.pdf" | grep Pages
```

要壓回一頁時依影響力排序：`layout.tex` 的 `geometry` 邊界 → 內文 `\fontsize` 的字級 →
`\setlist` 的 `itemsep` → 砍掉一條 bullet（通常最有效，也最不傷版面）。

**版面數字集中在 `layout.tex`**，不要散在各處。

## 編譯

```bash
xelatex -synctex=1 -interaction=nonstopmode -output-directory=output "Resume_Alex_SoftwareEngineer.tex"
```

VS Code 裝 `James-Yu.latex-workshop`，設定在 `.vscode/settings.json`（未進版控），
存檔會自動重編並連帶重產 preview 圖。

環境是 TinyTeX，裝在 `~/Library/TinyTeX`，不需要 sudo。換機器要重裝：

```bash
curl -sL "https://yihui.org/tinytex/install-bin-unix.sh" | sh
tlmgr install enumitem ulem xecjk l3experimental accsupp
```

## Icons

`icons/` 裡的五個圖示取自 [Iconify](https://icon-sets.iconify.design/)：

| 檔案 | Iconify 名稱 | 高度 |
| --- | --- | --- |
| `graduation-cap` | `mdi:school` | 9pt |
| `email` | `famicons:mail` | 8pt |
| `phone` | `boxicons:phone-filled` | 10pt |
| `linkedin` | `brandico:linkedin-rect` | 10pt |
| `github` | `fa6-brands:square-github` | 10pt |

LaTeX 的 `\includegraphics` 不吃 SVG，向量格式只認 PDF，所以兩種格式都要進版控。
改了 SVG 之後要重新轉檔（需要 `brew install librsvg`）：

```bash
for f in icons/*.svg; do rsvg-convert -f pdf "$f" -o "${f%.svg}.pdf"; done
```

### 換圖示的步驟

1. 去 Iconify 挑一個下載
2. 把 `currentColor` 換成 `#111111`（`rsvg-convert` 不解析 `currentColor`）
3. 把 `viewBox` 裁到圖形的實際邊界，這樣 `height` 才等於圖形本身的高度，而不是含留白的外框
4. 重算高度並更新 `layout.tex` 的 `\seticonheight`
5. 轉成 PDF

高度不是隨手填的。圖示的長寬比不一，給同一個高度的話橫的會顯得太大。
基準是讓每個圖示的 `√(寬 × 高)` 接近 10pt，信封則再壓低到 8pt 讓寬度與其他圖示齊平。

## 字型

| 用途 | 字型 | 來源 |
| --- | --- | --- |
| 英文 | Arial | macOS 內建 |
| 中文 | Heiti TC（黑體-繁） | macOS 內建（`STHeiti Light/Medium.ttc`） |

**不要用 variable font。** `xdvipdfmx` 無法嵌入，會直接 `fatal: Invalid font: -1`
且不產生 PDF。Homebrew 的 `font-roboto-slab`、Google Fonts 的 Noto TC 都是。

`PingFang.ttc` 在新版 macOS 上不存在（已改成按需下載的資產），所以中文用 `Heiti TC`。

中文粗體用 `BoldFont={Heiti TC Medium}` 這個真實字重。`AutoFakeBold=2` 的假粗體太淡，
內文 10pt 幾乎看不出來。

Noto Sans CJK 也不要用：`NotoSansCJK.ttc` 是 45 個 face 的合集，fontconfig 在 macOS 上
不管用家族名還是 PostScript 名稱都會解析到 JP face，要拿到 TC 得寫死字型檔絕對路徑。

## 中文版的行內連結

xeCJK 會把 `\ulink` 這種盒裝內容當成西文，在兩側自動插入中西文字距。
如果連結後面**緊接中文字**，就會多出一個明顯的空隙：

```
開發\ulink{...}{銀行網站}的會員點數    →   開發銀行網站 的會員點數
```

所以中文版的行內連結一律安排在**標點符號前**收尾：

```
...並實作輸入驗證，應用於\ulink{...}{銀行官方網站}。
```

## 連結樣式

| 巨集 | 樣式 | 用在 |
| --- | --- | --- |
| `\plink` | 藍字 | 右上角聯絡方式 |
| `\blink` | 藍字粗體 | 公司名 |
| `\ulink` | 藍字加底線 | 內文中的連結 |
