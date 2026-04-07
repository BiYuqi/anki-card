---
name: anki-card
description: 将英语文本文件转换为可导入 Anki 的 CSV 卡片文件。当用户运行 /anki-card 命令时触发（格式：/anki-card /path/to/file.txt）。根据用户词汇量等级智能筛选生词，生成带音标和中文释义的卡片，输出为当前目录下的 CSV 文件。
---

# Anki CSV 生成器

用户通过 `/anki-card /path/to/file.txt` 调用此 skill。

## 第一步：询问词汇量等级

读取文件之前，先向用户展示选项：

```
请选择你的英语水平：
1. 初中水平（约 1500 词）
2. 高中水平（约 3000 词）
3. 四级（CET-4，约 4000 词）
4. 六级（CET-6，约 6000 词）
5. 考研 / 专业（约 8000 词）
6. 雅思（IELTS，约 8500+ 词）

请输入数字（1-6）：
```

等用户输入数字后，再继续处理。

## 第二步：读取并处理文本

读取用户指定的文件，然后：

- 提取完整句子和核心短语
- 去重（相同句子只保留一次）
- 清理多余内容：编号、引用符号（"" ''）、多余空格、段落标记、HTML 标签（保留 `<br>`）

**优先选取：**
- 短句、高频表达、演讲核心句
- 结构清晰、可记忆的句子

**跳过：**
- 过长或结构复杂的学术句
- 无意义重复句

## 第三步：为每句话生成卡片

### Front（正面）

使用原英文句子或短语，但有一项必须处理：

**分号替换**：如果原句中含有分号（`;`），将其替换为逗号（`,`）或破折号（`—`），选择语义更自然的那个。原因：CSV 以分号为字段分隔符，Front 中出现分号会导致导入格式错误。

禁止：编号、中文、括号说明、任何注释。

### Back（背面）

根据用户等级，判断句中每个词是否超出其词汇范围：

**有超出等级的生词时：**

```
词组或单词 /IPA音标/ 中文释义<br>词组或单词 /IPA音标/ 中文释义<br>整句自然中文翻译
```

**所有词对该等级都太简单时：**

```
整句自然中文翻译
```

直接输出翻译，不附加词汇注解。

### 选词原则

**跳过功能词**（无论等级）：
a, the, to, of, and, is, are, was, were, in, on, for, with, that, this, it, he, she, they, we, you, I, be, not, at, by, from

**优先拆解**（按词汇难度筛选）：
- 动词（如 amplify, distinguish, persevere）
- 名词（如 innovation, resilience, paradigm）
- 形容词（如 relentless, profound, concise）
- 副词（如 remarkably, consistently）
- 高频短语（如 follow your heart, think differently）

**只列出超出用户等级的词**，已掌握的词不重复注解。

**跨卡片词汇去重（重要）**：

在处理整批句子时，维护一个「已注解词汇表」（按原形记录）。规则如下：

- 某个词**第一次**出现 → 正常注解（原形 /IPA/ 释义）
- 同一个词（原形相同）在后续卡片中**再次出现** → 不再注解，仅保留在中文翻译中体现

词形判断基于原形：`cornered` / `corners` / `cornering` 都算同一个词 `corner`，第一次注解后后续全部跳过。

示例（假设 `puma` 已在第 1 张卡片注解过）：

- 第 1 张：`puma /ˈpjuːmə/ 美洲狮`（首次出现，正常注解）
- 第 3 张：Back 里不再出现 `puma` 的注解行，翻译中直接用"美洲狮"即可

### 词形规则

词汇注解**统一使用原形（词典形式）**，不管单词在句中以何种变形出现：

- 动词：一律用原形（不用过去式、过去分词、进行时）
- 形容词：一律用原级（不用比较级、最高级）
- 名词：一律用单数（不用复数）

示例：
- 句中 `cornered` → 注解写 `corner /ˈkɔːrnər/ 逼入绝境`
- 句中 `disturbing` → 注解写 `disturb /dɪˈstɜːrb/ 使…不安`
- 句中 `accumulated` → 注解写 `accumulate /əˈkjuːmjəleɪt/ 积累`
- 句中 `extraordinarily` → 副词本身就是原形，直接用

原因：学习者背的是单词根形，变形规则另外掌握；原形也与词典保持一致。

### 音标规则

必须使用标准 IPA 国际音标，格式如 `/steɪ/`、`/ɪnˈtelɪdʒəns/`。

禁止使用拼音或非标准注音。

### 中文翻译规则

最后一行是整句的**自然中文翻译**：
- 符合中文表达习惯
- 语义完整
- 不逐词硬翻

示例：
- Think differently → 以不同方式思考（不是"思考 不同地"）
- Stay hungry, stay foolish → 保持饥渴，保持愚蠢（保留原文节奏）

## 第四步：输出 CSV 文件

### 文件命名

格式：`{源文件名}_{时间戳}.csv`

- 源文件名：去掉扩展名的原始文件名
- 时间戳：当前时间精确到秒（YYYYMMDDHHMMSS）

例如：
- 输入 `apa.txt` → 输出 `apa_20260407143052.csv`
- 输入 `jobs_speech.txt` → 输出 `jobs_speech_20260407143052.csv`

### 保存位置

保存到运行命令时所在目录下的 `output/` 子目录：

```
{当前目录}/output/{源文件名}_{时间戳}.csv
```

如果 `output/` 目录不存在，自动创建。

### 文件格式

- 分隔符：`;`
- 编码：UTF-8
- 无表头行
- 每行一张卡片

### 示例输出

```
Computers are tools for the mind.;tool /tuːl/ 工具<br>mind /maɪnd/ 思维<br>计算机是思维的工具。
They amplify human intelligence.;amplify /ˈæmplɪfaɪ/ 放大<br>intelligence /ɪnˈtelɪdʒəns/ 智力<br>它们放大人类智力。
This is a book.;这是一本书。
A computer is like a bicycle for the mind.;bicycle /ˈbaɪsɪkəl/ 自行车<br>mind /maɪnd/ 思维<br>计算机就像思维的自行车。
```

## 完成后告知用户

生成完毕后，简短告知：

```
已生成 X 张卡片，保存至：/当前目录/20260407143052.csv

Anki 导入设置：分隔符选 ; ，字段顺序：正面 / 背面
```
