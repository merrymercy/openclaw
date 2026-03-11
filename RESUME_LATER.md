# Session Resume: add-sglang-openclaw

## What was done

Added SGLang as a first-class model provider to OpenClaw, mirroring the existing vLLM implementation.

## PR

https://github.com/merrymercy/openclaw/pull/1
Branch: `lianmin/add-sglang`

## Files changed

### New files
- `src/commands/sglang-setup.ts` — `promptAndConfigureSglang()`, default base URL `http://127.0.0.1:30000/v1`
- `src/commands/auth-choice.apply.sglang.ts` — `applyAuthChoiceSglang()` handler
- `docs/providers/sglang.md` — provider documentation

### Modified files
- `src/agents/model-auth-env-vars.ts` — added `sglang: ["SGLANG_API_KEY"]` to `PROVIDER_ENV_API_KEY_CANDIDATES` (**critical fix** — without this the env var was never read and discovery silently failed)
- `src/agents/models-config.providers.discovery.ts` — `discoverSglangModels()`, `buildSglangProvider()`
- `src/agents/models-config.providers.ts` — `resolveSglangImplicitProvider()`, called before vLLM in `resolveImplicitProviders()`
- `src/commands/onboard-types.ts` — `"sglang"` in `AuthChoice` + `AuthChoiceGroupId`
- `src/commands/auth-choice-options.ts` — SGLang group + base option (before vLLM)
- `src/commands/auth-choice.apply.ts` — `applyAuthChoiceSglang` registered before `applyAuthChoiceVllm`
- `src/commands/auth-choice.preferred-provider.ts` — `sglang: "sglang"` entry
- `src/commands/model-picker.ts` — `SGLANG_VALUE` + SGLang picker option + handler
- `src/commands/onboard-non-interactive/local/auth-choice.ts` — `"sglang"` guard block
- `docs/providers/index.md` — SGLang entry before vLLM
- `docs/concepts/model-providers.md` — SGLang section before vLLM
- Test files: `model-picker.test.ts`, `auth-choice-options.test.ts`, `onboard-non-interactive.provider-auth.test.ts`, `models-config.providers.discovery-auth.test.ts`

## Environment setup on this machine

Node.js is **not** in the default PATH. Before running any `node`/`pnpm` commands:

```bash
source ~/.bashrc   # activates fnm (Node 22.22.1)
```

Then from `/root/openclaw`:

```bash
export SGLANG_API_KEY=sglang-local
node scripts/run-node.mjs models list --all | grep sglang
```

SGLang server (Qwen/Qwen3-8B) was running at `http://127.0.0.1:30000/v1` during testing — it may need to be restarted:

```bash
python3 -m sglang.launch_server --model-path Qwen/Qwen3-8B --port 30000 --host 127.0.0.1
```

## Known behaviour

- `models list` (no flags) shows only configured models — SGLang won't appear here unless set via `models set`.
- `models list --all` shows all discovered providers including SGLang when `SGLANG_API_KEY` is set and the server is running.
- `pnpm` is not in PATH by default either — use `source ~/.bashrc` first; pnpm was installed globally via npm.

## Git identity configured

```
user.name  = Lianmin Zheng
user.email = lianmin@sglang.ai
```

## Remaining / possible next steps

- Review and merge PR #1
- Consider making `models list --local` also surface implicit discovered local providers (SGLang, vLLM, Ollama) — currently returns nothing for discovered-but-not-configured local providers. Filed as a potential follow-up.
- Run full test suite (`pnpm test`) once build tooling is confirmed stable on this host.
