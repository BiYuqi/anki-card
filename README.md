# anki-card

一个 Claude Code Skill，将英语文本文件自动转换为可导入 Anki 的 CSV 卡片。

根据你的词汇量等级（四级 / 六级 / 考研 / 雅思）智能筛选生词，为每个句子生成带 IPA 音标和中文释义的学习卡片。

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
Innovation distinguishes between a leader and a follower.;innovation /ˌɪnəˈveɪʃn/ 创新<br>distinguishes /dɪˈstɪŋɡwɪʃɪz/ 区分<br>创新区分了领导者与追随者。
We're here to put a dent in the universe.;dent /dent/ 凹痕；冲击<br>我们来到这里，是为了在宇宙中留下印记。
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

或手动创建：
```bash
mkdir -p ~/.claude/skills/anki-card
cp SKILL.md ~/.claude/skills/anki-card/
```

---

## 使用方法

### 基本用法

```
/anki-card /path/to/article.txt
```

### 流程

1. 运行命令后，Claude 会提示选择词汇量等级：

```
请选择你的英语水平：
1. 四级以下
2. 四级（CET-4，约 4000 词）
3. 六级（CET-6，约 6000 词）
4. 考研 / 专业（约 8000 词）
5. 雅思（IELTS，约 8500+ 词）
```

2. 输入数字（1-5）后，自动生成 CSV 文件

3. 文件保存在**你运行命令时所在的目录**，文件名为时间戳（如 `20260407143052.csv`）

### 示例

```bash
# 在桌面运行
cd ~/Desktop
/anki-card ~/Downloads/speech.txt
# → 生成 ~/Desktop/20260407143052.csv
```

---

## CSV 格式说明

| 字段 | 内容 |
|------|------|
| 分隔符 | `;`（分号）|
| 编码 | UTF-8 |
| Front | 英文原句 |
| Back | 词汇注解（音标 + 中文）+ 整句翻译 |

Back 结构：
```
单词 /IPA音标/ 中文<br>单词 /IPA音标/ 中文<br>整句自然中文翻译
```

若该句所有词对你的等级而言都是已知词，Back 只输出中文翻译：
```
英文句子;整句中文翻译
```

---

## 导入 Anki

1. 打开 Anki → 文件 → 导入
2. 选择生成的 `.csv` 文件
3. 设置：
   - **字段分隔符**：`;`（分号）
   - **字段映射**：字段 1 → 正面，字段 2 → 背面
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
| 四级以下 | ~2000 词 | 初学者，高中英语水平 |
| 四级（CET-4）| ~4000 词 | 大学英语四级 |
| 六级（CET-6）| ~6000 词 | 大学英语六级 |
| 考研 / 专业 | ~8000 词 | 研究生入学考试水平 |
| 雅思（IELTS）| ~8500+ 词 | 雅思 / 学术英语水平 |

等级越低，被标注的词越多（对你来说生词更多）。

---

## License

MIT
