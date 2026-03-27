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
|                                  +---------------------+    |
|                                  | 快捷键 (动画下方)   |    |
+----------------------------------------------------------+
```

### 关键约束
- **页面高度固定**：使用 `vh` 单位，整体占满视口高度，不允许页面滚动
- **左右分栏**：左侧代码区 45%，右侧可视化区 55%
- **右侧固定**：动画/可视化区域保持完全固定，不滚动
- **参数输入区**：位于右侧可视化区最上方，支持用户自定义参数
- **快捷键位置**：必须在动画/可视化区域的下方，页面最底部

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

## 12. 动态适配设计（重要）

### 12.1 核心原则
- 可视化元素必须根据容器大小动态调整
- 禁止出现元素遮挡问题
- 支持窗口 resize 时自动重新渲染

### 12.2 数组动态适配
```css
/* 动态柱子宽度 */
.bar {
    width: 100%;
    max-width: 60px;
    min-width: 8px;
    flex: 1;
}
```

### 12.3 二叉树动态适配（解决节点遮挡问题）

**【重要】必须按照以下步骤实现，否则会出现节点遮挡问题！**

```javascript
// ===== 正确的二叉树动态适配实现 =====

function renderTree(tree, currentNode, heights, imbalanced, visited) {
    const container = document.getElementById('treeContainer');
    container.innerHTML = '';
    if (!tree) { container.innerHTML = '<div>空树</div>'; return; }

    const width = container.clientWidth;
    const height = container.clientHeight;

    // 第一步：收集所有节点位置（使用相对坐标，包含父子关系）
    const rawPositions = [];
    const depth = getTreeDepth(tree);
    // 根据深度计算基础间距，深度越深间距越小
    const baseSpacingX = Math.max(60, 200 / Math.pow(1.5, depth - 1));
    calculateTreePositionsWithParent(tree, rawPositions, 0, 0, baseSpacingX, undefined, undefined);

    if (rawPositions.length === 0) return;

    // 第二步：计算位置范围
    let minX = Infinity, maxX = -Infinity, maxY = -Infinity;
    rawPositions.forEach(p => {
        minX = Math.min(minX, p.x);
        maxX = Math.max(maxX, p.x);
        maxY = Math.max(maxY, p.y);
    });

    const rangeX = maxX - minX || 1;
    const rangeY = maxY || 1;

    // 第三步：计算缩放比例，确保适应容器（解决遮挡问题的核心）
    const padding = 50; // 边距
    const availableWidth = width - padding * 2;
    const availableHeight = height - padding * 2;

    const scaleX = availableWidth / rangeX;
    const scaleY = availableHeight / rangeY;
    const scale = Math.min(scaleX, scaleY, 1.5); // 最大缩放1.5倍

    // 第四步：计算居中偏移
    const scaledWidth = rangeX * scale;
    const scaledHeight = rangeY * scale;
    const offsetX = (width - scaledWidth) / 2 - minX * scale;
    const offsetY = (height - scaledHeight) / 2;

    // 第五步：应用缩放和偏移，渲染边
    const positions = rawPositions.map(p => ({
        ...p,
        x: p.x * scale + offsetX,
        y: p.y * scale + offsetY,
        parentX: p.parentX !== undefined ? p.parentX * scale + offsetX : undefined,
        parentY: p.parentY !== undefined ? p.parentY * scale + offsetY : undefined
    }));

    // 渲染边
    positions.forEach(pos => {
        if (pos.parentX !== undefined) {
            const edge = document.createElement('div');
            edge.className = 'tree-edge';
            const length = Math.sqrt(Math.pow(pos.x - pos.parentX, 2) + Math.pow(pos.y - pos.parentY, 2));
            const angle = Math.atan2(pos.y - pos.parentY, pos.x - pos.parentX) * 180 / Math.PI;
            edge.style.width = length + 'px';
            edge.style.left = pos.parentX + 'px';
            edge.style.top = pos.parentY + 'px';
            edge.style.transform = `rotate(${angle}deg)`;
            container.appendChild(edge);
        }
    });

    // 第六步：动态计算节点大小（解决节点遮挡）
    const nodeRadius = Math.max(14, Math.min(22, 18 * scale));

    // 渲染节点
    positions.forEach(pos => {
        if (pos.val === null) return;
        const nodeEl = document.createElement('div');
        nodeEl.className = 'tree-node';
        // ... 设置节点样式
        nodeEl.style.width = (nodeRadius * 2) + 'px';
        nodeEl.style.height = (nodeRadius * 2) + 'px';
        nodeEl.style.fontSize = Math.max(0.7, 0.85 * scale) + 'rem';
        nodeEl.style.left = (pos.x - nodeRadius) + 'px';
        nodeEl.style.top = (pos.y - nodeRadius) + 'px';
        container.appendChild(nodeEl);
    });
}

// 计算节点位置（包含父子关系）
function calculateTreePositionsWithParent(node, positions, level, x, spacing, parentX, parentY) {
    if (!node) return;
    const y = level * 80;
    positions.push({ val: node.val, x: x, y: y, parentX: parentX, parentY: parentY });
    calculateTreePositionsWithParent(node.left, positions, level + 1, x - spacing, spacing * 0.6, x, y);
    calculateTreePositionsWithParent(node.right, positions, level + 1, x + spacing, spacing * 0.6, x, y);
}
```

**关键要点总结：**
1. **使用相对坐标**：先收集节点位置，再统一缩放
2. **动态计算缩放比例**：`scale = Math.min(availableWidth / rangeX, availableHeight / rangeY)`
3. **居中偏移**：确保树在容器中央显示
4. **动态节点大小**：根据缩放比例调整节点半径
5. **父子关系保存**：确保边的正确连接

### 12.4 窗口 resize 监听
```javascript
window.addEventListener('resize', () => {
    if (steps.length > 0 && currentStep < steps.length) {
        const step = steps[currentStep];
        renderTree(step.tree, step.currentNode, step.heights, step.imbalanced, step.visited);
    }
});
```

## 13. 链表可视化设计（重要！）

链表是 LeetCode 中常见的数据结构，本节定义链表可视化的标准实现。

### 13.1 链表节点结构

```html
<div class="node-wrapper">
    <div class="node">
        <span class="val">3</span>
        <span class="idx">idx:0</span>
    </div>
    <div class="arrow"></div>
</div>
```

### 13.2 CSS 样式

```css
/* 链表容器 */
.linked-list-container {
    flex: 1;
    display: flex;
    flex-direction: column;
    min-height: 0;
    overflow: hidden;
    background: var(--bg-secondary);
    border-radius: 8px;
    padding: 16px;
}

.linked-list {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-wrap: wrap;
    gap: 4px;
    flex: 1;
    min-height: 0;
    overflow: auto;
    padding: 10px;
}

/* 节点样式 */
.node-wrapper {
    display: flex;
    align-items: center;
    flex-shrink: 0;
}

.node {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 52px;
    height: 52px;
    background: linear-gradient(135deg, var(--bg-tertiary) 0%, var(--bg-secondary) 100%);
    border: 2px solid var(--border);
    border-radius: 10px;
    font-family: 'Fira Code', monospace;
    transition: all 0.3s ease;
    position: relative;
}

.node .val {
    font-size: 0.9rem;
    font-weight: 600;
}

.node .idx {
    font-size: 0.6rem;
    color: var(--text-muted);
    margin-top: 2px;
}

/* 指针标签 */
.pointer-label {
    position: absolute;
    top: -18px;
    font-size: 0.6rem;
    font-weight: bold;
    padding: 2px 6px;
    border-radius: 3px;
    white-space: nowrap;
}

.pointer-label.slow-label {
    background: rgba(245, 158, 11, 0.3);
    color: var(--warning);
}

.pointer-label.fast-label {
    background: rgba(239, 68, 68, 0.3);
    color: var(--error);
}

/* 箭头 */
.arrow {
    width: 20px;
    height: 2px;
    background: var(--text-muted);
    position: relative;
    flex-shrink: 0;
}

.arrow::after {
    content: '';
    position: absolute;
    right: -4px;
    top: -4px;
    border: 5px solid transparent;
    border-left-color: var(--text-muted);
}

/* 状态样式 */
.node.current {
    border-color: var(--primary);
    box-shadow: 0 0 0 4px rgba(99, 102, 241, 0.3);
    transform: scale(1.05);
}

.node.slow {
    border-color: var(--warning);
    box-shadow: 0 0 0 3px rgba(245, 158, 11, 0.3);
}

.node.fast {
    border-color: var(--error);
    box-shadow: 0 0 0 3px rgba(239, 68, 68, 0.3);
}

.node.meet {
    border-color: var(--error);
    background: linear-gradient(135deg, rgba(239, 68, 68, 0.2) 0%, var(--bg-secondary) 100%);
    animation: pulse-red 1s infinite;
}

.node.result {
    border-color: var(--warning);
    background: linear-gradient(135deg, rgba(245, 158, 11, 0.2) 0%, var(--bg-secondary) 100%);
    box-shadow: 0 0 0 4px rgba(245, 158, 11, 0.4);
    animation: pulse 1.5s infinite;
}

.node.visited {
    border-color: var(--success);
    background: linear-gradient(135deg, rgba(16, 185, 129, 0.15) 0%, var(--bg-secondary) 100%);
}

/* 环标记 */
.cycle-label {
    color: var(--error);
    font-size: 0.7rem;
    font-weight: bold;
    margin-left: 8px;
    flex-shrink: 0;
}
```

### 13.3 JavaScript 实现

```javascript
// 链表渲染函数
function renderLinkedList(state) {
    const container = document.getElementById('linkedList');
    container.innerHTML = '';

    if (!state || !state.nodes || state.nodes.length === 0) {
        container.innerHTML = '<div class="empty-hint">空链表</div>';
        return;
    }

    const { nodes, slow, fast, meet, result, finding, index1, index2 } = state;
    const cyclePos = state.cyclePos || -1; // 环入口位置

    for (let i = 0; i < nodes.length; i++) {
        const wrapper = document.createElement('div');
        wrapper.className = 'node-wrapper';

        const node = document.createElement('div');
        node.className = 'node';

        // 添加节点值和索引
        const valSpan = document.createElement('span');
        valSpan.className = 'val';
        valSpan.textContent = nodes[i];
        node.appendChild(valSpan);

        const idxSpan = document.createElement('span');
        idxSpan.className = 'idx';
        idxSpan.textContent = 'idx:' + i;
        node.appendChild(idxSpan);

        // 状态样式
        if (i === result) {
            node.classList.add('result');
            const label = document.createElement('div');
            label.className = 'pointer-label';
            label.style.top = '-20px';
            label.style.background = 'rgba(245, 158, 11, 0.5)';
            label.style.color = '#fff';
            label.textContent = '入口';
            node.appendChild(label);
        }

        if (meet !== null && i === meet && result === null) {
            node.classList.add('meet');
        }

        // slow 指针
        if (i === slow && slow >= 0) {
            node.classList.add('slow');
            const label = document.createElement('div');
            label.className = 'pointer-label slow-label';
            label.textContent = 'S';
            node.appendChild(label);
        }

        // fast 指针
        if (i === fast && fast >= 0) {
            node.classList.add('fast');
            const label = document.createElement('div');
            label.className = 'pointer-label fast-label';
            label.textContent = 'F';
            node.appendChild(label);
        }

        // 找环入口时的指针
        if (finding) {
            if (i === index1) {
                node.classList.add('current');
                const label = document.createElement('div');
                label.className = 'pointer-label';
                label.style.background = 'rgba(99, 102, 241, 0.5)';
                label.style.color = '#fff';
                label.textContent = 'i1';
                node.appendChild(label);
            }
            if (i === index2) {
                node.classList.add('visited');
                const label = document.createElement('div');
                label.className = 'pointer-label';
                label.style.background = 'rgba(16, 185, 129, 0.5)';
                label.style.color = '#fff';
                label.textContent = 'i2';
                node.appendChild(label);
            }
        }

        wrapper.appendChild(node);

        // 添加箭头（除了最后一个节点）
        if (i < nodes.length - 1) {
            const arrow = document.createElement('div');
            arrow.className = 'arrow';
            wrapper.appendChild(arrow);
        } else if (cyclePos > 0 && cyclePos < nodes.length) {
            // 环的箭头
            const cycleLabel = document.createElement('div');
            cycleLabel.className = 'cycle-label';
            cycleLabel.textContent = '↺ 环';
            wrapper.appendChild(cycleLabel);
        }

        container.appendChild(wrapper);
    }
}
```

### 13.4 步骤对象设计

```javascript
// 链表步骤对象
{
    codeLine: 3,
    variables: {
        slow: 0,
        fast: 0
    },
    listState: {
        nodes: [3, 2, 0, -4],  // 链表节点值
        slow: 0,               // slow 指针位置 (-1 表示不显示)
        fast: 0,               // fast 指针位置
        meet: null,            // 相遇点位置
        result: null,          // 环入口位置
        finding: false,        // 是否在找环入口
        index1: -1,            // 找入口时的 index1
        index2: -1,            // 找入口时的 index2
        cyclePos: 1             // 环入口位置（用于显示环标记）
    },
    desc: "第3行: 初始化 slow 和 fast"
}
```

### 13.5 环形链表状态说明

| 状态字段 | 类型 | 说明 |
|---------|------|------|
| nodes | Array | 链表节点值数组 |
| slow | number | slow 指针位置，-1 表示不显示 |
| fast | number | fast 指针位置，-1 表示不显示 |
| meet | number | 快慢指针相遇位置 |
| result | number | 环入口位置（找到时） |
| finding | boolean | 是否正在查找环入口 |
| index1 | number | 查找时的 index1 指针 |
| index2 | number | 查找时的 index2 指针 |
| cyclePos | number | 环入口位置（用于显示环标记） |

### 13.6 常见链表题目步骤生成模式

#### 环形链表检测（141题）
- 步骤1：初始化 slow = head, fast = head
- 步骤2-4：while 循环，每次 slow 移动一步，fast 移动两步
- 步骤5：检查是否相遇 (slow == fast)
- 步骤6：返回结果

#### 环形链表 II（142题）
- 步骤1-2：初始化 slow, fast
- 步骤3-8：快慢指针移动 + 检测相遇
- 步骤9-14：相遇后，两个指针各走一步找环入口
- 步骤15：返回环入口

#### 反转链表（206题）
- 步骤1：初始化 prev = null, curr = head
- 步骤2-4：while 循环，每次反转一个节点
- 步骤5：更新指针
- 步骤6：返回新头节点

### 13.7 初始化流程（重要！）

**【关键】必须使用 DOMContentLoaded 事件确保 DOM 加载完成后再执行初始化！**

```javascript
// 全局变量
let codeLines = [];
let currentStep = 0;
let steps = [];

// 初始化代码显示
function initCodeDisplay() {
    const codeDisplay = document.getElementById('codeDisplay');
    if (!codeDisplay) {
        console.error('找不到 codeDisplay 元素');
        return;
    }

    codeLines = codeText.split('\n');

    let html = '';
    for (let i = 0; i < codeLines.length; i++) {
        const lineNum = i + 1;
        const line = codeLines[i];
        const isComment = line.trim().startsWith('//') || line.trim().startsWith('/*');
        html += '<span class="code-line ' + (isComment ? 'comment' : '') + '" data-line="' + lineNum + '">' + escapeHtml(line) + '</span>';
    }
    codeDisplay.innerHTML = html;
}

function escapeHtml(text) {
    return text.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;');
}

// 重置函数
function reset() {
    stopPlay();
    currentStep = 0;
    currentParams = parseParams();
    steps = generateSteps(currentParams);
    updateUI();
}

// 页面加载完成后初始化 - 使用 DOMContentLoaded
window.addEventListener('DOMContentLoaded', function() {
    console.log('DOM加载完成');
    initCodeDisplay();
    reset();
});

// 备用：如果 DOMContentLoaded 已经触发
if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', function() {
        initCodeDisplay();
        reset();
    });
} else {
    // DOM 已经加载完成
    setTimeout(function() {
        initCodeDisplay();
        reset();
    }, 0);
}
```

**常见问题排查：**
1. 如果代码不显示，检查 initCodeDisplay() 是否在 DOM 加载后调用
2. 如果链表不显示，检查 listState 是否正确传递
3. 如果步骤不更新，检查 updateUI() 是否正确执行

### 13.8 代码行号映射强制规范（重要！）

**【关键】生成步骤时，必须先建立准确的代码行号映射表！**

#### 步骤1: 确定代码实际行号

在使用 `split('\n')` 分割代码后，**必须**手动验证每行代码对应的行号。示例：

```javascript
const codeText = `public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode slow = head;
        ListNode fast = slow;

        // 这题是找环的入口
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) {
                ListNode index1 = fast;
                ListNode index2 = head;
                // 两个指针，各走一步，直到相遇
                while (index1 != index2) {
                    index1 = index1.next;
                    index2 = index2.next;
                }
                return index1;
            }
        }
        return null;
    }
}`;

// 手动统计行号：
// 第1行: public class Solution {
// 第2行:     public ListNode detectCycle(ListNode head) {
// 第3行:         ListNode slow = head;
// 第4行:         ListNode fast = slow;
// 第5行: (空行)
// 第6行:         // 这题是找环的入口
// 第7行:         while (fast != null && fast.next != null) {
// 第8行:             slow = slow.next;
// 第9行:             fast = fast.next.next;
// ...以此类推
```

#### 步骤2: 在代码注释中记录行号映射

在生成步骤的函数中，**必须**在注释中明确标注每段代码对应的行号：

```javascript
// 生成步骤 - 代码行号与实际执行流程对应
function generateSteps(params) {
    const newSteps = [];

    // ========== 第2行: 方法入口 ==========
    newSteps.push({
        codeLine: 2,  // 必须是实际代码行号！
        desc: '第2行: public ListNode detectCycle(ListNode head)'
    });

    // ========== 第3行: slow = head ==========
    newSteps.push({
        codeLine: 3,
        desc: '第3行: ListNode slow = head'
    });

    // ========== 第7行: while 条件检查 ==========
    newSteps.push({
        codeLine: 7,
        desc: '第7行: while (fast != null && fast.next != null)'
    });
    // ...
}
```

#### 步骤3: 验证行号一致性

生成步骤后，**必须**添加验证代码：

```javascript
// 生成完步骤后验证
function validateSteps() {
    const codeLineCount = codeLines.length;  // 代码总行数
    const maxCodeLine = Math.max(...steps.map(s => s.codeLine));  // 步骤中最大行号
    const minCodeLine = Math.min(...steps.map(s => s.codeLine));  // 步骤中最小行号

    console.log('代码总行数:', codeLineCount);
    console.log('步骤 codeLine 范围:', minCodeLine, '-', maxCodeLine);

    // 验证1: codeLine 不能超过代码总行数
    if (maxCodeLine > codeLineCount) {
        console.error('错误: codeLine ' + maxCodeLine + ' 超过代码总行数 ' + codeLineCount);
        return false;
    }

    // 验证2: codeLine 不能小于1
    if (minCodeLine < 1) {
        console.error('错误: codeLine 小于1');
        return false;
    }

    // 验证3: 每个步骤的 codeLine 都必须有对应的代码行
    steps.forEach((step, idx) => {
        const lineEl = document.querySelector('.code-line[data-line="' + step.codeLine + '"]');
        if (!lineEl) {
            console.error('步骤 ' + (idx + 1) + ': 找不到 codeLine=' + step.codeLine + ' 的代码行');
        }
    });

    return true;
}
```

#### 常见错误：代码行号偏移

| 错误类型 | 原因 | 修复方法 |
|---------|------|---------|
| codeLine 比实际少2 | 忽略了类声明和空行 | 手动统计实际行号 |
| codeLine 错位 | 复制粘贴代码后未更新 | 在注释中标注行号 |
| codeLine 跳跃过大 | 跳过了某些代码行 | 检查每行代码是否都有对应步骤 |

#### 验证检查清单

生成 HTML 后，必须检查：
- [ ] 代码总行数与 `codeLines.length` 一致
- [ ] 每个步骤的 `codeLine` 在 1 到 `codeLines.length` 范围内
- [ ] 步骤执行顺序与代码逻辑一致
- [ ] 打开浏览器控制台无报错

## 15. 生成后验证检查流程（重要！）

每次生成 HTML 后，必须执行以下完整的验证检查流程，确保代码行对齐正确且功能正常。

### 13.1 代码行号映射表（必须创建）

在生成步骤之前，必须先建立代码行号映射表，确保每个 codeLine 都能对应到实际的代码行：

```javascript
// 代码行号映射 - 根据格式化后的代码创建
// 第1行: class Solution {
// 第2行:     public List<List<Integer>> threeSum(int[] nums) {
// 第3行:         List<List<Integer>> result = new ArrayList<>();
// 第4行:         Arrays.sort(nums);
// 第5行:         for (int i = 0; i < nums.length; i++) {
// 第6行:             // 减枝
// 第7行:             if (nums[i] > 0 || i > 0 && nums[i] == nums[i - 1])
// 第8行:                 continue;
// 第9行:             int left = i + 1;
// 第10行:            int right = nums.length - 1;
// ... 以此类推
```

**重要原则**：
- codeLine 从 1 开始计数（对应第一行代码）
- 空行不计入行号
- 注释行也算作有效代码行

### 13.2 步骤验证函数（必须添加）

在生成的 HTML 中必须包含验证函数：

```javascript
// 验证步骤与代码行对齐
function validateSteps() {
    const maxCodeLine = Math.max(...steps.map(s => s.codeLine));
    const minCodeLine = Math.min(...steps.map(s => s.codeLine));
    const codeLineCount = codeLines.length;

    console.log(`步骤验证: 共${steps.length}个步骤, codeLine范围: ${minCodeLine}-${maxCodeLine}, 代码总行数: ${codeLineCount}`);

    // 检查1: codeLine 不能超过代码总行数
    if (maxCodeLine > codeLineCount) {
        console.error('错误: codeLine ' + maxCodeLine + ' 超过代码总行数 ' + codeLineCount);
        return false;
    }

    // 检查2: codeLine 不能小于1
    if (minCodeLine < 1) {
        console.error('错误: codeLine 小于1');
        return false;
    }

    // 检查3: steps 不能为空
    if (steps.length === 0) {
        console.error('错误: 步骤数组为空');
        return false;
    }

    return true;
}
```

### 13.3 生成后验证清单

生成 HTML 后，必须执行以下检查：

| 检查项 | 验证方法 | 通过标准 |
|--------|----------|----------|
| 代码行数 | `codeLines.length` | ≥ 1 |
| 步骤数量 | `steps.length` | ≥ 1 |
| codeLine 范围 | `Math.max/min(...steps.map(s => s.codeLine))` | 在 1 到 codeLines.length 之间 |
| 每步有变量 | `step.variables` 存在 | 所有步骤都有 variables 对象 |
| 每步有描述 | `step.desc` 存在 | 所有步骤都有 desc 字符串 |
| 高亮行存在 | `document.querySelector('.code-line[data-line="' + step.codeLine + '"]')` | 能找到对应 DOM 元素 |

### 13.4 调试日志（必须添加）

在关键位置添加 console.log 便于排查问题：

```javascript
// 初始化时
console.log('代码总行数:', codeLines.length);

// 生成步骤后
console.log('生成步骤数:', steps.length);

// 每步更新时
console.log('当前步骤:', currentStep + 1, '/', steps.length, ', codeLine:', step.codeLine);

// 找不到高亮行时
if (!activeLine) {
    console.error('找不到codeLine对应的代码行:', step.codeLine);
}
```

### 13.5 常见问题自动修复

| 问题 | 修复方法 |
|------|----------|
| codeLine 超出范围 | 将步骤的 codeLine 调整为代码有效行范围 (1 到 codeLines.length) |
| 步骤缺失 | 为代码的每一行（尤其是循环体）生成对应步骤 |
| 高亮行不显示 | 检查 activeLine 是否为 null，确认 codeLine 与 data-line 属性匹配 |
| 步骤跳跃过大 | 检查步骤生成逻辑，确保每行代码都有对应步骤 |

### 13.6 验证流程图

```
开始生成 HTML
    ↓
创建代码行号映射表（明确每行代码对应的行号）
    ↓
生成步骤数组（每个步骤包含 codeLine）
    ↓
执行 validateSteps() 验证
    ↓
┌─ 验证通过 ─→ 输出 HTML 文件
│
└─ 验证失败 ─→ 修复问题后重新验证
    │
    ├── 检查 codeLine 范围
    ├── 检查步骤数量
    └── 检查高亮行是否存在
```

### 13.7 HTML 文件头部注释验证报告

在生成的 HTML 文件中添加验证报告注释：

```html
<!--
代码行对齐验证报告:
- 代码总行数: 33
- 步骤总数: 60
- codeLine 范围: 1 - 31
- 验证函数: validateSteps() 已添加
- 调试日志: console.log 已添加
- 验证状态: 通过
-->
```

## 16. 常见错误与预防措施（重要！）

本章节总结了在生成可视化 HTML 时经常出现的错误，以及对应的预防方法。

### 14.1 JavaScript 语法错误

#### 错误1: Assignment to constant variable

**错误原因**：在循环中修改用 `const` 声明的变量

**错误代码**：
```javascript
const left = i + 1;
const right = n - 1;
// ... 后面有 left++ 或 right--
left++;
right--;
```

**修复方法**：
```javascript
let left = i + 1;  // 使用 let 而非 const
let right = n - 1;
// ... 后面可以修改
left++;
right--;
```

**预防措施**：
- 在循环或需要修改的变量必须使用 `let` 声明
- 使用 `const` 前先确认该变量不会被重新赋值

### 14.2 代码行步骤不完整

#### 错误2: 缺少关键代码行的步骤

**常见缺失的行号**：
| 代码类型 | 容易缺失的行 |
|----------|-------------|
| 变量初始化 | 第9行: `int left = i + 1` |
| while 条件 | 第11行: `while (left < right)` |
| if 条件 | 第13行: `if (sum == 0)` |
| else if | 第24行: `else if (sum < 0)` |
| else | 第26行: `else` |
| 去重循环 | 第16、19行的 while 条件判断 |
| 指针移动 | 第22、23行的 `left++`、`right--` |

**预防方法**：在生成步骤时，必须为以下每一行生成对应的步骤：
1. **变量声明行** - 任何变量初始化都要有步骤
2. **循环条件行** - `while`、`for`、`if` 条件判断都要有步骤
3. **循环体内部** - 每次迭代都要生成完整步骤
4. **指针移动行** - `left++`、`right--` 等操作要有步骤

**完整步骤生成模板**：
```javascript
// 外层 for 循环
for (let i = 0; i < n; i++) {
    // 第X行：for 循环开始
    newSteps.push({ codeLine: X, ... });

    // 第Y行：if 条件判断
    if (condition) {
        newSteps.push({ codeLine: Y, ... });

        if (condition2) {
            // 第Z行：嵌套 if
            newSteps.push({ codeLine: Z, ... });
        }
    }

    // 内层 while 循环
    while (left < right) {
        // 第A行：while 条件
        newSteps.push({ codeLine: A, ... });

        // 第B行：循环体操作
        newSteps.push({ codeLine: B, ... });
    }

    // 第C行：while 结束
    newSteps.push({ codeLine: C, ... });
}
```

### 14.3 步骤与代码行不对齐

#### 错误3: codeLine 超出范围

**错误原因**：步骤中的 `codeLine` 超过了代码总行数

**预防措施**：
1. 先确定代码总行数：`const codeLines = codeText.split('\n');`
2. 确保每个步骤的 `codeLine` 都在 1 到 `codeLines.length` 之间
3. 使用 validateSteps() 函数验证

#### 错误4: 缺少步骤导致代码行跳跃

**错误原因**：某些代码行没有对应步骤，导致高亮行跳跃过大

**预防措施**：
1. 为每一行有效代码（不含空行）都生成步骤
2. 在生成后使用代码行数检查：
```javascript
// 检查所有代码行是否都有对应步骤
for (let lineNum = 1; lineNum <= codeLines.length; lineNum++) {
    const hasStep = steps.some(s => s.codeLine === lineNum);
    if (!hasStep) {
        console.warn(`警告: 代码行 ${lineNum} 没有对应步骤`);
    }
}
```

### 14.4 逐行执行检查清单

在生成步骤前，先列出代码的完整行号映射：

```javascript
// 完整行号映射表示例
const lineMapping = [
    'class Solution {',
    '    public List<List<Integer>> threeSum(int[] nums) {',
    '        List<List<Integer>> result = new ArrayList<>();',  // 行3
    '        Arrays.sort(nums);',                                // 行4
    '        for (int i = 0; i < nums.length; i++) {',         // 行5
    '            // 减枝',                                        // 行6 (注释)
    '            if (nums[i] > 0 || i > 0 && nums[i] == nums[i - 1])', // 行7
    '                continue;',                                  // 行8
    '            int left = i + 1;',                             // 行9
    '            int right = nums.length - 1;',                   // 行10
    '            while (left < right) {',                         // 行11
    // ... 以此类推
];
```

然后按顺序为每一行生成步骤，确保不遗漏。

### 14.5 完整代码示例（正确版本）

```javascript
function generateSteps(arr) {
    const sortedArr = [...arr].sort((a, b) => a - b);
    const n = sortedArr.length;
    const results = [];
    const newSteps = [];

    // 第3行：创建结果列表
    newSteps.push({
        codeLine: 3,
        variables: { result: '[]' },
        desc: '第3行: 创建结果列表',
        results: []
    });

    // 第4行：排序
    newSteps.push({
        codeLine: 4,
        variables: { nums: sortedArr },
        desc: '第4行: Arrays.sort(nums)',
        results: []
    });

    // 外层循环
    for (let i = 0; i < n; i++) {
        // 第5行：for 循环
        newSteps.push({
            codeLine: 5,
            variables: { i: i, n: n },
            desc: `第5行: i = ${i}`,
            results: [...results]
        });

        // 第7行：if 条件
        const shouldSkip = sortedArr[i] > 0 || (i > 0 && sortedArr[i] === sortedArr[i-1]);
        newSteps.push({
            codeLine: 7,
            variables: { 'nums[i]': sortedArr[i], shouldSkip },
            desc: `第7行: if 条件检查`,
            results: [...results]
        });

        if (shouldSkip) {
            // 第8行：continue
            newSteps.push({
                codeLine: 8,
                desc: '第8行: continue',
                results: [...results]
            });
            continue;
        }

        // 第9行：left 初始化 (注意用 let!)
        let left = i + 1;
        newSteps.push({
            codeLine: 9,
            variables: { left },
            desc: `第9行: left = ${left}`,
            results: [...results]
        });

        // 第10行：right 初始化 (注意用 let!)
        let right = n - 1;
        newSteps.push({
            codeLine: 10,
            variables: { right },
            desc: `第10行: right = ${right}`,
            results: [...results]
        });

        // 内层 while 循环
        while (left < right) {
            // 第11行：while 条件
            newSteps.push({
                codeLine: 11,
                variables: { left, right, condition: left < right },
                desc: `第11行: while (${left} < ${right})`,
                results: [...results]
            });

            // ... 更多步骤
        }

        // 第29行：while 结束
        newSteps.push({
            codeLine: 29,
            desc: '第29行: while 结束',
            results: [...results]
        });
    }

    // 第31行：return
    newSteps.push({
        codeLine: 31,
        desc: `第31行: return ${results.length}个结果`,
        results: [...results]
    });

    return newSteps;
}
```

### 14.6 生成后必做检查

完成 HTML 生成后，必须执行以下检查：

1. **语法检查**：确保 JavaScript 无语法错误
2. **行数检查**：`codeLines.length` 与实际代码行数一致
3. **步骤检查**：`steps.length` > 0
4. **codeLine 范围检查**：
   ```javascript
   const maxLine = Math.max(...steps.map(s => s.codeLine));
   const minLine = Math.min(...steps.map(s => s.codeLine));
   if (maxLine > codeLines.length || minLine < 1) {
       console.error('codeLine 范围错误!');
   }
   ```
5. **每行代码检查**：确保每个代码行都有对应步骤
6. **变量声明检查**：确保会修改的变量用 `let` 而非 `const`
7. **控制台测试**：在浏览器控制台查看是否有报错

### 14.7 代码执行流程对齐（重要！）

**【关键】生成的步骤必须严格遵循代码的实际执行顺序！**

对于递归函数（如二叉树遍历、平衡二叉树等），必须按照以下顺序生成步骤：

#### 14.7.1 代码行号与执行顺序映射

在生成步骤前，必须先建立代码执行流程图：

```javascript
// 示例：平衡二叉树 isBalanced/balance 方法
// 代码行号对应执行顺序：

// isBalanced 方法:
5: public boolean isBalanced(TreeNode root) {  // 方法入口
6:     balance(root);  // ← 首先执行：调用 balance
7:     return len <= 1;  // ← 最后执行：返回结果

// balance 方法 (递归):
10: private int balance(TreeNode root) {  // 方法入口
11:     if (root == null) {  // ← 检查条件
12:         return 0;  // ← 空节点返回
13:     }  // ← if 结束

15:     int left = balance(root.left);  // ← 递归调用左子树
16:     int right = balance(root.right);  // ← 递归调用右子树
17:     if (Math.abs(left - right) > 1) {  // ← 检查平衡
18:         len = Math.max(len, ...);  // ← 更新不平衡程度
19:     }  // ← if 结束

20:     return Math.max(left, right) + 1;  // ← 返回高度
21: }  // ← 方法结束
```

#### 14.7.2 递归执行流程

递归函数的步骤生成必须严格按照调用栈顺序：

```
balance(root)
  ├── 11: if (root == null) → false, 继续
  ├── 15: int left = balance(root.left)
  │     ├── 11: if (root.left == null) → 检查
  │     ├── 15: int left = balance(root.left.left) → 递归
  │     ├── 16: int right = balance(root.left.right) → 递归
  │     ├── 17: if (Math.abs(left - right) > 1) → 检查
  │     └── 20: return height
  ├── 16: int right = balance(root.right)
  │     ├── 类似左子树...
  ├── 17: if (Math.abs(left - right) > 1) → 检查平衡
  ├── 18: len = Math.max(...) → 更新（如果需要）
  └── 20: return height
```

#### 14.7.3 正确生成步骤的示例

```javascript
function generateSteps(root) {
    const steps = [];

    // 第6行: 调用 balance(root)
    steps.push({
        codeLine: 6,
        desc: "第6行: 调用 balance(root)"
    });

    // 进入递归
    const recursiveSteps = generateBalanceRecursive(root);
    steps.push(...recursiveSteps);

    // 第7行: 返回结果
    steps.push({
        codeLine: 7,
        desc: "第7行: return len <= 1"
    });

    return steps;
}

function generateBalanceRecursive(node) {
    const steps = [];

    if (!node) {
        // 第11行: if (root == null)
        steps.push({ codeLine: 11, desc: "第11行: 检查 root == null" });
        // 第12行: return 0
        steps.push({ codeLine: 12, desc: "第12行: return 0" });
        return steps;
    }

    // 第10行: 进入方法
    steps.push({ codeLine: 10, desc: `第10行: 进入 balance(${node.val})` });

    // 第11行: if (root == null)
    steps.push({ codeLine: 11, desc: "第11行: 检查 root == null" });

    // 第15行: 递归调用左子树
    steps.push({ codeLine: 15, desc: "第15行: int left = balance(root.left)" });
    steps.push(...generateBalanceRecursive(node.left));

    // 第16行: 递归调用右子树
    steps.push({ codeLine: 16, desc: "第16行: int right = balance(root.right)" });
    steps.push(...generateBalanceRecursive(node.right));

    // 第17行: 检查平衡
    steps.push({ codeLine: 17, desc: "第17行: 检查平衡条件" });

    // 第18行: 更新 len (如果需要)
    // 根据条件添加...

    // 第20行: 返回高度
    steps.push({ codeLine: 20, desc: "第20行: return height" });

    return steps;
}
```

#### 14.7.4 常见错误与修正

**错误**：步骤的 codeLine 顺序不正确，例如：
- 跳过了递归调用前的代码行
- 递归返回后的步骤顺序错误
- 没有为每个方法调用生成对应的步骤

**修正方法**：
1. 先画出代码执行流程图
2. 按照流程图顺序生成步骤
3. 使用递归函数时，确保每层递归都生成完整的步骤序列
