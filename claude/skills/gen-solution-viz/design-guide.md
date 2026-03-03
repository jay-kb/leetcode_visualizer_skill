# 可视化 HTML 设计规范

本规范定义了 LeetCode 题解交互式可视化页面的设计标准。

## 1. 布局规范

### 整体结构
```
+----------------------------------------------------------+
|                    顶部：题目名称和编号和算法类型             |
+----------------------------------------------------------+
|  +---------------------------+  +---------------------+   |
|  |    代码显示窗口            |  |    参数输入区域     |  |
|  |    (左侧固定)             |  +---------------------+    |
|  |                           |  |    功能按钮区域     |  |
|  +---------------------------+  +---------------------+    |
|                                  |    动画/可视化区域   |    |
|                                  |    (右侧固定)       |    |
+----------------------------------------------------------+
|                    底部：键盘快捷键支持                      |
+----------------------------------------------------------+
```

### 关键约束
- **页面高度固定**：使用 `vh` 单位，整体占满视口高度，不允许页面滚动
- **左右分栏**：左侧代码区 40%，右侧可视化区 60%
- **右侧固定**：动画/可视化区域保持完全固定，不滚动
- **参数输入区**：位于右侧可视化区最上方，支持用户自定义参数

## 2. CSS 变量定义

```css
:root {
    /* 背景色 */
    --bg-primary: #0f0f23;
    --bg-secondary: #1a1a2e;
    --bg-tertiary: #16213e;

    /* 主色调 */
    --primary: #6366f1;
    --primary-light: #818cf8;
    --primary-dark: #4f46e5;

    /* 状态色 */
    --success: #10b981;
    --warning: #f59e0b;
    --error: #ef4444;

    /* 文字色 */
    --text-primary: #e4e4e7;
    --text-secondary: #a1a1aa;
    --text-muted: #71717a;

    /* 其他 */
    --border: #27272a;
    --code-bg: #1e1e2e;
}
```

## 3. 组件样式

### 按钮
```css
.btn {
    padding: 8px 16px;
    border: none;
    border-radius: 8px;
    font-size: 0.875rem;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s ease;
}

.btn-primary {
    background: var(--primary);
    color: white;
}

.btn-secondary {
    background: var(--bg-secondary);
    color: var(--text-primary);
    border: 1px solid var(--border);
}
```

### 数组节点（动态适配）
```css
.bars-container {
    display: flex;
    align-items: flex-end;
    justify-content: center;
    gap: 2px;
    flex: 1;
    min-height: 0;
    padding: 8px;
}

.bar-wrapper {
    display: flex;
    flex-direction: column;
    align-items: center;
    flex: 1;
    min-width: 0;  /* 关键：防止flex子项溢出 */
}

.bar {
    width: 100%;  /* 占满wrapper宽度 */
    max-width: 60px;
    min-width: 8px;  /* 最小宽度，防止太窄 */
    background: linear-gradient(180deg, var(--primary) 0%, var(--primary-dark) 100%);
    border-radius: 4px 4px 0 0;
    transition: all 0.3s ease;
}
```

### 动态适配规则（重要）
- **柱子宽度**：根据数组长度动态计算，元素多时自动变窄，元素少时自动变宽
- **最小宽度限制**：`min-width: 8px` 防止元素过小无法显示
- **最大宽度限制**：`max-width: 60px` 防止元素过大
- **使用 flex: 1** 让所有柱子均匀分配空间
- **容器溢出防护**：`overflow: hidden` + `min-width: 0`

### 卡片
```css
.card {
    background: var(--bg-tertiary);
    border-radius: 12px;
    padding: 16px;
    border: 1px solid var(--border);
}
```

## 4. 代码高亮

### HTML 结构
```html
<pre><code>
<span class="code-line" data-line="1">第一行代码</span>
<span class="code-line" data-line="2">第二行代码</span>
</code></pre>
```

### 高亮样式
```css
.code-line {
    display: block;
    padding: 2px 8px;
    border-radius: 4px;
    border-left: 3px solid transparent;
    transition: all 0.2s ease;
}

.code-line.highlight {
    background: rgba(99, 102, 241, 0.25);
    border-left-color: var(--primary);
}
```

## 5. 动画过渡

```css
/* 默认过渡 */
* {
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

/* 脉冲效果 */
@keyframes pulse {
    0%, 100% { box-shadow: 0 0 0 0 rgba(99, 102, 241, 0.4); }
    50% { box-shadow: 0 0 0 8px rgba(99, 102, 241, 0); }
}

.element.current {
    animation: pulse 1.5s infinite;
}
```

## 6. 响应式断点

```css
/* 平板 */
@media (max-width: 1024px) {
    .main-content {
        flex-direction: column;
    }
    .code-panel, .right-panel {
        width: 100%;
    }
}

/* 手机 */
@media (max-width: 768px) {
    .header h1 {
        font-size: 1.25rem;
    }
    .bar {
        width: 24px;
    }
}
```

## 7. 关键实现要求

### 固定高度布局（重要）
```css
/* 禁止页面整体滚动 */
html, body {
    height: 100%;
    overflow: hidden;
}

/* 主内容区占满剩余高度 */
.main-content {
    height: calc(100vh - 60px - 48px); /* 减去头部和底部高度 */
    overflow: hidden;
}

/* 左侧代码面板 - 占满高度 */
.code-panel {
    width: 45%;
    height: 100%;
    display: flex;
    flex-direction: column;
}

/* 左侧代码块可滚动【重要】- 独立滚动窗口 */
.code-block {
    flex: 1;
    overflow: auto;  /* 独立滚动条，可手动拖动 */
    min-height: 0;
    /* 自定义滚动条样式 */
    scrollbar-width: thin;
    scrollbar-color: var(--primary-dark) var(--bg-secondary);
}

.code-block::-webkit-scrollbar {
    width: 8px;
}

.code-block::-webkit-scrollbar-track {
    background: var(--bg-secondary);
}

.code-block::-webkit-scrollbar-thumb {
    background: var(--primary-dark);
    border-radius: 4px;
}

/* 右侧面板 - 禁止滚动，动画固定 */
.right-panel {
    width: 55%;
    overflow: hidden;  /* 禁止滚动 */
}
```

### 代码行高亮滚动到中央
```javascript
const codeBlock = document.querySelector('.code-block');
const containerHeight = codeBlock.clientHeight;
const lineTop = activeLine.offsetTop;
const lineHeight = activeLine.offsetHeight;
const scrollTop = lineTop - containerHeight / 2 + lineHeight / 2;

codeBlock.scrollTo({
    top: Math.max(0, scrollTop),
    behavior: 'smooth'
});
```

## 8. 变量状态面板

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
    color: #818cf8;
    font-weight: 600;
}

.variable .var-value {
    color: #e4e4e7;
}

/* 变量类型区分 */
.variable.pointer .var-name { color: #f59e0b; }   /* 指针变量 */
.variable.bool .var-name { color: #10b981; }      /* 布尔变量 */
.variable.map .var-name { color: #fb923c; }       /* 对象变量 */
```

## 9. 参数输入区域

### HTML 结构
```html
<div class="params-panel">
    <h3>参数输入</h3>
    <div class="params-form">
        <div class="param-item">
            <label>height</label>
            <input type="text" id="param-height" value="[0,1,0,2,1,0,1,3,2,1,2,1]">
        </div>
        <button class="btn btn-primary" onclick="runWithParams()">运行</button>
    </div>
</div>
```

### CSS 样式
```css
.params-panel {
    background: var(--bg-tertiary);
    border-radius: 8px;
    padding: 12px;
    border: 1px solid var(--border);
    flex-shrink: 0;
}

.params-form {
    display: flex;
    align-items: center;
    gap: 12px;
    flex-wrap: wrap;
}

.param-item {
    display: flex;
    align-items: center;
    gap: 8px;
}

.param-item label {
    font-size: 0.75rem;
    color: var(--text-secondary);
    font-weight: 500;
}

.param-item input {
    padding: 6px 10px;
    border-radius: 4px;
    background: var(--bg-secondary);
    color: var(--text-primary);
    border: 1px solid var(--border);
    font-family: 'Fira Code', monospace;
    font-size: 0.8rem;
    width: 200px;
}

.param-item input:focus {
    outline: none;
    border-color: var(--primary);
}
```

## 10. 参数解析与执行

### JavaScript 实现
```javascript
// 默认参数
const defaultParams = {
    height: [0,1,0,2,1,0,1,3,2,1,2,1]
};

// 解析用户输入
function parseParams(input) {
    try {
        // 支持数组格式: [1,2,3] 或 "1,2,3"
        const trimmed = input.trim();
        if (trimmed.startsWith('[')) {
            return JSON.parse(trimmed);
        }
        return trimmed.split(',').map(s => parseFloat(s.trim()));
    } catch (e) {
        return null;
    }
}

// 使用参数生成步骤
function generateStepsWithParams(params) {
    const steps = [];
    const height = params.height;

    // 根据参数生成步骤
    // ...

    return steps;
}

// 运行并重置
function runWithParams() {
    const input = document.getElementById('param-height').value;
    const parsed = parseParams(input);

    if (!parsed) {
        alert('参数格式错误，请输入有效的数组格式');
        return;
    }

    // 更新参数并重新生成步骤
    currentParams.height = parsed;
    steps = generateStepsWithParams(currentParams);

    // 重置到第一步
    currentStep = 0;
    updateUI();
}
```

### 参数类型支持
- **数组**: `[1,2,3,4,5]` 或 `1,2,3,4,5`
- **字符串**: `"hello world"` 或 `hello world`
- **数字**: `123` 或 `12.5`
- **二维数组**: `[[1,2],[3,4]]`

## 11. 验收标准

1. ✅ 页面占满视口高度，无垂直滚动条
2. ✅ 左侧代码区可滚动，右侧可视化区固定
3. ✅ 代码高亮行自动滚动到可视范围中央
4. ✅ 变量状态实时更新
5. ✅ 所有过渡动画平滑 (0.3s)
6. ✅ 深色主题一致性
7. ✅ 键盘快捷键正常工作
8. ✅ 参数输入区域显示在可视化区最上方
9. ✅ 用户可自定义参数并实时更新动画
10. ✅ 参数解析支持多种格式（数组、字符串、数字）
