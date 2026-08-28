# 考试配置规范

在初始化、迁移或修改考试配置时读取本文件。推荐路径为 `.easy-exam/profile.yaml`。配置只保存相对稳定的事实与策略参数，不保存每天变化的成绩和队列。

## 最小配置

```yaml
version: 1
exam:
  id: example-exam-2027
  name: 示例资格考试
  date: 2027-10-24
  timezone: Asia/Shanghai
  all_subjects_must_pass: true

subjects:
  - id: subject-a
    name: 科目 A
    score_scale: 100
    pass_score: 60

targets:
  training_score: 75
  daily_blind_overall_rate: 0.75
  daily_blind_subject_floor: 0.60
  require_correction_queue_empty: true

time_budget:
  weekday_hours: [3, 4]
  weekend_hours: 6

state:
  mode: structured
  path: .easy-exam/state.json

outputs:
  root: 每日学习
```

## 可选策略字段

按考试实际需要增加，不要为了填满模板而虚构：

```yaml
stages:
  - id: foundation
    start: 2027-08-01
    end: 2027-09-10
    objective: 建立高权重知识框架

coverage:
  daily_subject_policy: all
  allocation: weighted_score_equivalent
  weekday_equivalent_per_subject: 2
  weekend_equivalent_per_subject: 3

assessment:
  teaching_label: A
  blind_label: B
  physical_isolation: true
  first_attempt_immutable: true
  question_types:
    - single_choice
    - multiple_choice
    - case_response
  scoring_notes:
    multiple_choice: 按本考试官方规则
    case_response: 按得分点给分

verification:
  mastery_window: D+1
  stability_window: D+7
  conditional_windows: [D+3]
  aggregate_windows: [D+14, D+30]

sealed_sets:
  release_stage: final_mock
  sets: [2026, 2027]

sources:
  - subject_id: subject-a
    paths: [科目A]
  
validity:
  effective_on: 2027-10-24
  authoritative_source_order:
    - 现行官方规则
    - 官方教材
    - 可靠题库
  high_risk_topics: [法律责任, 金额, 时间, 主体, 标准]

retention:
  enabled: true
  adapter_skill: example-flashcards
  trigger: after_first_attempt_graded

legacy:
  rules_path: 复习元规则.md
  ledger_heading: 八、进度账本
  authoritative: true
```

## 字段语义

- `exam.date` 是正式考试日；阶段日期不得越过它。
- `pass_score` 是正式合格线，`training_score` 是安全边际目标，两者不能混写。
- `daily_blind_*` 是日常训练完成门槛，不等于正式考试预测。
- `coverage.daily_subject_policy` 可取 `all`、`weighted_rotation` 或配置明确描述的策略；不得把某个考试的“四科每日出现”升级为通用硬规则。
- `sealed_sets` 只保存集合标识和解封条件。封存阶段不得检查其内容是否匹配配置。
- `validity.effective_on` 决定法规、标准和官方政策的核验时点。
- `retention` 是可选适配器。缺失时不应为了完整性自动安装或调用任何闪卡工具。
- `state.mode` 推荐 `structured`；迁移已有项目时可用 `legacy_markdown`。只要 `legacy.authoritative` 为真，原文档就是唯一账本，不得同步维护第二份状态。

## 初始化检查

至少确认考试日期、时区、科目、正式合格规则、训练目标、可用时间、题型计分和状态源。之后检查：

- 每个阶段日期连续且不重叠；
- 训练目标不低于正式合格线；
- 科目 ID 稳定且唯一；
- 封存集合不被普通资料路径意外包含；
- 输出目录与状态文件不是同一文件；
- 旧项目的历史成绩在迁移前保持只读，除非用户明确要求转换。
