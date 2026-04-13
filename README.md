# AI Headhunting Skill

自动化人才搜索与筛选工具 — AI 驱动的招聘平台寻访 Skill。

替代招聘人员在平台上手动翻页、逐份看简历的重复劳动，将「理解岗位 → 设计策略 → 搜人 → 筛人 → 读简历 → 出报告」的完整寻访流程自动化。

---

## 核心能力

**岗位理解分析**
不只是从 JD 摘关键词，而是深度理解岗位要解决什么问题、需要什么能力、什么人能匹配。输出岗位定位、能力模型（加权）、理想人才画像、硬性过滤条件、薪资反推分析。

**寻访策略设计**
搜索词从能力模型和人才画像反推，每个搜索词必须说明「目标人群」和「匹配逻辑」。强制配置平台筛选器（岗位类型、薪资范围、经验区间），非技术岗前置排除。

**逐页搜索 + 深读**
在招聘平台执行搜索，逐页提取候选人信息并立即深读本页 P0/P1 人选，避免翻页不可回退的问题。支持三级数据提取降级方案（JS eval → snapshot + get_attribute → 纯 snapshot）。

**量化评分框架**
列表页粗筛三档（P0/P1/P2）+ 详情页精评打分（四维加权模型：技能 40% + 经验 25% + 薪资 20% + 学历 15%），初筛有依据，推荐有评分。

**结构化寻访报告**
输出包含简历编号和详情页链接的完整报告 + Excel 汇总表，拿到即可在平台上搜索定位、下载简历、发起沟通。

**自学习进化**
每次寻访结束后自动提取操作经验（选择器命中率、操作失败记录、搜索词效果），按平台分别写入独立记忆文件，持续进化。

---

## 适用场景

硬科技 / 前沿技术岗位的精准寻访，尤其是候选人稀缺、需要跨领域关键词组合搜索的岗位。

比如 VLA、具身智能、Jetson 嵌入式这类岗位，传统单一关键词搜索容易漏人，多轮分流搜索能显著提升覆盖面。

---

## 寻访流程

| 阶段 | 做什么 | 产出 |
|------|--------|------|
| Phase 1：岗位理解 + 策略设计 | Step 1.1 岗位理解分析（定位、能力模型、人才画像、薪资反推）→ Step 1.2 寻访策略设计（搜索词推导、筛选器配置）→ **强制等待用户确认** | 岗位理解 + 搜索策略 |
| Phase 2：逐页搜索 + 初筛 + 深读 | 执行搜索 → 逐页提取列表数据 → 粗筛三档 → 本页 P0/P1 深读详情页 → 精评打分 → 翻页 | 候选人列表 + 评分 + 深度分析 |
| Phase 3：输出报告 | 生成结构化寻访报告 + Excel 汇总表 | 完整寻访报告（.md + .xlsx） |
| Phase 4：自学习进化 | 提取选择器命中率、操作失败、搜索词效果等经验 | 平台记忆文件增量更新 |

---

## 报告样例

报告中每个人选会输出如下信息卡：

```
### 曾** | P0 强推 | 综合评分 8.2
- 简历编号：966XXXXXXXXXXX56f
- 详情链接：https://lpt.liepin.com/cvview/showresumedetail?resIdEncode=966XXXXXXXXXXX56f
- 当前：清宝引擎 · VLA算法工程师
- 关键匹配：3年VLA落地经验，主导端到端视觉-语言-动作模型
- 风险点：近两份工作平均在职不足1年；期望薪资略超预算
```

---

## 支持平台

| 平台 | 状态 | 说明 |
|------|------|------|
| 猎聘网 | ✅ 已验证 | 完整页面操作地图，含 React 输入技巧、详情页深读、简历编号提取 |
| 前程无忧 | ✅ 已验证 | 完整页面操作地图，含 SPA 导航技巧、新 Tab 详情页、搜索词效果对比 |
| Boss 直聘 | ⏳ 待补充 | 平台操作地图待实战验证后写入 |
| LinkedIn | ⏳ 待补充 | 支持字段搜索语法，待实战验证 |

---

## 安装方式

### 方式一：直接安装

将本 skill 文件夹放到 WorkBuddy 的 skills 目录下：

```
~/.workbuddy/skills/ai-headhunting/
```

### 方式二：Skill Hub 导入

在 WorkBuddy 中导入 zip 包即可。

重启 WorkBuddy 后，在对话中提到「寻访」「搜简历」「找候选人」等关键词即可触发。

---

## 目录结构

```
ai-headhunting/
├── SKILL.md                  # Skill 主文件（流程定义 + 评分框架 + 自学习 Phase 4）
├── README.md                 # 本文件
├── LICENSE                   # MIT License
├── .gitignore                # Git 忽略规则
├── references/
│   ├── platforms.md          # 各平台页面操作地图（选择器、输入技巧、已知坑）
│   ├── age-ranges.md         # 年龄段与工作经验对照表
│   ├── self-learning.md      # 自学习进化方案（架构 + 模块设计 + 实施路径）
│   ├── operation-logs-format.md  # 执行日志格式规范
│   ├── positions-format.md   # 岗位库格式规范
│   ├── operation_logs.jsonl  # 执行日志（自动积累，不入 Git）
│   ├── positions.jsonl       # 岗位库（自动积累，不入 Git）
│   └── memory/               # 平台自学习记忆（自动积累，不入 Git）
│       ├── _TEMPLATE.md      # 新平台记忆文件模板
│       ├── liepin.md         # 猎聘网（自动生成）
│       ├── 51job.md          # 前程无忧（自动生成）
│       └── zhipin.md         # Boss 直聘（自动生成）
└── _skillhub_meta.json       # Skill Hub 元数据（不入 Git）
```

> `memory/`、`operation_logs.jsonl`、`positions.jsonl` 存放个人使用数据（验证次数、搜索词效果、执行记录），属于个人数据，不纳入版本控制，首次使用时自动生成。

---

## 技术细节

### 猎聘网

**React 受控组件输入**：猎聘搜索框是 React 受控组件，不能直接 `fill` 或设 `input.value`，必须用 `nativeInputValueSetter` 触发 React 事件更新。多轮搜索时建议用「导航重置」方案（重新导航到搜索页 URL）。

**简历编号提取**：候选人卡片上的 `data-resumeidencode` 属性，是简历的加密 ID，可用于详情页直链拼接。

**详情页模式**：点击卡片后以 Modal 弹窗形式展示（URL 加 `#preview`），不跳转新页面。**直接用 URL 打开详情页会触发付费墙**，只能通过搜索结果页点击卡片弹窗查看。

**CSS 选择器注意事项**：猎聘的 CSS class 带 hash 后缀（如 `--FcnzW`），发版后可能变化，选择器失效时需重新 snapshot 确认。

**搜索输入重置**：多轮搜索时，React 受控组件无法用 `clear` 清空。推荐方案：导航到 `https://lpt.liepin.com/search` 重置所有状态，重新输入搜索词。如果导航后跳转到 `#login`，退回 Ctrl+A 全选替换方案。

**数据提取降级**：支持三级提取。Level 1（JS eval 批量提取，一次拿整页 20 人）→ Level 2（snapshot + get_attribute 补充 resumeId）→ Level 3（纯 snapshot，resumeId 留空）。

完整的选择器、脚本和已知坑，见 `references/platforms.md`。

### 前程无忧

**SPA 导航**：搜索页必须从工作台左侧导航「人才搜索」进入，直接 URL 打开会得到空白页面。

**非 React 组件**：搜索框可直接 `fill` 输入，比猎聘简单。搜索按钮用 `page.getByRole('button', {name: '搜索'}).click()` 最可靠。

**不支持布尔搜索**：51job 不支持 AND/OR/NOT 语法，空格 = 松散 OR。搜索词应控制在 2-3 个核心词。

**详情页新 Tab**：点击候选人姓名会打开新标签页（非弹窗），需用 `tab-select` / `tab-close` 管理，`Promise.all([waitForEvent('page'), click])` 捕获新 tab。

**CSS Modules 动态命名**：没有统一的候选人卡片 class，`innerText` 提取更可靠。

---

## 安全设计

- **强制门控**：启动前必须完成岗位理解分析 + 寻访策略设计并等待用户确认，禁止自动执行
- **浏览器通道优先级**：MCP browser → headed → headless（headless 不再硬禁，作为降级方案）
- **简历编号脱敏**：报告中姓名脱敏输出，简历编号仅用于平台内部定位
- **速率控制**：连续操作间隔 2-3 秒，避免触发反爬
- **输出路径规范**：使用 `{OUTPUT_DIR}` 占位符，不硬编码绝对路径

---

## 贡献指南

欢迎贡献！你可以：

1. **补充新平台操作地图** — 在 `references/platforms.md` 中添加 Boss 直聘等平台的选择器和操作技巧
2. **更新已有选择器** — 平台发版后选择器变化时，提交 PR 更新
3. **分享搜索词策略** — 不同岗位类型的高效搜索词组合
4. **改进评分框架** — 优化维度权重、一票否决条件

### 贡献流程

1. Fork 本仓库
2. 创建功能分支（`git checkout -b feature/new-platform`）
3. 提交改动（`git commit -m 'Add Boss直聘操作地图'`）
4. 推送分支（`git push origin feature/new-platform`）
5. 创建 Pull Request

---

## 致谢

本 Skill 基于 [WorkBuddy](https://www.codebuddy.cn/) + playwright-cli 能力构建，实战验证于猎聘网企业端和前程无忧企业端。
