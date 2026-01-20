# 強震儀響應分析系統 (Pole-Zero Response Analyzer)

## 專案概述

本專案提供台灣地震觀測網強震儀儀器響應的互動式分析工具，包含 SAC Pole-Zero 格式檔案解析與視覺化網頁應用程式。透過本工具，使用者可以深入理解地震儀器的頻率響應特性、Pole-Zero 圖，以及儀器參數的物理意義。

### 主要功能

- 📊 **互動式 Bode Plot**：即時顯示儀器的振幅響應與相位響應
- 🗺️ **Pole-Zero 圖**：在 S-平面上視覺化極點與零點的分布
- 🔄 **雙模式切換**：可切換查看位移響應與加速度響應
- 📱 **響應式設計**：支援桌面與行動裝置瀏覽
- 🎯 **懸停數值顯示**：移動游標即可查看任意頻率的響應數值

### 測站資訊

- **測站代碼**：TW.CHK.10.HLZ
- **測站名稱**：成功氣象站 (Cheng-Gong Weather Station)
- **儀器型號**：SMART24A 地震紀錄器
- **感測器**：PA-2 加速度感測器 (內建)
- **位置**：北緯 23.098°，東經 121.373°，海拔 37 公尺
- **取樣率**：100 Hz

---

## 快速開始

### 方法一：直接開啟網頁

在瀏覽器中開啟 `index.html` 檔案即可使用互動式分析工具，無需安裝任何相依套件。

```bash
# 使用預設瀏覽器開啟
open index.html  # macOS
xdg-open index.html  # Linux
start index.html  # Windows
```

### 方法二：使用本地伺服器

若需要更好的開發體驗，可使用本地 HTTP 伺服器：

```bash
# 使用 Python 3
python3 -m http.server 8000

# 或使用 Node.js
npx http-server -p 8000
```

然後在瀏覽器開啟 `http://localhost:8000`。

---

## SAC Pole-Zero 檔案格式說明

### 檔案結構

本專案使用的 SAC (Seismic Analysis Code) Pole-Zero 檔案包含以下主要資訊：

```
SAC_PZs_TW_CHK_HLZ_10_2018.344.00.00.00.0000_2599.365.23.59.59.99999
```

#### Metadata（元資料區塊）

位於檔案開頭的星號註解區，包含測站與儀器的完整資訊：

| 欄位 | 說明 | 範例值 |
|------|------|--------|
| NETWORK | 地震網代碼 | TW (台灣) |
| STATION | 測站代碼 | CHK (成功) |
| LOCATION | 位置代碼 | 10 |
| CHANNEL | 通道代碼 | HLZ (垂直向高增益) |
| LATITUDE/LONGITUDE | 測站座標 | 23.098°N, 121.373°E |
| SAMPLE RATE | 取樣率 (Hz) | 100.0 |
| INPUT UNIT | 輸入單位 | M (公尺，位移) |
| OUTPUT UNIT | 輸出單位 | COUNTS (數位計數值) |
| INSTTYPE | 儀器型號 | SMART24A + PA-2 |
| SENSITIVITY | 靈敏度 | 3.121640×10⁵ |
| A0 | 正規化因子 | 2.313560×10⁹ |

#### Zeros（零點）

定義轉換函數的零點，在 S-平面上的位置：

```
ZEROS   3
    +0.000000e+00   +0.000000e+00   # 零點 1 (原點)
    +0.000000e+00   +0.000000e+00   # 零點 2 (原點)
    -3.333000e+03   +0.000000e+00   # 零點 3 (實軸)
```

#### Poles（極點）

定義轉換函數的極點，決定系統的頻率響應特性：

```
POLES   4
    -7.420000e+02   +1.014000e+03   # 極點 1 (複數對)
    -7.420000e+02   -1.014000e+03   # 極點 2 (複數對的共軛)
    -8.663000e+02   +0.000000e+00   # 極點 3 (實軸)
    -5.638000e+03   +0.000000e+00   # 極點 4 (實軸)
```

#### Constant（總增益常數）

```
CONSTANT    +7.222102e+14
```

這個值等於 A0 × Sensitivity，用於將數位計數值轉換回物理單位。

---

## 物理意義深度解析

### 1. 為什麼輸入單位是「位移」而非「加速度」？

雖然 PA-2 是一台**加速度計 (Accelerometer)**，但在地震學的標準作法中，所有儀器響應都統一以**地面位移 (Ground Displacement)** 為輸入基準。這樣做的好處是：

- ✅ 不同類型儀器（地震儀、強震儀）可以統一處理
- ✅ 便於進行儀器響應去除 (Instrument Response Removal)
- ✅ 符合 SEED/FDSN 國際標準

因此，Pole-Zero 的配置必須包含「將位移轉換為加速度」的數學運算。

### 2. 零點 (Zeros) 的物理意義

#### 兩個原點零點：位移→加速度的轉換

```
s = 0 + 0i  (出現兩次)
```

在拉普拉斯域中，這兩個零點對應到 **s² 項**，代表**二次微分**：

$$
\text{加速度} = \frac{d^2(\text{位移})}{dt^2}
$$

這個數學運算的物理意義是：
- 第一次微分：位移 → 速度
- 第二次微分：速度 → 加速度

因此，儀器對**靜態位移**（低頻或 DC 分量）的響應為零，但能正確記錄**加速度變化**。

#### 第三個零點：高頻補償

```
s = -3333 + 0i
```

這個零點位於頻率約 **530 Hz** 的位置：

$$
f = \frac{|-3333|}{2\pi} \approx 530 \text{ Hz}
$$

這個頻率遠超過地震波的主要能量範圍（通常 < 50 Hz），主要用於：
- 🔧 電子電路的高頻響應微調
- 🔧 補償感測器的物理特性
- 🔧 確保整體轉換函數的穩定性

### 3. 極點 (Poles) 的物理意義

#### 複數極點對：決定儀器頻寬

```
s = -742 ± 1014i
```

這對共軛複數極點是系統最重要的特徵，決定了儀器的**轉角頻率 (Corner Frequency)**：

$$
\omega_n = \sqrt{742^2 + 1014^2} \approx 1257 \text{ rad/s}
$$

$$
f_n = \frac{\omega_n}{2\pi} \approx 200 \text{ Hz}
$$

**物理意義**：
- 在 0.01 Hz 到 200 Hz 之間，儀器對加速度的響應非常平坦 (Flat)
- 超過 200 Hz 後，響應開始快速衰減
- 這確保了儀器能忠實記錄地震波（主要能量在 0.1-20 Hz），同時濾除高頻雜訊

**與寬頻地震儀的對比**：
- 🌊 寬頻地震儀：轉角頻率約 0.008 Hz（週期 120 秒），適合記錄長週期波動
- ⚡ 強震儀：轉角頻率約 200 Hz，適合記錄高頻強烈震動

#### 實數極點：抗混疊濾波

```
s = -866.3        (138 Hz)
s = -5638         (897 Hz)
```

這些極點提供額外的高頻濾波，主要功能是：
- 🛡️ **抗混疊 (Anti-aliasing)**：防止高於取樣率一半（Nyquist 頻率 = 50 Hz）的訊號造成失真
- 🛡️ **電子電路特性**：反映類比濾波器的物理限制
- 🛡️ **雜訊抑制**：確保輸出訊號的品質

### 4. 靈敏度與增益常數

#### Sensitivity（靈敏度）

```
3.121640×10⁵ counts/(m/s²)
```

這個數值表示：當地面加速度為 **1 m/s²** 時，儀器輸出約 **312,164 counts**。

#### A0（正規化因子）

```
2.313560×10⁹
```

這是在參考頻率（通常是 1 Hz）下，為了使轉換函數正規化所需的係數。

#### CONSTANT（總增益）

```
7.222102×10¹⁴ = A0 × Sensitivity
```

這個常數用於將 Pole-Zero 轉換函數完整地應用到實際資料上。在訊號處理流程中：

$$
\text{物理單位} = \frac{\text{數位計數值}}{\text{轉換函數}(\omega) \times \text{CONSTANT}}
$$

---

## 技術實作說明

### 前端架構

本專案的網頁應用程式採用現代化的前端技術棧：

#### 核心技術

- **React 18**：使用 Hooks（useState, useEffect, useMemo）進行狀態管理
- **Tailwind CSS**：響應式設計與快速樣式開發
- **Babel Standalone**：瀏覽器端即時編譯 JSX
- **無相依性 SVG**：自行實現圖表繪製，無需外部圖表函式庫

#### 關鍵演算法

##### 1. 複數運算類別

```javascript
class Complex {
    constructor(re, im) {
        this.re = re;  // 實部
        this.im = im;  // 虛部
    }
    
    // 複數加法、減法、乘法、除法
    add(c) { ... }
    sub(c) { ... }
    mult(c) { ... }
    div(c) { ... }
    
    // 極座標表示
    mag() { return Math.sqrt(this.re² + this.im²); }
    phase() { return Math.atan2(this.im, this.re); }
}
```

##### 2. 頻率響應計算

對於每個頻率點 $f$，計算轉換函數 $H(s)$：

$$
H(s) = \text{CONSTANT} \times \frac{\prod_{i}(s - z_i)}{\prod_{j}(s - p_j)}
$$

其中 $s = j\omega = j \cdot 2\pi f$

```javascript
const s = new Complex(0, 2 * Math.PI * freq);

let numerator = new Complex(CONSTANT, 0);
ZEROS.forEach(z => {
    numerator = numerator.mult(s.sub(z));
});

let denominator = new Complex(1, 0);
POLES.forEach(p => {
    denominator = denominator.mult(s.sub(p));
});

const H = numerator.div(denominator);
const magnitude = H.mag();
const phase = H.phase() * (180 / Math.PI);
```

##### 3. 對數刻度繪圖

由於頻率範圍跨越多個數量級（0.01 Hz 到 1000 Hz），使用對數刻度：

```javascript
// X軸：對數刻度頻率
const xMin = Math.log10(0.01);
const xMax = Math.log10(1000);
const getX = (freq) => {
    const logFreq = Math.log10(freq);
    return padding.left + ((logFreq - xMin) / (xMax - xMin)) * plotWidth;
};

// Y軸：對數刻度振幅
const getY = (magnitude) => {
    const logMag = Math.log10(magnitude);
    return height - padding.bottom - 
           ((logMag - yMin) / (yMax - yMin)) * plotHeight;
};
```

### 使用者互動功能

#### 1. 模式切換

使用者可以在兩種響應模式間切換：

- **位移響應模式**：顯示 $H_{\text{disp}}(\omega)$，單位為 counts/m
- **加速度響應模式**：顯示 $H_{\text{acc}}(\omega)$，單位為 counts/(m/s²)

加速度響應透過除以 $\omega^2$ 從位移響應轉換而來：

$$
H_{\text{acc}}(\omega) = \frac{H_{\text{disp}}(\omega)}{\omega^2}
$$

#### 2. 懸停游標追蹤

當使用者將滑鼠移動到圖表上時，系統會：
1. 計算滑鼠位置對應的頻率值
2. 從預先計算的資料點中插值
3. 即時顯示該頻率的振幅與相位

---

## 應用情境與範例

### 情境一：地震訊號處理

在分析強震記錄時，需要將儀器記錄的數位計數值還原為真實的地面運動參數：

```python
# 使用 ObsPy 處理地震資料
from obspy import read

# 讀取波形資料
st = read('TW.CHK.10.HLZ.mseed')

# 去除儀器響應，轉換為加速度 (m/s²)
st.remove_response(
    inventory=inv,
    output='ACC',  # 輸出加速度
    water_level=60  # 穩定化參數
)
```

本專案提供的 Pole-Zero 檔案可以直接用於 ObsPy、SAC 等地震分析軟體。

### 情境二：儀器選型與規劃

假設要在某地區建置地震觀測站，需要選擇合適的儀器：

| 應用目標 | 建議儀器類型 | 轉角頻率 | 動態範圍 |
|----------|--------------|----------|----------|
| 遠震研究 | 寬頻地震儀 | 0.008 Hz (120s) | ±140 dB |
| 近震分析 | 短週期地震儀 | 1 Hz | ±120 dB |
| **強震監測** | **強震儀 (FBA)** | **100-200 Hz** | **±110 dB** |
| 地動觀測 | 甚寬頻地震儀 | 0.001 Hz (1000s) | ±150 dB |

本專案分析的 SMART24A/PA-2 屬於**強震監測**類別，特別適合：
- ⚡ 近震源的強烈震動記錄
- 🏢 結構物震動監測
- 🚨 地震預警系統
- 📊 工程地震學研究

### 情境三：教學與訓練

本互動式工具可用於：

1. **地震學課程**：展示儀器響應的概念
2. **訊號處理教學**：理解 Pole-Zero 與轉換函數
3. **工程地震學訓練**：認識強震儀的特性
4. **研究方法工作坊**：實際操作儀器響應分析

---

## 常見問題 (FAQ)

### Q1：為什麼加速度響應在低頻會上升？

**A**：這是**視覺錯覺**。實際上，加速度響應在觀測頻帶內（0.01-100 Hz）是平坦的。圖表看起來上升是因為使用對數刻度，而數值本身在低頻確實較小，但這不影響儀器記錄加速度的準確性。

### Q2：轉角頻率 200 Hz 是不是太高了？

**A**：不會。雖然地震波的主要能量通常在 0.1-20 Hz，但：
- 近震源的強震記錄可能含有高頻分量（20-50 Hz）
- 高轉角頻率確保在觀測頻帶內響應完全平坦
- 這避免了相位失真，保證波形的忠實度

### Q3：可以用這個 Pole-Zero 檔案處理其他測站的資料嗎？

**A**：**不可以**。每個測站、每個儀器都有獨特的響應特性。使用錯誤的響應檔案會導致：
- ❌ 振幅計算錯誤
- ❌ 相位失真
- ❌ 頻譜扭曲
- ❌ 科學結論錯誤

務必使用與資料對應的正確響應檔案。

### Q4：Pole-Zero 格式與 RESP 格式有什麼差別？

| 特性 | Pole-Zero (PZ) | RESP (Response) |
|------|----------------|-----------------|
| 格式 | SAC 專用，簡潔 | SEED 標準，詳細 |
| 內容 | Poles、Zeros、Constant | 包含多級放大、濾波器等 |
| 適用軟體 | SAC、ObsPy、SeisComP | 所有地震學軟體 |
| 精確度 | 適用於大多數情況 | 最完整精確 |

本專案使用 PZ 格式是因為它簡潔易讀，適合教學與快速分析。

### Q5：如何驗證儀器響應計算是否正確？

可以使用以下方法驗證：

1. **檢查平坦頻帶**：加速度響應在 0.1-50 Hz 應該接近常數
2. **比較相位**：加速度響應的相位應該在平坦區接近 0°
3. **使用標準軟體**：與 ObsPy、SAC 的計算結果比對
4. **單位檢查**：確保輸入/輸出單位的維度正確

---

## 參考資料與延伸閱讀

### 官方文檔

- [IRIS DMC: Instrument Response](https://ds.iris.edu/ds/nodes/dmc/data/formats/resp/)
- [SAC Manual: Instrument Response](http://ds.iris.edu/files/sac-manual/)
- [FDSN StationXML Format](https://www.fdsn.org/xml/station/)

### 學術文獻

1. **Bormann, P.** (2012). *New Manual of Seismological Observatory Practice (NMSOP-2)*. IASPEI, GFZ German Research Centre for Geosciences.
   - 第四章：地震儀器與響應

2. **Scherbaum, F.** (2001). *Of Poles and Zeros: Fundamentals of Digital Seismology*. Springer.
   - 經典教科書，深入探討 Pole-Zero 理論

3. **Havskov, J., & Alguacil, G.** (2016). *Instrumentation in Earthquake Seismology*. Springer.
   - 涵蓋現代地震儀器的完整介紹

### 線上資源

- [ObsPy Tutorial](https://docs.obspy.org/tutorial/): Python 地震學分析工具
- [IRIS Synthetics Engine](http://service.iris.edu/irisws/syngine/1/): 合成地震圖計算
- [台灣中央氣象署地震測報中心](https://www.cwa.gov.tw/V8/C/E/index.html): 台灣地震觀測網資訊

### 相關課程

- MIT OpenCourseWare: *Introduction to Seismology*
- Coursera: *Geophysical Data Analysis*
- IRIS Education and Outreach: *Seismology Skill Building Workshop*

---

## 授權與貢獻

### 資料來源

- Pole-Zero 檔案由中央氣象署提供
- 測站資料來自台灣地震觀測網 (TW Network)

### 開源授權

本專案程式碼採用 MIT License，歡迎自由使用、修改與分享。

### 如何貢獻

歡迎透過以下方式貢獻：
- 🐛 回報問題或錯誤
- 💡 提出新功能建議
- 📝 改善文件說明
- 🔧 提交程式碼修正

---

## 聯絡資訊

如有任何問題或建議，歡迎聯繫：

- **專案作者**：oceanicdayi
- **GitHub**：[oceanicdayi/pole_zero](https://github.com/oceanicdayi/pole_zero)

---

**最後更新**：2026 年 1 月 20 日
