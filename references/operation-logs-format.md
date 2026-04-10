# 执行日志格式规范

_定义：operation_logs.jsonl — 记录每次寻访的完整执行轨迹，支撑月度审计和学习进化。_

---

## 存储位置

```
references/operation_logs.jsonl
```

每行一条 JSON 记录（JSONL 格式），顺序追加，不覆盖。

---

## 行格式

```json
{
  "session_id": "2026-04-10-001",
  "platform": "liepin",
  "position": "具身智能研究员",
  "jd_summary": "VLA/Diffusion/具身智能算法，3-10年，硕士",
  "keywords": {
    "R1": "VLA 具身智能 机器人",
    "R2": "OpenVLA Diffusion Policy ACT",
    "R3": "具身智能 算法研究员"
  },
  "chosen_plan": "方案 B",
  "timestamp_start": "2026-04-10T22:15:00",
  "timestamp_end": "2026-04-10T22:48:00",
  "duration_min": 33,
  "search_result_count": 71,
  "initial_pass_count": 12,
  "deep_read_count": 7,
  "strong_recommend": 4,
  "contactable": 3,
  "pending": 5,
  "selector_failures": [],
  "new_discoveries": ["直接 URL 打开详情页触发付费墙"],
  "report_file": "具身智能研究员-寻访报告-v3.md",
  "excel_file": "candidates_liepin_20260410_001.xlsx",
  "user_confirmed": true
}
```

---

## 字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `session_id` | string | 日期 + 当日序号，如 `2026-04-10-001` |
| `platform` | string | 执行平台：`liepin` / `51job` / `zhipin` |
| `position` | string | 岗位名称 |
| `jd_summary` | string | JD 摘要（不超过 100 字） |
| `keywords` | object | 各轮搜索词，key 为 R1/R2/R3 |
| `chosen_plan` | string | 用户选择的搜索方案描述 |
| `timestamp_start` | ISO 8601 | 开始时间 |
| `timestamp_end` | ISO 8601 | 结束时间 |
| `duration_min` | int | 执行耗时（分钟） |
| `search_result_count` | int | 总搜索结果数（去重前） |
| `initial_pass_count` | int | 初筛通过总数（强推+可联系+待定） |
| `deep_read_count` | int | 深读人数 |
| `strong_recommend` | int | 强推数 |
| `contactable` | int | 可联系数 |
| `pending` | int | 待定数 |
| `selector_failures` | array | 选择器失效记录，如 `[{selector, error, timestamp}]` |
| `new_discoveries` | array | 本次新发现的坑或页面行为变化 |
| `report_file` | string | Markdown 报告文件名 |
| `excel_file` | string | Excel 汇总表文件名 |
| `user_confirmed` | bool | 用户是否完成了 Phase 1 确认 |

---

## 读取方式（按需）

- **月度审计**：grep 同月 session_id，汇总耗时、结果数、成功率
- **关键词推荐**：按 position 读取历史 keywords_used，推荐高频高效搜索词
- **选择器健康度**：统计 selector_failures 出现频率，触发阈值时告警
- **平台对比**：按 platform 分组，对比 avg duration / avg success_rate

---

## session_id 生成规则

每次 Phase 5 执行时：
1. 读取 operation_logs.jsonl 末行，取 session_id 的日期部分
2. 若日期 = 今天，序号 +1；否则序号重置为 001
3. 格式：`{YYYY-MM-DD}-{3位序号}`，如 `2026-04-11-003`
