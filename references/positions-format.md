# 岗位库格式规范

_定义：positions.jsonl — 按岗位类型记录历史搜索词效果，为后续推荐提供数据支撑。_

---

## 存储位置

```
references/positions.jsonl
```

每行一条 JSON 记录（JSONL 格式），不存在则创建。

---

## 行格式

```json
{
  "position": "具身智能研究员",
  "platform": "liepin",
  "keywords_used": {
    "R1_VLA具身智能机器人": {
      "keyword": "VLA 具身智能 机器人",
      "times": 2,
      "avg_result_count": 40,
      "avg_match_rate": "高",
      "last_used": "2026-04-10"
    },
    "R2_OpenVLA": {
      "keyword": "OpenVLA Diffusion Policy ACT",
      "times": 1,
      "avg_result_count": 11,
      "avg_match_rate": "中",
      "last_used": "2026-04-10"
    }
  },
  "candidate_profile": {
    "experience_min": 3,
    "experience_max": 10,
    "education": "硕士及以上",
    "key_skills": ["VLA", "Diffusion Policy", "模仿学习"]
  },
  "usage_count": 2,
  "last_used": "2026-04-10"
}
```

---

## 字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `position` | string | 岗位名称（主键，用于去重/查找） |
| `platform` | string | 本次执行使用的平台 |
| `keywords_used` | object | 各轮搜索词统计，key 为搜索轮次+词简称 |
| `keywords_used.*.keyword` | string | 完整搜索词 |
| `keywords_used.*.times` | int | 累计使用次数 |
| `keywords_used.*.avg_result_count` | int | 平均结果数 |
| `keywords_used.*.avg_match_rate` | string | 「高」「中」「低」，由 AI 评估 |
| `keywords_used.*.last_used` | date | 最后使用日期 |
| `candidate_profile` | object | 从 JD 解析的候选人画像 |
| `usage_count` | int | 该岗位被寻访的累计次数 |
| `last_used` | date | 最后寻访日期 |

---

## 写入规则

1. **新岗位**：追加一行新记录
2. **已有岗位**：
   - 追加本次使用的搜索词到 `keywords_used`（key 相同则覆盖，times 累加）
   - `usage_count` +1
   - `last_used` 更新为今天日期
   - `candidate_profile` 以最新 JD 为准覆盖
3. **推荐逻辑（Phase 1）**：
   - 读取 positions.jsonl 查找同名岗位
   - 若找到，按 `avg_match_rate` 排序，优先推荐「高」评分搜索词
   - 若找不到，生成默认 3 个搜索方案（新岗位流程）

---

## 轻量版说明

本版本仅记录搜索词的使用频次和效果评估，**不包含评分算法**。

后续可升级方向：
- 计算每个搜索词的「强推产出率」= 强推数 / 结果数
- 按岗位类型聚类，跨岗位迁移高效搜索词
- 动态调整搜索词权重
