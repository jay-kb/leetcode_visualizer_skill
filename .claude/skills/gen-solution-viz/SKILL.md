# LeetCode Solution Visualizer

基于设计规范生成 LeetCode 题解的交互式可视化 HTML。生成的 HTML 采用深色主题，与主站浅色液态玻璃风格形成对比。

## 触发条件

当用户请求以下内容时使用此 skill：
- "生成题解可视化"
- "创建一个可视化页面"
- "帮我写一个算法演示"
- /gen-viz
- 需要为一个算法/题目创建交互式 HTML 演示

## 执行流程

### Step 1: 确认需求
如果用户没有提供足够信息，请询问：
- 题目名称和编号（如 "1. 两数之和"）
- 算法类型（数组、哈希表、链表、树、排序等）
- 难度（简单/中等/困难）
- 使用的编程语言
- 是否已有代码需要可视化

### Step 2: 读取设计规范
必须读取 `gen-solution-viz/design-guide.md` 获取设计规范，包括：
- 深色主题配色方案
- CSS 组件样式（按钮、数组节点、卡片等）
- 动画过渡效果
- 布局规范
- 响应式断点
- **固定高度布局要求（重要：页面必须占满视口，不允许整体滚动）**

### Step 3: 格式化代码（重要！）

**【关键】在生成 HTML 之前，必须对代码进行格式化处理！**

从 leetcode.cn 获取的原始代码通常是未格式化的文本，缺少正确的缩进和空格。必须先进行格式化。

#### 3.1 检测代码是否需要格式化

检查原始代码是否存在以下问题：
- 缺少正确的缩进（没有 4 空格缩进）
- 运算符前后缺少空格（如 `for(int i=0;` 而不是 `for (int i = 0;`）
- 代码行连在一起
- 大括号风格不正确

#### 3.2 格式化代码

使用 AI 能力对代码进行格式化，要求：

- **保持代码逻辑完全不变**
- **使用 4 空格缩进**
- **运算符前后添加空格**（如 `i=0` → `i = 0`，`a+b` → `a + b`）
- **方法括号前不空格，括号后有空格**（如 `method(` 而非 `method (`）
- **if/for/while 等语句括号内有空格**（如 `if(` → `if (`）
- **大括号风格**：左大括号在行尾，右大括号单独一行
- **逗号后必须有空格**
- **添加适当的空行增加可读性**
- **保留原有注释**

#### 3.3 格式化 prompt 示例

请使用以下 prompt 让 AI 格式化代码：

```
请将以下 Java 代码按照 Google Java Style 规范格式化，保持代码逻辑不变，只调整格式：

[在此粘贴原始代码]

要求：
1. 使用 4 空格缩进
2. 运算符前后加空格 (a+b → a + b)
3. 方法调用括号前不加空格 (method( → method( )
4. if/for/while 等语句括号内有空格 (if( → if ( )
5. 大括号风格：左大括号在行尾，右大括号单独一行
6. 逗号后加空格
7. 适当添加空行增加可读性
8. 保留原有注释
```

#### 3.4 格式化后的代码用于生成 HTML

格式化后的代码将用于：
1. 插入到 HTML 的 `<pre><code>` 标签中显示在左侧代码窗口
2. 生成步骤对象时使用（用于代码高亮行对应）

### Step 4: 生成 HTML
生成的 HTML 必须遵循以下布局结构：

**布局约束（重要）：**
- **顶部**：题目名称,编号和算法类型（如 "1. 两数之和 数组"）**居中显示** （必须有）
- **左侧**：代码显示窗口（固定宽度约 45%）（必须有）
  - **【重要】必须是独立可滚动窗口**：代码区域有独立的滚动条，可以手动拖动查看
  - **必须有代码高亮行移动（步骤指示器）**，随着步骤变化，当前执行的代码行要实时高亮显示
  - **高亮行自动滚动到中央**：随着步骤变化，当前执行的代码行**必须要实时高亮显示、并移动到窗口的中央**
- **右侧**：分四部分 （必须有）
  - **最上部：参数输入区域**（重要！支持用户自定义参数）
  - 第二部分：功能控制按钮
  - **第三部分：动画/可视化区域（核心，必须占满右侧剩余空间，使用 flex: 1）**
  - **第四部分：键盘快捷键提示（必须在动画区域下方）**
- **【重要】右侧面板必须占满页面**：可视化区域使用 `flex: 1` 和 `min-height: 0` 确保占满所有剩余空间

```
+----------------------------------------------------------+
|                    顶部：题目名称和编号和算法类型             |
+----------------------------------------------------------+
|  +---------------------------+  +---------------------+   |
|  |    代码显示窗口            |  |    参数输入区域     |  |
|  |    (可滚动窗口)           |  +---------------------+   |
|  |    [独立滚动条]           |  |    功能按钮区域     |   |
|  +---------------------------+  +---------------------+   |
|                                  |    动画/可视化区域   |    |
|                                  |    (占满剩余空间)   |    |
|                                  +---------------------+    |
|                                  | 快捷键 (动画下方)   |    |
+----------------------------------------------------------+
```

**样式要求：**
- 使用设计规范中的 CSS 变量定义颜色
- 深色背景渐变 (#0f0f23 → #1a1a2e)
- 与主站一致的 accent color (#6366f1)
- VS Code Dark+ 风格的代码高亮
- 圆角 8px-12px（比主站稍小）
- 使用 Flexbox 实现左右分栏布局

**功能要求：**
- **参数输入区域（重要！）**：位于右侧可视化区最上方
  - 支持用户自定义算法参数
  - 支持多种格式：数组 `[1,2,3]`、字符串 `"hello"`、数字 `123`
  - 输入后点击"运行"按钮应用新参数
  - 参数变化后动画自动适配
- 开始/暂停按钮
- 上一步/下一步按钮
- 重置按钮
- 自动播放按钮（带速度控制更好）
- **步骤指示器：左侧代码窗口必须有代码行高亮跟随（重要！）**
- 代码展示区（在左侧面板，**必须支持步骤变化时高亮行实时移动**）
- 结果展示区（在右侧动画区域下方）
- 键盘快捷键支持（在底部）

**代码高亮实现要点（重要）：**
- 代码使用 `<pre><code>` 展示，每行用 `<span class="code-line" data-line="行号">` 包裹
- CSS 中定义 `.code-line.highlight` 样式：背景色 rgba(99, 102, 241, 0.25)，左边框 3px solid var(--primary)
- **关键：每个代码行都必须有对应的步骤，不能跳过任何代码行！**
  - 例如代码有 19 行，就要生成至少 19 个步骤
  - 每个步骤的 `codeLine` 必须按顺序对应每一行代码
  - 循环体内的代码行在每次循环迭代时都要有对应步骤
  - **方法调用时也要生成对应的步骤**：如 `getMax(height)` 调用，需要为被调用方法的每一行生成步骤
- JavaScript 中每个步骤对象必须包含 `codeLine` 属性，指明当前高亮行号
- `updateUI()` 函数中根据当前步骤的 `codeLine` 更新高亮样式

**【重要】方法调用时步骤指示器必须跳转：**
- 当代码执行到方法调用时（如 `int max = getMax(height);`），步骤指示器**必须**跳转到被调用方法的代码行
- 执行流程：
  1. 先跳转到被调用方法的起始行（如行20）
  2. 逐行执行被调用方法的每一行代码
  3. 方法执行完毕后，再回到主方法的调用处继续执行
- **关键**：每个被调用方法的每一行代码都必须有对应的步骤

**【重要】左侧代码区域是可滑动窗口【必须实现】，右侧动画保持不动：**

**核心需求【重要】：**
- 左侧代码区域是一个**可以滑动的窗口**，内部代码可以上下滚动【必须实现】
- 步骤指示器移动时，**仅左侧代码块内部滚动**，让高亮行始终显示在可视范围内
- **整体画面不动**，右侧动画/可视化区域完全保持固定

**实现要求：**
- **禁止**：使用 `window.scrollTo()` 或 `document.body.scrollTo()` 滚动整个页面
- **禁止**：右侧容器（`.right-panel`、`.visualization`）添加任何滚动或位置变动

**CSS 要求（关键）：**
```css
/* 中间主内容区 - 占满剩余高度 */
.main-content {
    flex: 1;
    display: flex;
    overflow: hidden;  /* 防止整体滚动 */
}

/* 左侧代码面板 - 固定宽度，占满高度 */
.code-panel {
    width: 45%;
    height: 100%;      /* 占满父容器高度 */
    overflow: hidden;  /* 面板本身不滚动 */
    display: flex;
    flex-direction: column;
}

/* 左侧代码块 - 可滑动窗口（关键！） */
.code-block {
    flex: 1;
    min-height: 0;     /* 允许 flex 子项收缩 */
    overflow: auto;    /* 只有代码块内部可以滚动 */
}

/* 右侧面板 - 完全固定，占满高度 */
.right-panel {
    width: 55%;
    height: 100%;
    display: flex;
    flex-direction: column;
    overflow: hidden;  /* 禁止滚动，保持固定 */
}

/* 可视化区域 - 占满剩余空间（关键！） */
.visualization {
    flex: 1;           /* 关键：占满右侧剩余空间 */
    min-height: 0;     /* 允许 flex 子项收缩 */
    overflow: hidden;  /* 禁止滚动，动画区域保持不动 */
    display: flex;
    flex-direction: column;
}

/* 树/数组容器 - 占满可视化区域 */
.tree-container {
    flex: 1;
    min-height: 0;
    overflow: hidden;
}
```

**JavaScript 实现（关键）：**
```javascript
// 只滚动左侧代码块内部，让高亮行显示在可视范围内
const codeBlock = document.querySelector('.code-block');
const containerHeight = codeBlock.clientHeight;  // 容器高度
const lineTop = activeLine.offsetTop;           // 高亮行顶部位置
const lineHeight = activeLine.offsetHeight;     // 高亮行高度

// 计算滚动位置，使高亮行在容器中央可见
const scrollTop = lineTop - containerHeight / 2 + lineHeight / 2;
codeBlock.scrollTo({
    top: Math.max(0, scrollTop),  // 确保不滚动到负数
    behavior: 'smooth'
});
```

**效果示意：**
```
+--------------------------------------------------+
|  顶部：题目名称（固定不动）                         |
+--------------------------------------------------+
|  +----------------+  +-------------------------+  |
|  | 左侧代码窗口   |  | 右侧动画区域             |  |
|  | (可滑动窗口)  |  | (固定不动)              |  |
|  |                |  |                         |  |
|  | [高亮行自动   |  | 变量实时更新             |  |
|  |  滚动到可见   |  | 数据可视化              |  |
|  |  代码移动到屏幕中央]|  |                    |  |
|  +----------------+  +-------------------------+  |
+--------------------------------------------------+
|  底部：快捷键提示（固定不动）                       |
+--------------------------------------------------+
```

**右侧动画/可视化区域设计（重要）：**
右侧可视化区域必须包含以下标准化布局：

```
+--------------------------------------------------+
|  变量状态面板 (Variables Panel)                   |
|  +------------+  +------------+  +-----------+  |
|  | 变量名:值  |  | 变量名:值  |  | 变量名:值 |  |
|  +------------+  +------------+  +-----------+  |
+--------------------------------------------------+
|  数据结构可视化区 (Data Visualization)            |
|  +----------------------------------------+    |
|  |    数组/链表/树等图形化展示              |    |
|  +----------------------------------------+    |
+--------------------------------------------------+
|  步骤说明 (Step Description)                    |
+--------------------------------------------------+
```

**变量状态面板设计要求（重要）：**
- **必须**：变量面板在每个步骤变化时**实时更新**
- **变量名词高亮显示**：变量名使用醒目的颜色（如 primary-light），值使用白色
- **变量类型区分颜色**：
  - 指针/索引变量（left, right, i, j 等）：使用 warning 颜色（橙色）
  - 数值变量（sum, ans, max 等）：使用 primary 颜色（紫色）
  - 布尔变量（isStart, found 等）：使用 success 颜色（绿色）
  - 复杂对象（map, array 等）：使用 string 颜色（橙色）
- **变量值实时变更**：变量值必须跟随步骤指示器逐行移动实时更新
- **动态显示/隐藏**：只有当前步骤使用到的变量才显示，未使用的变量不显示或显示为默认值
- CSS 样式示例：
```css
.variable {
    display: inline-flex;
    align-items: center;
    gap: 4px;
    padding: 4px 10px;
    background: rgba(99, 102, 241, 0.15);
    border: 1px solid rgba(99, 102, 241, 0.3);
    border-radius: 4px;
}
.variable .var-name {
    color: #818cf8;  /* 高亮变量名 */
    font-weight: 600;
}
.variable .var-value {
    color: #e4e4e7;  /* 值用白色 */
}
.variable.pointer .var-name { color: #f59e0b; }   /* 指针变量名用橙色 */
.variable.bool .var-name { color: #10b981; }      /* 布尔变量名用绿色 */
```
- 示例：
  ```
  ┌──────────┐ ┌──────────┐ ┌──────────┐
  │left: 3   │ │right: 5  │ │ans: 7    │
  │(pointer) │ │(pointer) │ │(number)  │
  └──────────┘ └──────────┘ └──────────┘
  ```

**JavaScript 步骤对象设计：**
```javascript
{
    codeLine: 3,           // 当前高亮代码行
    variables: {           // 变量状态（重要！每个步骤必须有）
        left: 3,           // 指针变量
        right: 5,
        ans: 7,
        map: {a: 1, b: 2},
        isStart: true      // 布尔变量
    },
    dataState: [...],       // 数据结构状态
    desc: "步骤描述"       // 步骤说明
}
```

**updateUI() 中必须包含（关键实现）：**
```javascript
function updateUI() {
    if (currentStep >= steps.length) return;
    const step = steps[currentStep];

    // 1. 更新代码高亮行（左侧代码窗口）
    const codeLines = document.querySelectorAll('.code-line');
    codeLines.forEach(line => line.classList.remove('highlight'));
    const activeLine = document.querySelector(`.code-line[data-line="${step.codeLine}"]`);
    if (activeLine) {
        activeLine.classList.add('highlight');
        // 只滚动左侧代码容器，右侧保持不动
        document.querySelector('.code-block').scrollTo({
            top: activeLine.offsetTop - 100,
            behavior: 'smooth'
        });
    }

    // 2. 【重要】更新变量状态面板 - 实时更新变量值
    if (step.variables) {
        const vars = step.variables;
        const varsPanel = document.getElementById('variablesPanel');
        // 根据变量类型生成不同的样式类
        let html = '';
        for (const [key, value] of Object.entries(vars)) {
            let typeClass = '';
            if (['left', 'right', 'i', 'j', 'idx', 'index'].includes(key)) {
                typeClass = 'pointer';
            } else if (typeof value === 'boolean') {
                typeClass = 'bool';
            } else if (typeof value === 'object') {
                typeClass = 'map';
            }
            html += `<span class="variable ${typeClass}">
                <span class="var-name">${key}</span>=<span class="var-value">${JSON.stringify(value)}</span>
            </span>`;
        }
        varsPanel.innerHTML = html;
    }

    // 3. 更新数据结构可视化
    updateDataVisualization(step.dataState);

    // 4. 更新步骤说明
    document.getElementById('stepInfo').textContent = step.desc;

    // 5. 更新进度
    document.getElementById('progress').textContent = `步骤 ${currentStep + 1} / ${steps.length}`;
}
```

**关键要点总结：**
1. ✅ **变量实时更新**：`updateUI()` 中必须根据当前步骤的 `step.variables` 实时更新变量面板
2. ✅ **变量名高亮**：变量名使用醒目的颜色（如 `#818cf8`）
3. ✅ **伴随步骤移动**：代码高亮行变化时，`step.variables` 必须同步更新对应变量的值
4. ✅ **只滚动左侧**：使用 `document.querySelector('.code-block').scrollTo()` 而非整个页面

**动画要求：**
- 元素高亮/找到结果时使用 CSS transition
- 平滑的过渡效果 (0.3s, cubic-bezier)
- 当前处理元素的脉冲效果

### 动态适配设计（重要）

生成的 HTML 必须支持根据用户输入的参数动态适配可视化元素，确保不与其他元素产生遮挡。

#### 数组动态适配
```javascript
// 根据数组长度动态计算柱子宽度
const maxBarWidth = 60;
const minBarWidth = 8;
const containerWidth = container.clientWidth;
const padding = 16;
const availableWidth = containerWidth - padding * 2;
const barWidth = Math.max(minBarWidth, Math.min(maxBarWidth, availableWidth / arr.length));
const gap = Math.max(1, Math.min(4, 8 / arr.length));
```

#### 二叉树动态适配（关键实现）
```javascript
// 计算树的节点位置（动态适配）
function calculatePositions(node, positions = [], level = 0, parentX = 0) {
    if (!node) return;

    const treeDepth = getDepth(node);
    const treeWidth = Math.pow(2, treeDepth - 1);

    // 动态计算节点间距，根据树的深度和宽度
    const baseSpacingX = Math.max(30, 300 / treeWidth);  // 根据树的宽度动态调整
    const baseSpacingY = Math.max(50, 300 / treeDepth);  // 根据树的深度动态调整

    // ... 位置计算逻辑
}

// 渲染时动态缩放
function renderTree(positions, visitedNodes, currentNode, resultNode) {
    const container = document.getElementById('treeContainer');
    const width = container.clientWidth;
    const height = container.clientHeight;

    // 计算x和y的范围
    let minX = Infinity, maxX = -Infinity, maxY = -Infinity;
    positions.forEach(p => {
        minX = Math.min(minX, p.x);
        maxX = Math.max(maxX, p.x);
        maxY = Math.max(maxY, p.y);
    });

    const rangeX = maxX - minX || 1;
    const rangeY = maxY || 1;

    // 动态计算缩放，确保完全适应容器
    const padding = 40;
    const scaleX = Math.min(1, (width - padding * 2) / rangeX);
    const scaleY = Math.min(1, (height - padding * 2) / rangeY);
    const scale = Math.min(scaleX, scaleY, 1.2);

    // 节点半径根据缩放动态调整
    const nodeRadius = Math.max(12, Math.min(20, 18 * scale));

    // 居中偏移
    const offsetX = (width - rangeX * scale) / 2 - minX * scale;
    const offsetY = (height - rangeY * scale) / 2;

    // 渲染节点和边，使用动态计算的参数
}
```

#### 动态适配核心要点
1. **容器尺寸检测**：每次渲染前获取容器的实际 `clientWidth` 和 `clientHeight`
2. **元素范围计算**：遍历所有元素，计算 x/y 的 min/max 值
3. **缩放比例计算**：`scale = Math.min((width - padding) / range, 1)`
4. **居中偏移**：根据计算出的范围和缩放，计算居中偏移量
5. **元素尺寸调整**：节点半径、柱子宽度等根据 scale 动态调整
6. **窗口 resize 监听**：监听 `window.resize` 事件，容器大小时重新渲染

#### 验证方法
每次生成 HTML 后，必须测试：
1. 输入最小参数（如 `[1]` 或 `[1]`）- 检查是否正常显示
2. 输入大参数（如 20+ 元素的数组或深度 5+ 的树）- 检查是否有遮挡
3. 调整浏览器窗口大小 - 检查是否自适应

### Step 5: 验证生成的 HTML（重要！）

**每次生成 HTML 后，必须执行以下验证步骤，确保代码行对齐正确：**

#### 5.1 验证代码行数匹配

检查格式化后的代码行数与 steps 数组中的 codeLine 范围是否匹配：

```javascript
// 在生成 steps 后添加验证逻辑
const codeLineCount = codeLines.length;  // 格式化后的代码总行数
const maxCodeLine = Math.max(...steps.map(s => s.codeLine));  // steps 中最大的 codeLine
const minCodeLine = Math.min(...steps.map(s => s.codeLine));  // steps 中最小的 codeLine

if (maxCodeLine > codeLineCount) {
    throw new Error(`代码行对齐错误: step 中最大的 codeLine (${maxCodeLine}) 超过了代码总行数 (${codeLineCount})`);
}
if (minCodeLine < 1) {
    throw new Error(`代码行对齐错误: step 中最小的 codeLine (${minCodeLine}) 小于 1`);
}
```

#### 5.2 验证每个代码行都有对应步骤

确保代码的每一行都有对应的步骤（尤其是循环体内的代码）：

```javascript
// 检查关键代码行是否都有对应步骤
const requiredLines = [];
for (let i = 1; i <= codeLineCount; i++) {
    const codeLineText = codeLines[i - 1].trim();
    // 跳过空行
    if (!codeLineText) continue;
    // 跳过纯注释行
    if (codeLineText.startsWith('//') || codeLineText.startsWith('/*')) continue;

    const hasStep = steps.some(s => s.codeLine === i);
    if (!hasStep) {
        console.warn(`警告: 代码行 ${i} ("${codeLineText.substring(0, 30)}...") 没有对应的步骤`);
    }
}
```

#### 5.3 验证步骤的 codeLine 连续性

确保步骤的 codeLine 按执行顺序合理递增：

```javascript
// 验证步骤顺序
let lastCodeLine = 0;
for (const step of steps) {
    // codeLine 可以不变、递增或跳跃（循环），但不能倒退
    if (step.codeLine < lastCodeLine && step.codeLine !== lastCodeLine) {
        console.warn(`警告: codeLine 从 ${lastCodeLine} 倒退到 ${step.codeLine}，请检查步骤顺序`);
    }
    lastCodeLine = step.codeLine;
}
```

#### 5.4 生成验证报告

在 HTML 文件中添加注释，说明验证结果：

```html
<!--
代码行对齐验证报告:
- 代码总行数: X
- 步骤总数: Y
- codeLine 范围: 1 - Z
- 验证状态: 通过/失败
-->
```

#### 5.5 自动修复常见问题

如果发现以下常见问题，自动进行修复：

1. **codeLine 超出范围**: 将超出的 codeLine 调整到合理范围
2. **缺少空行的步骤**: 为空行添加占位步骤
3. **循环体步骤不完整**: 为每次循环迭代生成完整步骤

#### 5.6 JavaScript 语法错误检查（重要！）

**【必须检查】生成 HTML 后，必须确保 JavaScript 语法正确：**

1. **动态键名语法错误（常见！）**：
  - 错误写法：`{ 'dp[' + j + ']': maxVal }`
  - 正确写法：
    ```javascript
    // 方案1：先定义变量
    var key = 'dp[' + j + ']';
    var variables = {};
    variables[key] = maxVal;
    steps.push({ variables: variables, ... });

    // 方案2：使用计算属性
    steps.push({
        variables: {
            [ 'dp[' + j + ']' ]: maxVal
        },
        ...
    });
    ```

2. **在生成完 steps 后，运行以下检查**：
   ```javascript
   // 验证 steps 数组语法正确
   try {
       JSON.stringify(steps);
       console.log('Steps 语法验证通过');
   } catch (e) {
       console.error('Steps 语法错误:', e.message);
   }
   ```

3. **使用浏览器开发者工具控制台检查**：打开生成的 HTML，按 F12 查看控制台是否有语法错误

### Step 6: 保存文件
- 路径：`/Users/qinsx/project/AI/claude-skill/html/<title-slug>-<current-timestamp>.html`
- 确保目录存在（首次使用需创建 html 文件夹）

## 示例 Prompt

用户可能说：
- "帮我生成两数之和的可视化"
- "创建一个快速排序的交互式演示"
- "我要一个二叉树遍历的可视化页面"

## 输出格式

### 验证清单（每次生成后必须检查）

完成 HTML 生成后，必须向用户报告以下验证结果：

```
✅ 代码行对齐验证:
   - 原始代码行数: X
   - 格式化后代码行数: Y
   - 步骤总数: Z
   - codeLine 范围: 1 - N
   - 验证状态: [通过/失败]

✅ 布局验证:
   - 顶部标题: [有/无]
   - 左侧代码窗口: [有/无] (可滚动)
   - 右侧参数输入: [有/无]
   - 右侧可视化区域: [有/无]
   - 快捷键提示: [有/无]

✅ 功能验证:
   - 步骤指示器: [有/无]
   - 变量实时更新: [有/无]
   - 高亮行自动滚动: [有/无]
```

### 告知用户

完成后告知用户：
1. 生成的文件路径
2. 验证报告
3. 如何在页面中预览
