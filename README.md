# 中国考古遗址动物骨骼测量数据自动化提取

从204篇中文动物考古学PDF文献中批量提取骨骼测量数据，基于大语言模型辅助的正则表达式自动化流程。

## 背景

骨骼测量数据是动物考古学研究的基础。von den Driesch (1976) 提出的标准测量体系（GL、Bp、Bd、SD等）是国际通用的测量语言。然而，这些数据散落在数百篇中文文献中，格式千差万别——有的用规范表格，有的嵌在正文里，有的是扫描件数字都复制不出来，没有人愿意花几个月去逐篇整理。

我们用自动化手段，在不到一周的时间内完成了这项工作。

## 方法

```
PDF文献 → PyMuPDF提取文本 → 文本清洗 → 正则匹配 → 种属推断 → 标准化 → 去重 → 输出
```

大语言模型（Claude）辅助编写和调试提取脚本，数据提取由确定性的正则表达式完成，每条数据都有据可查。

提取脚本核心逻辑：
- 全角字符→半角（"１８２．２１" → "182.21"）
- PUA私用区字符替换为空格（防止数字合并）
- 单位统一（"毫米" → "mm"）
- 按段落搜索关键词推断种属（家养动物优先匹配）

## 数据概况

| 指标 | 数值 |
|------|------|
| 处理文献 | 204篇 |
| 成功提取 | **92篇**（45%） |
| 总记录 | **3,217条**（去重后） |
| 覆盖遗址 | 92个 |
| 种属 | 30种 |
| 测量变量 | 392种（von den Driesch标准） |
| 时间跨度 | 旧石器时代 → 明代 |
| 地理范围 | 15个省/自治区 |

### 种属分布

| 种属 | 记录数 | 占比 |
|------|--------|------|
| pig（家猪） | 1,809 | 56.2% |
| cattle（黄牛） | 413 | 12.8% |
| sheep（羊） | 260 | 8.1% |
| sambar_deer（水鹿） | 186 | 5.8% |
| chicken（家鸡） | 119 | 3.7% |
| water_buffalo（水牛） | 114 | 3.5% |
| dog（家犬） | 112 | 3.5% |
| horse（马） | 99 | 3.1% |
| 其他22种 | 96 | 3.0% |

### 地理分布（主要省份）

| 省份 | 记录数 | 主要站点 |
|------|--------|---------|
| 内蒙古 | 810 | 哈民忙哈(549), 西岔(247) |
| 河南 | 665 | 大司空(317), 煤山(73) |
| 云南 | 576 | 河泊所(414), 武定新村(145) |
| 陕西 | 211 | 木柱柱梁(32), 大古界(31) |
| 吉林 | 205 | 汪清(118), 后套木嘎(68) |
| 四川 | 184 | 商业街(181) |
| 山东 | 162 | 焦家(39), 小北山(36) |

### 提取效果好的文献特征

1. **文本型PDF**（非扫描图像），文字可直接复制
2. **使用VDD标准代码**（如"最大长GL""近端最大宽Bp"）
3. **行内描述式写法**（如"标本最大长为38.52mm"）

### 提取失败的主要原因

112篇文献返回0条记录：
- **扫描版PDF，数值丢失**（~50%）：数字被替换为控制字符
- **同位素/综述文献，本无测量数据**（~25%）
- **数据以散点图/箱线图呈现**（~10%）
- **仅汇总统计**（~10%）

## 目录结构

```
├── CLAUDE.md                          # 开发者配置
├── README.md                          # 本文件
├── scripts/                           # 提取脚本
│   ├── step23_batch02_extraction.py   # Batch02: 20篇
│   ├── step24_batch03_extraction.py   # Batch03: 20篇
│   ├── step25_batch04_extraction.py   # Batch04: 10篇
│   ├── step26_recover_dasikong.py     # 大司空遗址专项恢复
│   ├── step27_batch05_extraction.py   # Batch05: 20篇
│   ├── step28_batch06_extraction.py   # Batch06: 20篇
│   ├── step29_batch07_extraction.py   # Batch07: 18篇
│   ├── step30_batch08_extraction.py   # Batch08: 15篇
│   ├── step31_batch09_extraction.py   # Batch09: 18篇
│   └── step32_batch_final.py          # 最终批处理: 99篇
├── data/                              # 提取数据
│   ├── bone_measurement_merged.csv    # 合并数据（CSV，1.7MB）
│   ├── bone_measurement_merged.xlsx   # 合并数据（Excel，632KB）
│   ├── batches/                       # 各批次原始数据
│   └── by_site/                       # 按遗址分文件（108个）
└── docs/                              # 文档
    ├── bone_measurement_final_report.md     # 技术报告
    └── 公众号_骨骼测量数据提取.md           # 科普文章
```

## 数据字段说明

| 字段 | 说明 | 示例 |
|------|------|------|
| batch_id | 批次编号 | BATCH05 |
| source_pdf_file | 来源PDF文件名 | 安阳大司空遗址2016年出土动物遗存研究_王红英.pdf |
| site_name | 遗址名称 | 大司空遗址 |
| province | 省份 | 河南省 |
| city_or_county | 市/县 | 安阳市 |
| chronological_period_original | 原始时代标注 | 殷墟时期（一至四期） |
| date_start | 起始时代 | 殷墟一期 |
| taxon_original | 原始种属名 | 家猪 |
| taxon_standardized | 标准化种属名 | pig |
| element_original | 原始骨骼部位 | 下颌骨 |
| element_standardized | 标准化骨骼部位 | mandible |
| measurement_variable_original | 原始测量变量 | 最大长 |
| measurement_variable_standardized | VDD标准代码 | GL |
| measurement_value_numeric | 数值(mm) | 149.99 |
| measurement_value_raw | 原始数值 | 149.99mm |
| value_min | 最小值 | 120.5 |
| value_max | 最大值 | 165.3 |
| value_mean | 平均值 | 142.8 |
| n_if_summary | 标本数(汇总统计) | 5 |
| confidence | 置信度 | high/medium/low |
| evidence_quote | 原文引用 | 最大长(GL): 149.99mm |
| extraction_method | 提取方式 | text_table/text_body |

## 使用方法

### 直接使用已提取数据

```python
import pandas as pd

df = pd.read_csv('data/bone_measurement_merged.csv')

# 查询家猪下颌骨GL值
pig_mandible = df[
    (df['taxon_standardized'] == 'pig') &
    (df['measurement_variable_standardized'] == 'GL')
]
```

### 运行提取脚本（需自行准备PDF文本）

```bash
# 安装依赖
pip install pandas openpyxl

# 以Batch05为例
python scripts/step27_batch05_extraction.py
```

脚本会自动：
1. 读取 `extracted_md/` 目录下的Markdown文件（需自行从PDF提取）
2. 清洗全角字符和OCR噪声
3. 用正则表达式提取测量数据
4. 输出CSV/Excel到 `outputs/bone_measurements/`

### 环境要求

```
Python 3.9+
PyMuPDF (fitz)（仅PDF提取时需要）
pandas
openpyxl
```

## 数据质量说明

| 置信度 | 占比 | 含义 |
|--------|------|------|
| high | 74.8% | 文本型PDF，VDD标准代码，数据可靠 |
| medium | 24.0% | 行内提取，种属推断可能存在偏差 |
| low | 1.3% | OCR干扰较大，需人工复核 |

已知问题：
- 2条异常值（>500mm）来自OCR错误（青邱堌堆遗址）
- 10条种属标记为undetermined
- 部分文献的种属推断可能不准确

## 数据发布建议

如果您的研究包含骨骼测量数据，以下做法能极大提高数据的可复用性：

1. **发表时附CSV/Excel数据表**——对作者举手之劳，对整个领域意义重大
2. **使用文本型PDF**——排版软件直接导出，保留文字可复制性
3. **统一VDD代码**——采用von den Driesch标准变量名
4. **避免全角数字**——使用半角数字和半角句点

## 引用

如使用本数据，请引用：
- 技术报告: `docs/bone_measurement_final_report.md`
- 原始文献: 见各条目的 `source_pdf_file` 字段

## 许可

数据仅用于学术研究目的。
