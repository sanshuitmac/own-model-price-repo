# 手工价格覆盖维护记录

最后更新：2026-09-05

## 仓库与运行方式

本仓库是 `sanshuitmac/own-model-price-repo`，用于给自己的 Sub2API 实例提供模型计价文件。当前采用**静态文件、手工维护**方式，不会定时同步 LiteLLM 上游：

- 价格文件：<https://raw.githubusercontent.com/sanshuitmac/own-model-price-repo/refs/heads/main/model_prices_and_context_window.json>
- 校验文件：<https://raw.githubusercontent.com/sanshuitmac/own-model-price-repo/refs/heads/main/model_prices_and_context_window.sha256>
- 自动同步工作流已在提交 [`056249f`](https://github.com/sanshuitmac/own-model-price-repo/commit/056249f) 中删除。

不要随意恢复 GitHub Actions，也不要直接点击 GitHub 的 `Sync fork` 覆盖当前分支。仓库里的 `rebuild.sh`、`scripts/sync_prices.py` 和 `config.json` 是上游遗留工具；其中 `codex-auto-review -> gpt-5.5` 别名及 `gpt-5.6-sol` 自定义项仍是旧规则，直接运行会覆盖手工价格。

## 当前手工覆盖

价格单位均为美元/百万 Token；JSON 内实际保存的是美元/单 Token，因此表中价格需要除以 `1,000,000` 后写入 JSON。

| 模型 | 标准输入 | 缓存读取 | 缓存写入 | 标准输出 | 说明 |
|---|---:|---:|---:|---:|---|
| `codex-auto-review` | $0.20 | $0.02 | 未单独计费 | $1.20 | 按 Sub2API v0.1.170 / PR #5145 的定价 |
| `gpt-5.6-luna` | $1.00 | $0.10 | 免费 | $6.00 | 保留 OpenAI 降价前的价格 |
| `gpt-5.6-sol` | $4.00 | $0.50 | $5.00 | $25.00 | 2026-08-25 自定义缓存读取及输出价格 |
| `gpt-6-astra` | $10.00 | $1.00 | $12.50 | $50.00 | 2026-09-05 手工新增；长上下文为输入 $20、缓存读取 $2、缓存写入 $25、输出 $75 |

前两项覆盖记录在提交 [`52baad6`](https://github.com/sanshuitmac/own-model-price-repo/commit/52baad6)，其中 `codex-auto-review` 的依据是 [Sub2API PR #5145](https://github.com/Wei-Shaw/sub2api/pull/5145)。

## 2026-08-25：GPT-5.6 Sol 自定义调整

在 2026-08-24 的 OpenAI 促销价格基础上，仅自定义调整缓存读取和输出价格；输入及缓存写入价格不变。单位为美元/百万 Token：

| 服务层级/上下文 | 输入 | 缓存读取 | 缓存写入 | 输出 |
|---|---:|---:|---:|---:|
| Standard，输入不超过 272K | $4.00 | $0.50 | $5.00 | $25.00 |
| Standard，输入超过 272K | $8.00 | $1.00 | $10.00 | $37.50 |
| Flex，输入不超过 272K | $2.00 | $0.25 | $2.50 | $12.50 |
| Flex，输入超过 272K | $4.00 | $0.50 | $5.00 | $18.75 |
| Batch，输入不超过 272K | $2.00 | $0.25 | $2.50 | $12.50 |
| Batch，输入超过 272K | $4.00 | $0.50 | $5.00 | $18.75 |
| Priority/Fast，输入不超过 272K | $8.00 | $1.00 | $10.00 | $50.00 |
| Priority/Fast，输入超过 272K | $16.00 | $2.00 | $20.00 | $75.00 |

该价格是本实例的手工覆盖，并非 OpenAI 当前官方价。OpenAI Docs 在本次调整时显示的基础价格仍为 `$4 / $0.40 / $5 / $20`。

本次调整后的价格文件 SHA-256：

```text
2a6a04411f68e2b70894cc8ffd622215a7bde72cad7f34b6dc8b8c652ff96be5
```

## 2026-08-24：GPT-5.6 Sol 降价

官方依据：

- [OpenAI GPT-5.6 Sol 模型页](https://developers.openai.com/api/docs/models/gpt-5.6-sol)
- [OpenAI API Pricing](https://developers.openai.com/api/docs/pricing)

OpenAI 官方说明该促销价至少持续到 **2026-11-21**。这里的“至少”不是确定的结束时间；到该日期前后应重新核对官方页面，不能直接假定恢复旧价。

本次写入仓库的价格如下，单位为美元/百万 Token：

| 服务层级/上下文 | 输入 | 缓存读取 | 缓存写入 | 输出 |
|---|---:|---:|---:|---:|
| Standard，输入不超过 272K | $4.00 | $0.40 | $5.00 | $20.00 |
| Standard，输入超过 272K | $8.00 | $0.80 | $10.00 | $30.00 |
| Flex，输入不超过 272K | $2.00 | $0.20 | $2.50 | $10.00 |
| Flex，输入超过 272K | $4.00 | $0.40 | $5.00 | $15.00 |
| Batch，输入不超过 272K | $2.00 | $0.20 | $2.50 | $10.00 |
| Batch，输入超过 272K | $4.00 | $0.40 | $5.00 | $15.00 |
| Priority/Fast，输入不超过 272K | $8.00 | $0.80 | $10.00 | $40.00 |
| Priority/Fast，输入超过 272K | $16.00 | $1.60 | $20.00 | $60.00 |

官方规则要点：超过 272K 输入 Token 时，整次请求的输入类 Token 按 2 倍、输出按 1.5 倍计价；缓存写入为对应未缓存输入价格的 1.25 倍。

价格修改提交：[`0b04acb`](https://github.com/sanshuitmac/own-model-price-repo/commit/0b04acb)

该提交修改前的基础价格是 `$5 / $0.50 / $6.25 / $30`，修改后是 `$4 / $0.40 / $5 / $20`。该次价格文件 SHA-256 为：

```text
27ab3fe77561e2fe963f9c6b2fbefcf65822641465471f2f421c31e69a5ed653
```

## 下次手工修改步骤

1. 先从模型厂商官方价格页确认模型名、计价单位、服务层级、长上下文倍率及生效时间。
2. 只修改 `model_prices_and_context_window.json` 中目标模型的字段。每百万 Token 价格除以 `1,000,000`，例如 `$4/百万` 写成 `4e-06`。
3. 验证 JSON 并重新生成哈希：

   ```bash
   jq empty model_prices_and_context_window.json
   sha256sum model_prices_and_context_window.json \
     | awk '{print $1}' > model_prices_and_context_window.sha256
   ```

4. 确认哈希一致：

   ```bash
   test "$(sha256sum model_prices_and_context_window.json | awk '{print $1}')" \
     = "$(tr -d '\n\r' < model_prices_and_context_window.sha256)"
   ```

5. 用 `git diff` 确认只改了目标模型和 SHA-256 文件，再提交并推送到 `main`。
6. 等待 Sub2API 按远端 SHA-256 检测并加载新文件，然后核对容器内文件哈希和前端新产生的用量记录。

## Sub2API 注意事项

- 本次推送后，本实例已自动拉取新价格，容器保持健康，未重启服务。
- 本实例缓存文件是 `/root/sub2api/deploy/data/model_pricing.json` 和 `/root/sub2api/deploy/data/model_pricing.sha256`。
- 价格更新只影响加载新价格后产生的计价；历史使用记录不会追溯重算。
- 若前端价格没有变化，依次检查远端 JSON、远端 SHA-256、本地两个缓存文件的哈希是否一致，再查看服务日志。不要先盲目重启或重新构建镜像。
