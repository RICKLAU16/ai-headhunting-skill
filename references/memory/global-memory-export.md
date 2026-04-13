# 猎聘 + 前程无忧寻访实战记忆

_来源：RB-005 长期记忆 MEMORY.md，2026-04-10/11 实战积累_

---

## 猎聘企业端操作笔记（2026-04-10 实战验证）

### 基本导航
- 企业端首页：`https://lpt.liepin.com`
- 搜索人才页面：`https://lpt.liepin.com/search`（⚠️ 不是 `/resume/search`，那个会 404）
- 首页有"搜索人才"链接，可直接点击跳转到 `/search`

### 登录
- 必须登录才能搜索人才，未登录时首页只显示"登录/注册"
- 登录方式：手机扫码或验证码
- playwright-cli 操作：需用 `--headed` 模式打开浏览器手动登录
- 登录后页面顶部会出现用户名 + "搜索人才"入口
- ⚠️ in-memory 浏览器会话容易过期，建议操作紧凑、不要间隔太久
- ⚠️ 如遇 `#login` hash 跳转，说明会话过期，需重新登录

### 搜索框操作
- 搜索输入框：通过 snapshot 找 `textbox` 类型 + placeholder 含"搜职位"
- ⚠️ React 受控组件，`fill` 命令能设置 DOM 值但 React 状态可能不同步
- 可靠方案：`fill e174 "搜索词"` → 检查 snapshot 确认值已生效 → 点击搜索
- React nativeInputValueSetter 备用方案（需 JS eval，playwright-cli 中文 eval 有序列化问题）

### 搜索按钮
- snapshot 中 button `ref=e180`（名称"搜索"）
- 点击方式：`playwright-cli click e180`
- ⚠️ 未读消息弹窗可能遮挡搜索按钮，需先关闭弹窗（按 ESC 或移除 DOM）

### 搜索结果页
- 候选人卡片容器：`.resumeCardWrap--FcnzW`，每页 20 张
- 候选人信息提取选择器：
  - 姓名：`.nest-resume-personal-name`
  - 基本信息（年龄/学历/城市等）：`.nest-resume-personal-detail`
  - 期望：`.nest-resume-personal-expect`
  - 技能：`.nest-resume-personal-skills`
  - 工作经历：`.nest-resume-work-item`（可多个）
  - 教育经历：`.nest-resume-edu-item`（可多个）

### 分页
- 分页按钮 class：`.ant-lpt-pagination-item-N`（N 为页码，从 1 开始）
- 当前页会加上 active 状态 class
- 点击第 N 页：`document.querySelector('.ant-lpt-pagination-item-N')?.click()`

### 简历详情页
- **触发方式**：点击 `.resumeCardContent--A03AZ`（卡片内容区域），非整个 `.resumeCardWrap`
- **展现形式**：页面内 Modal 弹窗，URL 加 `#preview`，不跳转新页面
- **详情容器**：`.resume-detail-modal-wrap`
- **内容提取**：`document.querySelector('.resume-detail-modal-wrap').innerText`
- **简历编号提取**：`card.getAttribute('data-resumeidencode')`
  - 详情页精简 URL：`https://lpt.liepin.com/cvview/showresumedetail?resIdEncode=${resumeId}`
- **关闭弹窗**：`document.querySelector('.resume-detail-modal-wrap .ant-lpt-modal-close')?.click()`
- ⚠️ 关闭弹窗后分页器可能消失，需重新搜索或翻页
- ⚠️ 连续快速点击多张卡片可能触发反爬，建议每次间隔 2-3 秒
- ⚠️ **直接用 resumeUrl 打开详情页会触发付费墙**

### 已知坑 & 经验
1. `playwright-cli click` 的 ref 在页面变化后会失效，优先每次 snapshot 后重新获取
2. `playwright-cli eval` 不支持中文/复杂函数序列化
3. 页面加载后等 2-3 秒让结果渲染
4. 搜索结果总数不精确显示，只能通过分页估算
5. `--profile` 参数指向完整 `User Data` 目录时可能因 Chrome 锁文件失败，需关闭所有 Chrome 进程
6. in-memory 浏览器会话超时快，操作间隔不能超过约 5 分钟

---

## 前程无忧操作地图（2026-04-10 首次探索 + 04-11 深度寻访）

### 基本导航
- 企业端：https://ehire.51job.com/
- 搜索页 URL：`https://ehire.51job.com/Revision/talent/search`
  - ⚠️ 必须从工作台左侧导航「人才搜索」进入，直接 URL 打开会得到空白 SPA
- 搜索框：标准 textbox，可直接 fill 输入（非 React 受控组件）
- 城市筛选：点「居住地」按钮弹窗，选城市后点「确定」
- 搜索按钮：`page.getByRole('button', {name: '搜索'}).click()` 最可靠

### 搜索语法
- **51job 不支持 AND/OR/NOT 布尔搜索**
- 空格 = 松散 OR
- `(具身智能 OR VLA)` 会被当成普通文本
- 有效搜索词策略需靠精准关键词组合

### 结果提取
- 候选人卡片在 `[class*=search]` 容器内，innerText 提取最可靠
- **详情页**：点击候选人姓名开新 tab（非弹窗）
- 详情页提取：`document.body.innerText.substring(0, 8000)` 获取完整简历
- resumeId 在 URL 参数中：`resumeId=XXXX`

### 已知坑
1. 没有像猎聘那样统一的候选人卡片 class，CSS Modules 动态命名
2. 直接用 URL `/Revision/search/resumesearch` 打开会空白，必须从导航菜单进入
3. snapshot 方式找搜索输入框/按钮 ref 不稳定，优先用 getByRole 定位
4. PowerShell 中 run-code 的分号会被截断，复杂 JS 用脚本文件
5. Jetson/树莓派方向人才极度稀缺，MIPI CSI 驱动是最有效搜索词

---

## 嵌入式工程师人才市场洞察（2026-04-11 51job 寻访）

### 市场概览
- "Jetson + MIPI-CSI + 机器人"三重叠加人才极度稀缺，51job 符合条件不超过 10 人
- 搜索词效果：Jetson Orin（117结果，信噪比高）> MIPI CSI 摄像头（1000，信噪比中）> 嵌入式 Linux 驱动（5000+，信噪比极差）

### 强推 4 人（51job）
1. 方先生 — 米文动力/Jetson驱动
2. 林先生 — Jetson Orin NX+L4T
3. 蔡先生 — AI研究院/Jetson+ROS+视觉伺服
4. 黄先生 — MIPI-CSI摄像头采集驱动

### 薪资参考
- 应届：1.0-1.8万/月
- 3-5年：1.5-2.5万
- 5-8年：2.5-3.5万
- 8年以上：3.0-4.5万

---

## VLA/具身智能人才市场洞察

### 市场概览
- 人才极度稀缺，猎聘全站真正 VLA 方向不超过 10 人，51job 不超过 20 人
- 大量是应届/实习，有工业落地经验的极少数

### 51job 核心发现（2026-04-11 深度寻访）
- **P0**：夏先生（美团/VLA+RT-1+VLM）、肖先生（比亚迪/慕尼黑工大+VLA微调）
- **P0→升级**：杨先生（亿嘉和/人形Loco+Manip+VLA+XR全栈/积极求职）
- **P1**：g先生（中电科+交大博后/双臂抓取+灵巧手）、丁先生①（哈工大应届/唯一VLA后训练+C++部署/已离职）
- **P1**：谭先生（千觉/VLA+元RL）、武先生（东华应届/ATEC亚军）、徐先生（重大23岁/ACT+DP+VLA）

### 主要雇主
- 美团、比亚迪、亿嘉和、中电科21所、千觉机器人、智科特

### 薪资参考
- 应届硕士：1.8-2.3万/月
- 1-3年：2.3-3.5万
- 3-5年：3-4.2万
- 博士：2.5-5万
- 有工业落地经验仅 4 人
