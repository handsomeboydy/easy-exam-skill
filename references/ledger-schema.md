# 进度账本规范

在新建、读取或更新进度时读取本文件。新项目推荐使用 `.easy-exam/state.json`；已有项目可以继续使用配置声明的权威旧账本。

## 结构化账本

建议的顶层结构：

```json
{
  "version": 1,
  "exam_id": "example-exam-2027",
  "revision": 1,
  "updated_at": "2027-08-01T20:30:00+08:00",
  "current_stage": "foundation",
  "last_planned_date": null,
  "last_completed_date": null,
  "knowledge": {},
  "questions": {},
  "attempts": [],
  "correction_queue": [],
  "verification_queue": [],
  "rolling_scores": {},
  "sealed_sets": {},
  "artifacts": [],
  "audit_log": []
}
```

### 知识点

每个知识点至少保存：稳定 ID、科目、主题、当前状态、来源、最近证据和下次验证。状态只能由证据驱动：

```json
{
  "subject-a::chapter-1::topic-x": {
    "subject_id": "subject-a",
    "status": "mastery_candidate",
    "sources": ["教材A，第1章"],
    "last_evidence_id": "attempt-20270801-b01",
    "next_verification": {
      "window": "D+1",
      "not_before": "2027-08-02",
      "requires_novel_item": true
    }
  }
}
```

推荐状态为 `unseen`、`learned`、`corrected`、`mastery_candidate`、`mastered`、`stable`。如果考试项目需要更多状态，应定义证据门槛，不能只增加名称。

### 题目与作答

题目保存稳定 ID、科目、题型、来源、覆盖知识点、有效性、A/B 角色和是否封存。同一道题不得在同一学习周期同时承担 A 与 B。

首次作答必须独立成不可变记录：

```json
{
  "attempt_id": "attempt-20270801-b01",
  "question_id": "subject-a-B-20270801-01",
  "role": "B",
  "attempt_kind": "first",
  "submitted_at": "2027-08-01T19:10:00+08:00",
  "conditions": {"closed_book": true, "timed": false, "novel": true},
  "raw_answer": "C",
  "score": 1,
  "max_score": 1,
  "grading_status": "final",
  "error_types": []
}
```

订正和重做创建新的 `attempt_id`，通过 `supersedes_for_correction` 或 `related_first_attempt_id` 指向首次记录；不得修改首次记录的答案、分数和条件。

## 写回事务顺序

1. 读取当前 `revision` 并验证关键不变量。
2. 持久化题卷版本、用户原始提交和逐题评分证据。
3. 追加订正或验证记录。
4. 根据证据计算知识状态、队列和滚动指标。
5. 将 `revision` 加一并追加审计日志。

任一步失败时，不得写入“今日已完成”。再次执行前先检查是否已有同一提交或同一题卷版本，避免重复追加。

## 必须保留的审计事实

- A 类表现与 B 类首次表现分开；
- 各科首次得分、总首次得分及题型分项；
- 首次错误、错误类型、纠正结果和后续验证；
- 掌握与稳定状态所依据的陌生题证据；
- 已使用题目及角色，封存题解封状态；
- 待核验的旧题、法规或评分项；
- 输出文件与题卷版本；
- 任何状态降级、冲突修复或人工覆盖。

## 旧账本兼容模式

当配置声明 `state.mode: legacy_markdown` 时：

- 完整读取配置指定的权威规则/账本文档以及目标账本章节；
- 历史口径保持原样，不追溯重算，除非用户明确要求迁移；
- 只在原账本中更新，不创建 `.easy-exam/state.json`；
- 新字段无法无损写入时，先在原账本中增加清晰字段定义，再写数据；
- 文件内规则与配置摘要冲突时，以标为权威的原文档为准并报告配置漂移；
- 真正迁移时先制作字段映射和不可变首次成绩清单，验证成功后再切换唯一状态源。
