# 题库 JSON

这个目录是**纯数据**，不依赖任何服务端代码，可以单独作为一个 git 仓库 / 静态资源目录托管。

> 已发布：**https://github.com/happysoysaucemin/quiz-dx-bank**（文件在仓库根目录）
> - jsDelivr：`https://cdn.jsdelivr.net/gh/happysoysaucemin/quiz-dx-bank@main/`
> - raw：`https://raw.githubusercontent.com/happysoysaucemin/quiz-dx-bank/main/`
> - 用法：把这些地址（含结尾 `/`）填到 `web/js/config.js` 的 `bankBase`。

```
bank/
├─ index.json    # 科目索引、题量、生成时间
├─ 13006.json    # 软件工程（实践）
├─ 13007.json    # 编译原理
├─ 13008.json    # 软件质量保证与测试
└─ 13014.json    # 高级语言程序设计（实践）
```

## index.json

```json
{
  "version": 1,
  "generatedAt": "2026-11-20T12:00:00+00:00",
  "source": "四科模拟试卷_纠错解析与挖空版",
  "total": 296,
  "subjects": [
    {
      "code": "13006",
      "name": "软件工程（实践）",
      "short": "软工",
      "file": "13006.json",
      "counts": { "fill": 16, "single": 30, "judge": 18 },
      "total": 64
    }
  ]
}
```

## 科目文件

```json
{
  "code": "13007",
  "name": "编译原理",
  "short": "编译",
  "file": "13007.json",
  "counts": { "fill": 27, "single": 26, "judge": 24 },
  "total": 77,
  "questions": [ … ]
}
```

每道题：

```jsonc
// 单选题
{
  "id": "13007-single-1",
  "no": 1,                         // 原卷题号
  "order": 28,                     // 在科目内的全局顺序
  "type": "single",
  "stem": "（ ）不是 NFA 的成分。",
  "options": [
    { "key": "A", "text": "有穷字母表" },
    { "key": "B", "text": "初始状态集合" },
    { "key": "C", "text": "终止状态集合" },
    { "key": "D", "text": "有限状态集合" }
  ],
  "answer": "B",
  "answerText": "B",
  "explain": "NFA 的五元组为 …",
  "note": "答案错误｜原答案标 D …"   // 可选：勘误说明
}
```

```jsonc
// 判断题
{ "type": "judge", "stem": "…", "answer": false, "answerText": "错", "explain": "…" }

// 填空题
{
  "type": "fill",
  "stem": "编译程序一般由词法分析器、______、______ …",
  "blanks": 4,
  "answer": "语法分析器、语义分析器、中间代码生成（代码优化器）、目标代码生成器",
  "accept": [                        // 每个空的可接受答案（用于自动判分）
    ["语法分析器"],
    ["语义分析器"],
    ["中间代码生成（代码优化器）", "代码优化器", "中间代码生成"],
    ["目标代码生成器"]
  ],
  "explain": "…"
}
```

> `accept` 由 `tools/build_bank.py` 从答案文本自动生成：按「、」「；」拆分，括号内同义答案、括号前的主体都算对。

## 托管方式

题库是静态文件，任何静态托管都可以：

### 1. 跟前端放在一起

把 `bank/` 放在 `index.html` 同级目录，`config.js` 里 `bankBase` 留空：

```
site/
├─ index.html
├─ js/…
└─ bank/
   ├─ index.json
   └─ 13006.json …
```

### 2. 单独一个仓库 / GitHub Pages

已发布的题库仓库（文件直接放在仓库根目录）：

```js
bankBase: 'https://cdn.jsdelivr.net/gh/happysoysaucemin/quiz-dx-bank@main/',
```

自己重新发布时，把 `bank/` 里的文件推到新仓库根目录即可；如果题库放在大仓库的子目录里（如 `bank/`），地址记得带上子目录：

```bash
cd 刷题器
git subtree push --prefix bank origin gh-pages   # 或单独建仓库
```

```js
bankBase: 'https://<user>.github.io/<repo>/bank/',   // 子目录时带 /bank/
```

### 3. jsDelivr（带 CDN 缓存）

```js
// 题库在仓库根目录
bankBase: 'https://cdn.jsdelivr.net/gh/<user>/<repo>@main/',
// 题库在仓库的 bank/ 子目录
bankBase: 'https://cdn.jsdelivr.net/gh/<user>/<repo>@main/bank/',
```

> 首次请求会回源，之后由 CDN 缓存；`@main` 可以换成 tag / commit 固定版本。

> `bankBase` 必须以 `/` 结尾，并且目录里要有 `index.json`。
> 前端会依次尝试 `./bank/`、`../bank/`、`/bank/` 三个同一来源的路径，最后才用配置里的地址。

## 更新题库

```bash
python tools/build_bank.py   # 重新生成
node tools/check.mjs         # 校验
```

`index.json` 的 `generatedAt` 会更新；前端不做持久化缓存，刷新即用新版。
