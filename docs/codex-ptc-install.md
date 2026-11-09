# codex-ptc preset：安装与验证

`codex-ptc` = 上游 `ptc` preset 的完整副本，仅把隔离 compaction 组换成 `dsh-compress-codex`。
**PTC 工具呈现（`run_code` + 生成的 SDK）完整保留**，`preset-ptc` 不被覆盖。

## 生成

```sh
node scripts/sync-preset-codex.mjs \
  --base ptc \
  --preset-id preset-codex-ptc \
  --config-id codex-ptc \
  --order 5 \
  --name "Codex (PTC)" \
  --description "PTC 工具呈现保持不变，压缩改用 dsh-compress-codex。" \
  --out /tmp/codex-ptc.yml
```

## 安装

新 profile 的 `cordis.patch.yml` 可能是空数组占位符 `[]`，直接追加会产出非法 YAML（dsh 解析报错）。
只有占位符时整文件替换，已有条目才追加：

```sh
P=~/.dsh/profiles/web/cordis.patch.yml
if grep -qE '^\[\]$' "$P"; then
  cat /tmp/codex-ptc.yml > "$P"
else
  printf '\n' >> "$P"; cat /tmp/codex-ptc.yml >> "$P"
fi
dsh --profile web --dump-config >/dev/null && echo "PARSE OK"
```

## 验证

组合级语义等价：与上游 `ptc.patch.yml` 的 diff 只有 5 处语义行（`preset-ptc` →
`preset-codex-ptc`、`config.id`/`order`、compaction 两行）+ 7 行生成头，其余行逐字相同。
所以 PTC 侧的任何上游行都被原样复制，生成器不解释它们的语义。

> **上游变更（`dsh 0.2.2-rc.1`）**：ptc preset 里原来的
> `tool-presentation`（`@deepseek-ai/dsh-agent-tool-presentation`，`config.mode: ptc`）
> 已被 `tool-executor-ptc`（`@deepseek-ai/dsh-tool-executor-ptc`）取代，新行没有
> `mode:` 字段。PTC 呈现语义不变，变化的是行名与模块名；生成器逐行复制，不关心这个差异，
> 但下面按行统计的判据要按新名字数。

| 项 | 期望 |
|---|---|
| `tool-executor-ptc`（`0.2.1-rc.2` 时是 `tool-presentation` + `mode: ptc`） | 1 |
| `disabled: true` | 4 |
| `compaction-codex` | 1 |
| `compaction-basic` | 0 |

## 回滚

移除 profile patch 里该 insert 块，或从 `~/.dsh/.manual-backup-before-codex-ptc-*` 恢复。
