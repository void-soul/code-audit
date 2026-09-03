# 进度文档（PROGRESS.md）— 防上下文丢失协议

审计是跨多阶段的长任务，上下文随时可能被压缩或会话中断。
PROGRESS.md 是进度的**唯一持久记录**：恢复现场全靠它，不依赖对话历史。

## 核心规则（强制）

1. **开工即建**：进入 Phase 0 之前，先做恢复检查（见下）；全新审计立即创建 PROGRESS.md。
2. **随做随记**：每完成一个最小工作单元——一个子项目定位、一个模块分析、一个审计 pass、一份文档——**立刻**更新 PROGRESS.md 再进入下一项。**禁止"最后统一补记"。**
3. **先查再动**：每次（重新）开始工作前，先读 PROGRESS.md；已标 `DONE` 的一律跳过，从第一个 `PENDING`/`IN_PROGRESS` 继续。
4. **自足可恢复**：PROGRESS.md 必须包含恢复现场所需的一切（定位结论、范围、排除目录、架构原则、已生成文档、下一步动作）。

## 位置

```
docs/audit/AUDIT-<YYYYMMDD>-<scope>/PROGRESS.md
```

与其他审计文档同目录。Phase 0 完成定位后即可确定目录并创建。

## 恢复检查（每次会话开始时执行）

1. 检查 `docs/audit/` 下是否已有本次项目的 `PROGRESS.md`
2. **不存在** → 全新审计：创建 PROGRESS.md，从 Phase 0 开始
3. **已存在** → 读取并恢复：
   - 定位/范围/排除目录/架构原则（不重新询问用户，除非记录标明未确认）
   - 各阶段与各 pass 的状态 → **跳过全部 `DONE` 项**
   - 从「当前步骤」与「下一步动作」继续
4. 冲突处理：磁盘上文档已存在但进度未记录 → 以磁盘为准补记，不重新生成

## 模板

```markdown
# 审计进度 — <项目名>

| 字段 | 值 |
|------|-----|
| 开始日期 | YYYY-MM-DD |
| 输出目录 | docs/audit/AUDIT-<YYYYMMDD>-<scope>/ |
| 当前步骤 | <如：Phase 2 / D4 Data & Storage> |
| 下一步动作 | <一句话，如：加载 d4-data.md 审计 service 子项目> |

## 定位与范围（Phase 0 结论）
- **项目定位**: 在线 / 本地 / 混合（逐子项目标注）
- **标注项过滤**: 查【在线】跳【本地】 / 查【本地】跳【在线】 / 混合逐子项目
- **审计范围**: <目录/子项目清单>
- **排除目录**: <目录 + 理由>
- **架构原则**: <用户确认的原则>
- **用户确认状态**: 已确认 / 待确认（未确认时恢复后需先补确认）

## 阶段进度

| 项 | 状态 | 备注 |
|----|------|------|
| Phase 0 定位与范围确认 | PENDING / DONE | 用户确认于 YYYY-MM-DD |
| Phase 1 子项目定位 | PENDING / IN_PROGRESS / DONE | 已完成: xxx, yyy；待: zzz |
| Phase 1 模块定位 | ... | |
| Phase 2 / D1 Security | ... | N issues（【在线】项 N/A 条） |
| Phase 2 / D2 Concurrency | ... | |
| Phase 2 / D3 ~ D8 | ... | |
| Phase 3 / OVERVIEW.md | ... | |
| Phase 3 / STRUCTURE.md | ... | |
| Phase 3 / SUBPROJECT-*.md | ... | 逐个子项目勾选 |
| Phase 4 审计总文档 | ... | |

状态值：`PENDING` / `IN_PROGRESS`（进行中，备注写明断点）/ `DONE`

## 发现问题计数（随做随记，最终汇总进审计总文档）

| Pass | CRITICAL | HIGH | MEDIUM | LOW |
|------|----------|------|--------|-----|
| D1 Security | | | | |
| ... | | | | |

## 已生成文档

- [ ] OVERVIEW.md
- [ ] STRUCTURE.md
- [ ] SUBPROJECT-<name>.md（逐个列出）
- [ ] AUDIT-<YYYYMMDD>-<scope>.md
```

## Completion criterion

- 每个最小工作单元完成后，PROGRESS.md 均有对应更新（状态 + 备注/计数）
- 任意时刻中断后，仅凭 PROGRESS.md + 磁盘文档即可无缝继续，无重复劳动
