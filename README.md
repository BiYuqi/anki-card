# anki-card

一个 Claude Code Skill，将英语文本文件自动转换为可导入 Anki 的 CSV 卡片。

根据你的词汇量等级智能筛选生词，为每个句子生成带 IPA 音标和中文释义的学习卡片。

---

## 效果预览

输入文章：
```
Innovation distinguishes between a leader and a follower.
We're here to put a dent in the universe.
Think differently.
```

生成 CSV（六级水平）：
```
Innovation distinguishes between a leader and a follower.;innovation /ˌɪnəˈveɪʃn/ 创新<br>distinguish /dɪˈstɪŋɡwɪʃ/ 区分<br>创新区分了领导者与追随者。
We're here to put a dent in the universe.;dent /dent/ 凹痕，冲击<br>我们来到这里，是为了在宇宙中留下印记。
Think differently.;以不同的方式思考。
```

---

## 安装

### 前提条件

- [Claude Code](https://claude.ai/code) CLI 已安装
- Claude Code 版本支持 Skills（`~/.claude/skills/` 目录）

### 安装步骤

```bash
# 1. 克隆或下载此仓库
git clone https://github.com/your-username/anki-card-skill.git

# 2. 复制到 Claude 全局 skills 目录
cp -r anki-card-skill ~/.claude/skills/anki-card

# 3. 验证安装（重启 Claude Code 后，输入 / 应能看到 anki-card）
```

---

## 使用方法

### 基本用法

```
/anki-card /path/to/article.txt
```

### 流程

1. 运行命令后，Claude 会弹出词汇量选择面板（4 个常用等级）：

   - 初中水平（约 1500 词）
   - 高中水平（约 3000 词）
   - 四级 CET-4（约 4000 词）
   - 六级 CET-6（约 6000 词）

   **需要考研 / 雅思 / 自定义词汇量？** 选「其他」手动输入：
   - `考研` 或 `研究生` → 按 8000 词处理
   - `雅思` 或 `IELTS` → 按 8500 词处理
   - 纯数字如 `2000`、`5000` → 按该词汇量处理

2. 选择后自动处理文件，生成 CSV

3. 文件保存在**你运行命令时所在目录的 `output/` 子目录**，文件名格式为 `{原文件名}_{时间戳}.csv`

### 示例

```bash
cd ~/project/english
/anki-card speech.txt
# → 生成 ~/project/english/output/speech_20260407171921.csv
```

---

## CSV 格式说明

| 字段 | 内容 |
|------|------|
| 分隔符 | `;`（分号）|
| 编码 | UTF-8 |
| Front | 英文原句（原句中的分号会被替换为逗号或破折号） |
| Back | 词汇注解（音标 + 中文）+ 整句翻译 |

Back 结构：
```
单词 /IPA音标/ 中文<br>单词 /IPA音标/ 中文<br>整句自然中文翻译
```

若该句所有词对你的等级而言都是已知词，Back 只输出中文翻译：
```
英文句子;整句中文翻译
```

**跨卡片词汇去重**：同一个词（按原形）只在第一次出现时注解，后续卡片不重复标注。

---

## 导入 Anki

1. 打开 Anki → 文件 → 导入
2. 选择生成的 `.csv` 文件
3. 设置：
   - **字段分隔符**：`;`（分号）
   - **字段映射**：字段 1 → 正面，字段 2 → 背面
   - **允许 HTML**：开启（Back 中含 `<br>` 换行标签）
4. 点击导入

---

## 支持的输入格式

- 纯文本（`.txt`）
- 内容：英文文章、演讲稿、课文、对话等
- 自动清理：编号、引号、段落标记、多余空格

---

## 词汇量等级说明

| 等级 | 词汇量 | 适合人群 |
|------|--------|---------|
| 初中水平 | ~1500 词 | 初中英语水平 |
| 高中水平 | ~3000 词 | 高中英语水平 |
| 四级（CET-4）| ~4000 词 | 大学英语四级 |
| 六级（CET-6）| ~6000 词 | 大学英语六级 |
| 考研 / 专业 | ~8000 词 | 研究生入学考试水平 |
| 雅思（IELTS）| ~8500+ 词 | 雅思 / 学术英语水平 |
| 自定义数字 | 任意 | 直接输入词汇量数字，如 2000 |

等级越低，被标注的词越多（对你来说生词越多）。

---

## License

MIT
