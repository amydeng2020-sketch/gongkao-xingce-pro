# Coze 平台运行说明（安装时追加）

本 Skill 脚本 `scripts/writeback.py` 为纯标准库 Python（零网络、零第三方依赖），
Coze 中通过 bash 调用，动作字段与协议（schema_version、event_id、action、
explanation_completed 等）与原平台完全一致，详见同目录 `写回运行协议.md`。

## 调用语法（Coze 实测）

脚本不接受位置参数动作名，统一用 `--cwd` / `--payload`：

```
python3 <skill_dir>/scripts/writeback.py --cwd <学习记录目录> --payload /tmp/payload.json
```

- payload 中的 `"action"` 字段决定动作（wrong_answer / review_result /
  pending_topic / mastery_upgrade / profile_refresh）。
- `--payload -` 表示从 stdin 读 JSON；正式调用建议先把 payload 写到 /tmp 临时文件再传路径，
  避免 shell 转义中文/引号问题。
- `--dry-run` 只校验并列出将要写入的文件，不落盘；正式写回不要默认加它。

## 状态根（state_root）

- 脚本在 `--cwd` 指定目录下解析状态根，标记名为 `review-engine`、`.workbuddy/memory`
  或 `复盘引擎`；Coze 中约定使用项目目录下的 `行测学习记录/` 作为 cwd，
  脚本首次成功写回时会自动在其下创建 `复盘引擎/`（含 `错因卡/`、`弱点档案/`、
  `.writeback/events.jsonl`），无需手工预建标记目录。
- 不向 Skill 安装目录、其他项目或 cwd 之外写入。

## 回执与降级

- 回执 `status`：`written`（确认 files 列表后才可对用户说"已落盘"）、
  `noop`（同一 event_id 已处理，不重复计数）、`blocked`（报告 code/message）。
- 仅当 Python 或脚本不可用时，才读 `复盘引擎说明.md` 降级为手工 Markdown 写回；
  输入校验失败、状态根冲突、答案冲突一律停止，不得绕过门禁。
- 错题本 Excel 导出用平台表格技能承接；题目截图/答题卡照片用内置多模态读图能力识别，
  不清处标记"待核对"。
