# memory_manager

**One capability bit from GBT V8, packaged standalone.** No. 164 of 341,
`mind` domain (Mind).

<p>
<img alt="python" src="https://img.shields.io/badge/python-3.9%2B-blue">
<img alt="license" src="https://img.shields.io/badge/license-MIT-green">
<img alt="deps" src="https://img.shields.io/badge/runtime%20dependencies-0-brightgreen">
</p>

> 记忆管理 + 万能精准秒搜（主人令 2026-09-27）：SQLite **FTS5 全文索引**（实测中文可检索）·六类源（记忆/文档/技能/状态/**能力位**/代理）⇒ find 秒级返回「路径+类型+高亮片段」，并**同时端出匹配的能力位**（"想到匹配面出现的能力"）；index 增量（mtime+sha 未变即跳过；**不许整表清空** —— 实测因此丢过索引）；store 存记忆过 filesystem_write 门。SQL 纪律：全部内联字面量 + `?` 占位符，不用变量拼 SQL（安全钩子实测拦过两次）。op=index/find/store/stats

That description is copied verbatim from the registry. `tests/test_spec.py` pins it,
so documentation drift fails the build instead of shipping quietly.

## What this package actually is

It is the **contract** for one capability: its name, its parameters, the values it
accepts, and whether it can change state that cannot be restored. It is not the
implementation — that lives in the runtime this bit was extracted from.

So `run()` goes through a narrow `Backend` protocol, and the default backend is a
stub that **says so**:

```
status="validated"   backend="stub"   data={"executed": false}
```

Never `status="ok"`. A stub that reports success is worse than no stub, because the
caller stops looking for the real thing.

## Install

```bash
git clone https://github.com/gbtweilai-star/gbt-memory-manager.git
cd gbt-memory-manager
python -m pip install -e ".[dev]"
```

## Use

```bash
python -m gbt_memory_manager info                  # capability card
python -m gbt_memory_manager spec --json           # machine-readable contract
python -m gbt_memory_manager check --payload '{}'
python -m gbt_memory_manager run   --payload '{}'
```

Real `info` output from this machine:

```text
memory_manager  (mind)  v1.0.0
----------------------------------------
记忆管理 + 万能精准秒搜（主人令 2026-09-27）：SQLite **FTS5 全文索引**（实测中文可检索）·六类源（记忆/文档/技能/状态/**能力位**/代理）⇒ find 秒级返回「路径+类型+高亮片段」，并**同时端出匹配的能力位**（"想到匹配面出现的能力"）；index 增量（mtime+sha 未变即跳过；**不许整表清空** —— 实测因此丢过索引）；store 存记忆过 filesystem_write 门。SQL 纪律：全部内联字面量 + `?` 占位符，不用变量拼 SQL（安全钩子实测拦过两次）。op=index/find/store/stats

gated      : False
external   : False
multi-op   : True
params     : (none)
values     : index, find, store, stats
```

Wire a real implementation:

```python
from gbt_memory_manager import run

class MyBackend:
    name = "my-backend"
    def invoke(self, cap, payload):
        return {"cap": cap, "handled": sorted(payload)}

result = run({}, backend=MyBackend())
assert result.status == "ok"
```

## Parameters

| name | required | notes |
|---|---|---|
| — | no | the registry declares no required parameters |

Registry hint (`params_hint`): (empty)

**注册表登记的取值 / 子动作（4 个）**：`index`、`find`、`store`、`stats`

Types are deliberately absent: the registry does not declare them, and an invented
type is worse than no type, because callers believe it.

## Boundaries

- Not gated: the registry marks this bit `safe: true`.
- No external service dependency declared.
- Multi-op (`multi_op: true`): several sub-actions share one name — read `op` before calling.
- No implementation ships here. Bring a backend; the protocol is one method.

## What this is not

Not a wrapper around a hosted API. Not a model. Not a promise that calling it does
anything on its own. It is the typed seam plus the validation, which is the part that
stays correct even when the implementation changes.

## Tests

```bash
python -m pytest
```

```text
20 passed, 3 skipped in 0.13s
```

## Provenance

| field | value |
|---|---|
| capability | `memory_manager` |
| domain | `mind` |
| handler | `memory_manager` |
| safe | `True` (gated = `False`) |
| external | `False` |
| atlas page | [mind/memory_manager](https://github.com/gbtweilai-star/gbt-caps/blob/main/caps/mind/memory_manager.md) |
| full index | [https://github.com/gbtweilai-star/gbt-caps](https://github.com/gbtweilai-star/gbt-caps) |

The parent README says 348 capabilities; an AST pass over the registry enumerates
`341 + 0 unimplemented + 0 disabled = 341`. The extra 7 are historical duplicates.
Numbering here follows 341 and matches the `gbt-caps` index one-to-one.

## License

MIT.
