# FBG 透射光谱谷值定位与拟合分析

使用 Python 分析真实 FBG 透射光谱，比较最低采样点、高斯拟合中心与局部二次拟合顶点，并检查结果对拟合范围的敏感性。这是一个可重复运行的光谱数据分析学习项目。

![高斯拟合与残差](results/gaussian_fit_residuals.png)

## 已完成的内容

- 读取并检查真实光谱，绘制完整与局部波段。
- 用最低点法定位谷底采样点。
- 在 dBm 数据上进行高斯谷经验拟合，并检查残差。
- 比较三组高斯拟合范围下的中心变化。
- 用谷底附近的二次多项式估计局部最低位置，检查不同邻域点数的影响。
- 汇总方法结果，导出图片与 CSV。

## 数据来源

原始数据由 Will Smith、Peter Mosley 提供，来自巴斯大学公开数据仓库（2026）：

[Dataset for “Narrowband heralded single photons via Bragg grating inscription in germanium-doped photonic crystal fiber”](https://researchdata.bath.ac.uk/1662/)

- DOI：[10.15125/BATH-01662](https://doi.org/10.15125/BATH-01662)
- 数据许可：[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
- 使用文件：`SourceGratingTrans.csv`。本项目读取完整数据后，在内存中截取局部波段并生成拟合、图片与统计表，不修改原始 CSV。
- `Wavelength` 为波长（nm），`dBm` 为对数光功率。共 1201 点，波长范围 1552.84–1558.84 nm，采样间隔 0.005 nm（5 pm）。

原始数据归原作者，本项目增加了分析代码与说明。数据许可不自动代表本项目代码的许可。

## 运行方法

1. 下载项目，将笔记本、数据和 requirements.txt 保留在同一目录。
2. 在项目目录安装依赖：

```bash
python -m pip install -r requirements.txt
```

3. 用支持 Jupyter Notebook 的编辑器打开 `fbg-spectrum-dip-fitting.ipynb`，选择已安装依赖的 Python 内核，并将工作目录设为项目目录。四个依赖库为 NumPy、Pandas、Matplotlib 和 SciPy；Notebook 编辑器及内核支持需由使用者的环境提供。
4. 重启内核，从上到下运行全部单元。最后一个单元将更新 `results/` 中的结果。

## 分析思路

### 最低点与高斯拟合

在 1555.20–1555.90 nm 波段内，使用 `np.argmin()` 找最低点，并使用 `curve_fit()` 拟合：

```python
def gaussian_dip(wavelength, b, A, c, w):
    return b - A * np.exp(-((wavelength - c) / w)**2)
```

参数依次为背景 b（dBm）、谷深 A（dB）、中心 c（nm）、宽度参数 w（nm）。w 不是半高全宽。该式是对 dBm 曲线的经验描述，并非完整 FBG 物理模型。

残差 = 测量值 − 拟合值，单位为 dB。连续的残差结构表明模型未充分描述数据。

### 高斯拟合区间敏感性

保持模型与初始猜测一致，分别拟合三个区间：

| 区间（nm） | 点数 | 拟合中心（nm） | 相对原高斯中心的变化（pm） |
|---|---:|---:|---:|
| 1555.20–1555.90 | 141 | 1555.575272 | 0.000 |
| 1555.30–1555.80 | 101 | 1555.574025 | −1.247 |
| 1555.40–1555.75 | 71 | 1555.572735 | −2.538 |

三个中心的变化范围约 2.538 pm。两侧边界同时改变，因此本实验不能单独归因于左侧或右侧边界。

![不同高斯拟合区间](results/gaussian_range_sensitivity.png)

### 局部二次拟合

以最低采样点为参考，令 u = 波长 − 最低点波长，拟合 P(u) = a·u² + b·u + d。通过顶点 u = −b/(2a) 得到局部位置估计。检查开口向上且顶点在窗口内；它们是基本检查，不保证准确。

![局部二次拟合](results/quadratic_local_fit.png)

## 结果汇总

下表差值全部以最低采样点波长为参考，正值表示向长波方向移动。

| 方法 | 位置估计（nm） | 相对最低采样点的变化（pm） |
|---|---:|---:|
| 最低点法 | 1555.560000 | 0.000 |
| 高斯拟合（原区间） | 1555.575272 | 15.272 |
| 二次拟合（左右各 3 点，共 7 点） | 1555.565765 | 5.765 |
| 二次拟合（左右各 5 点，共 11 点） | 1555.567875 | 7.875 |
| 二次拟合（左右各 8 点，共 17 点） | 1555.568413 | 8.413 |

显示的小数位仅用于记录计算结果，不表示实际测量精度。局部二次拟合在这三个窗口下的顶点变化范围约 2.648 pm。

## 结论与局限

- 实际光谱谷不对称，右侧存在肩部。对称高斯模型无法充分描述谷底与肩部，残差存在系统性结构。
- 拟合范围和局部邻域点数会影响位置估计。
- 最低点、高斯模型中心、局部抛物线顶点的含义不同，不应直接视为同一物理量的等价测量。
- 没有精确中心真值，因此上述差值是方法或设置之间的差异，不是定位误差或测量不确定度，不能据此排列准确度。
- 本项目仅分析一条光谱中的一个谷，尚未验证其他传感器和实验条件。局部切片假设两侧数据充足；处理其他数据前需要检查边界、异常值及退化拟合。
- 后续可用真值已知的模拟数据验证算法，或增加多条真实光谱；本版本不声称完整复现原论文。

## 项目文件

```text
fbg-spectrum-dip-fitting/
├── fbg-spectrum-dip-fitting.ipynb
├── SourceGratingTrans.csv
├── README.md
├── requirements.txt
├── .gitignore
└── results/
    ├── gaussian_fit_residuals.png
    ├── gaussian_range_sensitivity.png
    ├── quadratic_local_fit.png
    ├── method_comparison.csv
    ├── gaussian_range_sensitivity.csv
    └── quadratic_window_sensitivity.csv
```

`.venv/`、`.idea/` 和缓存目录属于本地环境，不需要上传。
