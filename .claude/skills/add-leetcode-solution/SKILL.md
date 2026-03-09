---
name: add-leetcode-solution
description: 自动添加 LeetCode 题解。使用浏览器访问 leetcode.cn 获取题目信息和 Java 题解代码，生成可视化 HTML，并自动填入管理后台完成题解新增（状态为草稿）。适用于用户想要快速添加一个 LeetCode 题解到可视化平台的场景。
---

# 自动添加 LeetCode 题解

自动化完成以下工作：
1. 使用浏览器访问 leetcode.cn 获取题目信息和 Java 题解代码
2. 调用 gen-solution-viz skill 生成可视化 HTML
3. 登录管理后台，新增题解（状态为草稿，未发布）

## 触发条件

当用户请求以下内容时使用此 skill：
- "添加一个 leetcode 题解"
- "帮我添加题解"
- "上传一个新题解"
- "我要添加一个算法题解"
- /add-solution

## 执行流程

### Step 1: 获取 LeetCode 题目信息

首先需要确定题目信息。请询问用户以下信息：
- **LeetCode 题目编号或名称**（必填）：如 "3"、"无重复字符的最长子串"、"longest-substring-without-repeating-characters"

### Step 2: 使用浏览器访问 leetcode.cn 获取题解代码

**【重要】必须使用浏览器 MCP 工具，不能自己生成代码！**

1. 使用浏览器访问题目页面：`https://leetcode.cn/problems/<题目slug>/`
   - 如果不知道 slug，可以直接搜索题目名称

2. 获取以下信息：
   - 题目编号（questionId）
   - 题目标题（中英文）
   - 难度（简单/中等/困难）
   - 题目描述
   - **Java 题解代码**：点击"题解"标签，查找 Java 语言的精选题解

3. 如果页面有反爬限制（如需要登录、验证码等）：
   - 尝试等待页面加载完成
   - 可以先获取题目基本信息，题解代码可以从 Java 代码模板结合算法知识生成
   - 或者询问用户是否可以直接提供题解代码

4. 获取 Java 代码的技巧：

   **【重要】优化后的获取流程：**

   a) **方法一：直接从题目页面获取（推荐）**
      - 访问题目描述页面：`https://leetcode.cn/problems/<slug>/description/`
      - 在代码编辑区域点击语言选择器，选择 Java
      - 使用 JavaScript 从编辑器获取代码：
        ```javascript
        // 获取编辑器中的代码
        document.querySelector('.monaco-editor .view-lines')?.textContent
        ```

   b) **方法二：题解列表获取**
      - 访问题解页面：`https://leetcode.cn/problems/<slug>/solutions/`
      - 点击"不限"筛选按钮，选择"Java"语言
      - **【关键】不要点击题解标题**，因为点击标题会弹出用户信息
      - 直接在页面上查找包含 `class Solution` 和 `isSameTree`（方法名）的代码块
      - 使用 JavaScript 提取：
        ```javascript
        // 查找页面中所有包含 class Solution 的代码
        const codes = document.querySelectorAll('pre, code');
        for (const code of codes) {
            const text = code.textContent;
            if (text.includes('class Solution') && text.includes('方法名')) {
                return text; // 返回找到的代码
            }
        }
        ```

   c) **方法三：如果官方题解没有 Java**
      - 尝试其他热门题解（如"画手大鹏"、"labuladong"等）
      - 如果实在找不到 Java 代码，可以使用题目页面自带的代码模板

   **【关键】不要点击题解标题！**
   - 点击标题会弹出用户信息弹窗，而不是打开题解详情
   - 应该直接使用 JavaScript 从页面提取代码

### Step 3: 生成可视化 HTML

调用 gen-solution-viz skill 生成可视化 HTML：

1. 准备题目信息：
   - 题目编号和名称
   - 算法类型（根据题目推断：数组、哈希表、链表、树、动态规划等）
   - 难度（1-简单，2-中等，3-困难）
   - Java 题解代码（从浏览器获取的代码）

2. 调用 skill：
   - 使用 Skill 工具调用 `gen-solution-viz`
   - 参数：题目名称、算法类型、难度、代码

3. 获取生成的 HTML 文件路径

### Step 4: 登录管理后台

1. 访问登录页：`http://localhost:8083`
2. 使用 API 登录获取 token：
   ```bash
   curl -X POST http://localhost:8080/admin/api/auth/login \
     -H "Content-Type: application/json" \
     -d '{"username":"admin","password":"admin123"}'
   ```
3. 将 token 设置到浏览器 localStorage

### Step 5: 新增题解

1. 使用 API 创建题解：`POST http://localhost:8080/admin/solutions`
   - 请求头：`Authorization: Bearer {token}`
   - 请求体：
     ```json
     {
       "title": "{题目编号}. {题目名称} - {难度}",
       "leetcodeQuestionId": 题目编号,
       "leetcodeUrl": "https://leetcode.cn/problems/{slug}/",
       "difficulty": 难度(1-3),
       "htmlFileUrl": "/solutions/{文件名}.html",
       "description": "简要描述",
       "status": 0,
       "tagIds": [标签ID]
     }
     ```

2. **【强制】status 必须设为 0（草稿/未发布）**，不允许设为已发布！

### Step 6: 验证

确认题解已成功添加：
1. 访问管理后台题解列表 `http://localhost:8083/admin/solutions`，确认新题解显示
2. 在管理后台点击"编辑"按钮，预览可视化效果
3. 确认效果满意后，手动将状态改为"已发布"

## 注意事项

1. **【强制】必须使用浏览器获取代码**：不能自己生成题解代码，必须从 leetcode.cn 获取
2. **反爬策略应对**：
   - 页面加载慢时使用 `wait_for` 等待
   - 如果需要登录，可以先获取题目基本信息
   - 题解代码可以从页面获取或用户提供
3. **登录凭证**：admin/admin123
4. **HTML 文件**：确保生成的 HTML 文件路径正确且可访问
5. **标签匹配**：根据算法类型自动推断合适的标签

## 错误处理

如果某一步骤失败：
1. 浏览器访问失败 → 尝试刷新页面或等待加载
2. 获取代码失败 → 询问用户直接提供题解代码
3. 生成 HTML 失败 → 报告错误，请用户手动处理
4. 登录失败 → 提示用户检查账号密码
5. 提交失败 → 显示错误信息，可手动重试

## 输出格式

完成后告知用户：
1. 题解标题
2. 难度
3. 状态：未发布（草稿）
4. 可视化 HTML 文件路径
5. 管理后台编辑链接：`http://localhost:8083/admin/solutions/{id}/edit`
6. 提醒用户在管理后台预览并手动发布
