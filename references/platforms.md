# 平台配置

## 各平台搜索语法速查

| 平台 | 支持语法 | 不支持 | 关键词策略 | 示例 |
|------|---------|--------|-----------|------|
| **猎聘** | 单关键词 + 标签匹配 | 复杂布尔运算 | 关键词简化（1-2个），依赖筛选器和标签 | `VLA 具身智能` |
| **前程无忧** | 空格分隔关键词（松散 OR） | AND/OR/NOT 布尔运算（会被当成普通文本） | 2-3 个核心词，空格分隔，靠筛选器收窄 | `VLA 机器人` |
| **Boss 直聘** | 关键词 + 筛选器组合 | 布尔运算、精确匹配 | 2-3个核心词，充分利用筛选器 | `VLA 机器人` |
| **LinkedIn** | 字段搜索 + 布尔运算（最强大） | — | 可用 title/skills/education/company 等字段精确搜索 | `title:"工程师" AND skills:"Verilog" AND location:"北京"` |

**Phase 1 生成搜索方案时，必须根据所选平台适配搜索语法：**
- 猎聘 → 简化关键词，1-2 个词，其余条件靠筛选器
- 前程无忧 → 2-3 个核心词空格分隔，不支持 AND/OR/NOT，靠筛选器收窄结果
- Boss → 2-3 个核心词，不要堆太多
- LinkedIn → 字段搜索语法，精确匹配

---

## 猎聘网（liepin.com）

### 基本导航
- 企业端首页：`https://lpt.liepin.com`
- 搜索人才页面：`https://lpt.liepin.com/search`（⚠️ 不是 `/resume/search`，那个会 404）
- 首页有「搜索人才」链接，可直接点击跳转到 `/search`

### 登录
- 必须登录才能搜索人才，未登录时首页只显示「登录/注册」
- 登录方式：手机扫码或验证码
- agent-browser 操作：需用 `--headed` 模式打开浏览器让用户手动登录
- 登录后页面顶部会出现用户名 +「搜索人才」入口

### 搜索与筛选
- 主搜索栏：顶部居中长文本输入框，class 为 `.searchInput--KgDn1`
- ⚠️ **React 受控组件输入技巧**：不能直接 `fill` 或设 `input.value`，必须用 `nativeInputValueSetter`：
```js
const el = document.querySelector('.searchInput--KgDn1');
const setter = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, 'value').set;
setter.call(el, '搜索词');
el.dispatchEvent(new Event('input', {bubbles: true}));
el.dispatchEvent(new Event('change', {bubbles: true}));
```
- 城市筛选：主搜索栏下方城市标签
- 经验筛选：不限/在校应届/1-3年/3-5年/5-10年/自定义
- 学历筛选：不限/本科/硕士/博士博士后等
- 搜索按钮：遍历 `document.querySelectorAll('button')` 找 `textContent.trim() === '搜索'`，用 JS eval 点击比 `click @ref` 更可靠：
```js
Array.from(document.querySelectorAll('button')).find(b => b.textContent.trim() === '搜索')?.click()
```

### 列表页
- **候选人卡片容器**：`.resumeCardWrap--FcnzW`，每页 20 张
- **信息密度：中等偏薄**
  - 展示：姓名(脱敏)、年龄、工作年限、学历标签、当前公司、职位、过往履历（公司+职位+时间）、教育背景（学校+专业+学历）、期望薪资、期望城市
  - ⚠️ 工作经历只有一行公司+职位，**缺少项目细节、论文、技术栈深度**
  - 列表页信息足以做初筛（是否对口），但**不足以输出深度分析报告**
- **候选人信息提取选择器**：
  - 简历编号：`card.getAttribute('data-resumeidencode')` — ⚠️ **必须提取**，用于报告输出和后续下载/联系
  - 详情页 URL：`card.getAttribute('data-resumeurl')` — 完整 URL 含追踪参数；精简版为 `https://lpt.liepin.com/cvview/showresumedetail?resIdEncode=${resumeId}`
  - 姓名：`.nest-resume-personal-name`
  - 基本信息（年龄/学历/城市等）：`.nest-resume-personal-detail`
  - 期望：`.nest-resume-personal-expect`
  - 技能：`.nest-resume-personal-skills`
  - 工作经历：`.nest-resume-work-item`（可多个）
  - 教育经历：`.nest-resume-edu-item`（可多个）
- **完整提取脚本**：
```js
Array.from(document.querySelectorAll('.resumeCardWrap--FcnzW')).map((card, i) => {
  const resumeId = card.getAttribute('data-resumeidencode') || '';
  const resumeUrl = `https://lpt.liepin.com/cvview/showresumedetail?resIdEncode=${resumeId}`;
  const name = card.querySelector('.nest-resume-personal-name')?.textContent?.trim() || '';
  const detail = card.querySelector('.nest-resume-personal-detail')?.textContent?.trim() || '';
  const expect = card.querySelector('.nest-resume-personal-expect')?.textContent?.trim() || '';
  const skills = card.querySelector('.nest-resume-personal-skills')?.textContent?.trim() || '';
  const works = Array.from(card.querySelectorAll('.nest-resume-work-item')).map(w => w.textContent?.trim()?.replace(/\s+/g,' ')).join(' | ');
  const edus = Array.from(card.querySelectorAll('.nest-resume-edu-item')).map(e => e.textContent?.trim()?.replace(/\s+/g,' ')).join(' | ');
  return {i: i+1, resumeId, resumeUrl, name, detail, expect, skills, works, edus};
})
```

### 分页
- 分页按钮 class：`.ant-lpt-pagination-item-N`（N 为页码，从 1 开始）
- 当前页会加上 active 状态 class
- 点击第 N 页：`document.querySelector('.ant-lpt-pagination-item-N')?.click()`
- ⚠️ 翻页状态会被保留：点击搜索按钮后可能自动跳到之前浏览的页码

### 详情页（必须访问，用于强推人选深读）
- **触发条件**：初筛标记为「强推」或「可联系」的候选人
- **触发方式**：点击 `.resumeCardContent--A03AZ`（卡片内容区域），非整个 `.resumeCardWrap`
- **展现形式**：页面内 Modal 弹窗，URL 加 `#preview`，不跳转新页面
- **详情容器**：`.resume-detail-modal-wrap`（ant-lpt-modal-wrap）
- **内容提取**：`document.querySelector('.resume-detail-modal-wrap').innerText`
  - 包含：完整项目描述、个人贡献、量化业绩、技术创新点、教育详情
  - 信息量远超列表页，**强推人选必须进详情页阅读**
- **操作流程**：
  1. 点击候选人卡片内容区域 `.resumeCardContent--A03AZ`
  2. 等待 2-3 秒让弹窗加载
  3. 提取详情：`document.querySelector('.resume-detail-modal-wrap').innerText`
  4. 内容可能较长，分段提取：`.substring(0, 3000)` / `.substring(3000, 6000)`
  5. 关闭弹窗：`document.querySelector('.resume-detail-modal-wrap .ant-lpt-modal-close')?.click()`
  6. 等待 2-3 秒后继续下一位
- ⚠️ **注意事项**：
  - 关闭弹窗后分页器可能消失，需重新搜索或翻页
  - 连续快速点击多张卡片可能触发反爬，建议每次间隔 2-3 秒
  - 部分候选人详情需「获取简历」权限，可能需要额外操作

### 已知坑 & 经验
1. `agent-browser click @ref` 的 ref 在页面变化后会失效，**优先用 JS eval 操作**
2. `agent-browser fill` 对 React 受控组件无效，必须用 nativeInputValueSetter
3. `agent-browser snapshot -i` 可获取交互元素列表，但输出可能被截断
4. 页面加载后用 `agent-browser wait --load networkidle` 等待完成
5. 搜索按钮点击后需要等待 2-3 秒让结果渲染
6. 猎聘搜索结果总数不精确显示，只能通过分页估算
7. CSS 选择器中的 hash 后缀（如 `--FcnzW`、`--KgDn1`）可能在猎聘发版后变化，如选择器失效需重新 snapshot 确认
8. **直接用 resumeUrl 打开详情页会触发付费墙**（「今日简历查看数已达上限」），只能通过搜索结果页点击卡片弹窗查看详情，不能走独立 URL

### 平台自带 AI 工具
- 猎聘「AI快读」：可批量快读最多1000份简历
- 建议：可作为粗筛前置步骤，再用本 SOP 精筛
- 局限：通用 AI 可能不理解 VLA/Diffusion Policy 等细分领域关键词

---

## Boss直聘（zhipin.com）

*待补充：如需使用 Boss 直聘，请在此添加页面操作地图。*

---

## 前程无忧（51job.com）

**企业端入口：** `https://ehire.51job.com/`

### 基本导航
- 企业端首页：`https://ehire.51job.com/`
- 搜索页 URL：`https://ehire.51job.com/Revision/talent/search`
- ⚠️ **搜索页必须从工作台左侧导航「人才搜索」进入**，直接用 URL 打开会得到空白 SPA 页面（0 个交互元素）
- ⚠️ `/Revision/search/resumesearch` 和 `/Revision/search/resumelist` 也会空白

### 登录
- 必须登录才能搜索人才
- 登录方式：账号密码 / 手机验证码
- agent-browser 操作：需用 `--headed` 模式打开浏览器让用户手动登录
- 登录后从左侧导航菜单点击「人才搜索」进入搜索页

### 搜索与筛选
- 搜索输入框：标准 textbox，可直接 `fill` 输入（非 React 受控组件，比猎聘简单）
- 搜索按钮：`page.getByRole('button', {name: '搜索'}).click()` 最可靠
- ⚠️ **不支持 AND/OR/NOT 布尔搜索**，空格 = 松散 OR，`(具身智能 OR VLA)` 会被当成普通文本
- 城市筛选：点「居住地」按钮弹窗，选城市后点「确定」
- 其他筛选：工作年限、年龄、学历、学校性质等
- ⚠️ snapshot 方式找搜索输入框/按钮 ref 不稳定（页面变化后 ref 失效），优先用 `getByRole` 定位

### 搜索词策略

**51job 搜索语法特点：** 空格分隔 = 松散 OR，不支持布尔运算。

| 搜索词 | 结果数 | 信噪比 | 推荐度 | 备注 |
|--------|--------|--------|--------|------|
| `VLA 机器人` | ~1000 | ★★★★ | **首选** | VLA/具身智能方向最佳平衡 |
| `OpenVLA 机器人` | ~1000 | ★★★ | 补充 | 有补充价值 |
| `具身智能 算法` | ~1144 | ★ | ❌ 慎用 | 信噪比极差，大量传统算法 |
| `(具身智能 OR VLA) AND (算法 OR 研究员)` | 5000+ | ★ | ❌ 不用 | 布尔语法不生效 |

### 列表页
- 候选人信息提取：`document.body.innerText`（全页面文本提取最可靠）
- CSS Modules 动态命名，没有统一的候选人卡片 class
- 信息密度：中等，含姓名、年龄、学历、当前公司、期望薪资等

### 详情页（必须访问，用于强推人选深读）
- **触发方式**：点击候选人姓名，会打开新 tab（非弹窗，与猎聘不同）
- **新 tab 捕获**：`Promise.all([page.context().waitForEvent('page'), link.click()])` 是可靠方式
- **姓名定位**：`page.locator('text=XX').first()` 或 `document.querySelector('[title=XXX]')`
- **内容提取**：`document.body.innerText.substring(0, 8000)` 获取完整简历
- **resumeId**：在 URL 参数中 `resumeId=XXXX`
- **tab 管理**：用 `tab-select` 切换到新 tab 提取，`tab-close` 关闭返回
- ⚠️ 详情页会有大量广告，不影响数据提取

### 已知坑 & 经验
1. ⚠️ **搜索页必须从左侧导航进入**，直接 URL 打开得到空白 SPA 页面
2. ⚠️ **不支持 AND/OR/NOT 布尔搜索**，空格 = 松散 OR
3. snapshot ref 不稳定，优先用 `getByRole` 定位
4. PowerShell 中 `run-code` 的分号会被截断，复杂 JS 用脚本文件或简化写法
5. CSS Modules 动态命名，没有统一的候选人卡片 class，`innerText` 提取更可靠
6. 硬科技/嵌入式方向人才极度稀缺
