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

### Step 3: 生成 HTML
生成的 HTML 必须遵循以下布局结构：

**布局约束（重要）：**
- **顶部**：题目名称,编号和算法类型（如 "1. 两数之和 数组"）**居中显示** （必须有）
- **左侧**：代码显示窗口（固定宽度约 45%）（必须有）
  - **【重要】必须是独立可滚动窗口**：代码区域有独立的滚动条，可以手动拖动查看
  - **必须有代码高亮行移动（步骤指示器）**，随着步骤变化，当前执行的代码行要实时高亮显示
  - **高亮行自动滚动到中央**：随着步骤变化，当前执行的代码行**必须要实时高亮显示、并移动到窗口的中央**
- **右侧**：分三部分 （必须有）
  - **最上部：参数输入区域**（重要！支持用户自定义参数）
  - 中部：功能控制按钮
  - 下部：动画/可视化区域（完全固定，不滚动）
- **底部**：键盘快捷键提示（必须有）

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
|                                  |    (固定不动)       |    |
+----------------------------------------------------------+
|                    底部：键盘快捷键支持                      |
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

/* 右侧面板 - 完全固定 */
.right-panel {
    width: 55%;
    overflow: hidden;  /* 禁止滚动，保持固定 */
}

.visualization {
    overflow: hidden;  /* 禁止滚动，动画区域保持不动 */
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

### Step 4: 保存文件
- 路径：`/Users/qinsx/project/AI/leetcode_visualizer/nginx/html/solutions/<title-slug>-<current-timestamp>.html`
- 确保目录存在

## 示例 Prompt

用户可能说：
- "帮我生成两数之和的可视化"
- "创建一个快速排序的交互式演示"
- "我要一个二叉树遍历的可视化页面"

## 输出格式

完成后告知用户：
1. 生成的文件路径
2. 如何在页面中预览
