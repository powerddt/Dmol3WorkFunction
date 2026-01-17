# DMol3 Work Function  / DMol3 功函数后处理

这是一个轻量级 Python 脚本，用于从 **DMol3** 结果中计算功函数：
- 静电势网格文件：`*.grd`
- 费米能级：`*.outmol`

输出内容包括：
- 平面平均静电势 `V(z)`
- 自动识别真空平台 `V_vac`
- 功函数：**Φ = V_vac − E_F**
- 一张**Materials Studio 风格**的功函数图（使用分数坐标并可做周期延展显示）

### 功能特点
- **与 Materials Studio 作图习惯一致**
  - 强制符号约定：`V_plot(z) = - V_grd(z)`
  - 将真空平台在图中平移到 **0 eV**
- **鲁棒的真空平台自动识别**
  - 全局平台检测 + 兜底滑动窗口（困难体系不会直接报错崩溃）
- **支持超大 .grd 文件**
  - 采用流式读取与计算，降低内存压力
- **MS 风格横坐标**
  - 使用分数坐标 `z/c`
  - 支持周期延展（例如 0.1–1.1 会显示 1.0–1.1 的曲线数据）

### 安装
```bash
pip install -r requirements.txt
```

### 快速开始

**示例 1：标准运行（默认 MS 风格 0.1–1.1）**
```bash
python dmol3_wf_stream_style_final.py --grd NCS.grd --outmol NCS.outmol --prefix NCS
```

**示例 2：调整横坐标范围**
```bash
python dmol3_wf_stream_style_final.py --grd NCS.grd --outmol NCS.outmol --prefix NCS --xlim -0.1 1.2
```

**示例 3：COF 等“平台较难识别”的体系**
尝试更强的平滑与更宽松的平台判据：
```bash
python dmol3_wf_stream_style_final.py --grd COF-WF_potential.grd --outmol COF-WF.outmol --prefix COF --smooth-w 41 --slope-percentile 80
```

### 输出文件说明
以 `--prefix NCS` 为例，会得到：

- `NCS_Vz.csv`  
  列：`z_A, fractional_z, V_plot_eV(=-Vgrd), V_shift_eV(vac=0)`
- `NCS_Vz_raw.png`  
  符号翻转后的原始平面平均 `V(z)`（尚未平移真空到 0 eV）
- `NCS_WF_style.png`  
  MS 风格功函数图（真空线在 0 eV、费米能级线与 Φ 箭头）

### 重要说明（假设与约定）
- `*.grd` 必须是 **DMol3 的 electrostatic potential（静电势）** 网格文件。
- 脚本强制使用 `V_plot = -V_grd` 以匹配 Materials Studio 的符号约定。
- 真空能级 `V_vac` 来自平面平均 `V(z)` 的真空平台区间平均值。
- 功函数计算公式：
  \[
  \Phi = V_{vac} - E_F
  \]
  其中 `E_F` 从 outmol 解析得到（单位 eV）。

### 常用参数
- `--xlim XMIN XMAX`：分数坐标 x 轴范围（默认 `0.10 1.10`）
  - 若 `XMAX > 1`，脚本会自动做周期延展，显示 `1.0–1.1`（与 MS 一致）
- `--smooth-w N`：平滑窗口（默认 `17`）
- `--slope-percentile P`：平坦判据分位数（默认 `30`）
- `--prefer-frac F0 F1`：偏好的真空平台范围（默认 `0.25 0.95`）
- `--order xyz|xzy|...`：如果曲线形态异常，尝试更换网格顺序
- `--chunk-mb N`：超大网格可降低（如 `8`）以减少内存占用

更多参数请查看脚本头部 docstring。

### 许可证
MIT License，见 `LICENSE`。


---


A lightweight Python script to compute work function from **DMol3**:
- Electrostatic potential grid: `*.grd`
- Fermi energy: `*.outmol`

It outputs:
- Planar-averaged potential `V(z)`
- Automatically detected vacuum plateau `V_vac`
- Work function: **Φ = V_vac − E_F**
- A **Materials Studio-like** work-function plot (fractional coordinate with periodic extension)

### Features
- **Matches Materials Studio plot convention**
  - Enforces sign convention: `V_plot(z) = - V_grd(z)`
  - Shifts vacuum plateau to **0 eV** in the plot
- **Robust vacuum detection**
  - Global plateau detection + fallback sliding-window (no hard crash on difficult cases)
- **Huge .grd supported**
  - Streaming parser for large text grids
- **MS-like x-axis**
  - Fractional coordinate `z/c`
  - Optional periodic extension (e.g., 0.1–1.1 shows data in 1.0–1.1)

### Installation
```bash
pip install -r requirements.txt
```

### Quick Start

**Example 1: Standard run (MS-like 0.1–1.1)**
```bash
python dmol3_wf_stream_style_final.py --grd NCS.grd --outmol NCS.outmol --prefix NCS
```

**Example 2: Adjust x-axis range**
```bash
python dmol3_wf_stream_style_final.py --grd NCS.grd --outmol NCS.outmol --prefix NCS --xlim -0.1 1.2
```

**Example 3: COF-like cases (plateau harder to detect)**
Try stronger smoothing + looser flatness threshold:
```bash
python dmol3_wf_stream_style_final.py --grd COF-WF_potential.grd --outmol COF-WF.outmol --prefix COF --smooth-w 41 --slope-percentile 80
```

### Output Files
For `--prefix NCS`, you will get:

- `NCS_Vz.csv`  
  Columns: `z_A, fractional_z, V_plot_eV(=-Vgrd), V_shift_eV(vac=0)`
- `NCS_Vz_raw.png`  
  Raw planar-averaged `V(z)` after sign flip (before vacuum shift)
- `NCS_WF_style.png`  
  MS-like work function plot (vacuum level at 0 eV, Fermi level, Φ arrow)

### Important Notes (Assumptions)
- The `*.grd` must be **DMol3 electrostatic potential** grid.
- The script **forces** `V_plot = -V_grd` to match Materials Studio convention.
- Vacuum level `V_vac` is obtained from the **planar-averaged** potential `V(z)` vacuum plateau.
- Work function is computed as:
  \[
  \Phi = V_{vac} - E_F
  \]
  where `E_F` is parsed from `outmol` (eV).

### Parameters (Common)
- `--xlim XMIN XMAX` : fractional-coordinate x-axis limits (default `0.10 1.10`)
  - if `XMAX > 1`, the curve is periodically extended to show `1.0–1.1` like MS
- `--smooth-w N` : smoothing window (default `17`)
- `--slope-percentile P` : flatness threshold percentile (default `30`)
- `--prefer-frac F0 F1` : preferred vacuum region (default `0.25 0.95`)
- `--order xyz|xzy|...` : try different grid orders if curve shape looks wrong
- `--chunk-mb N` : lower it (e.g., `8`) for huge grids or low-memory machines

See the script docstring for the full argument list.

### License
MIT License. See `LICENSE`.

---
