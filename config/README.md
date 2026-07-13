# config 维护须知

## model-registry.json（模型清单，唯一编辑源）

本文件是三千模型清单的单一事实源（sdk 侧 D43/D44 裁决），有**存量客户端直接远程消费**，
改动纪律如下：

1. **改内容必须 bump 顶层 `version`（整数递增）。** 消费端按版本单调防回滚：
   同版本改内容会被直接拒收且不再重试；版本回退一律拒收。「同版本 = 同内容」是硬契约。
2. 只在本仓编辑。sdk 的 bundled 种子经 `scripts/sync-model-registry.mjs` 从这里
   单向同步（脚本同样强制版本单调，并自动改写种子版本串）。
3. 改动前把读者面算全：
   - 老三千生产 App：jsdelivr + raw 双源远程拉取，`MIN_REGISTRY_VERSION=36`，
     未知 `validateType` 的 provider 行会被逐行丢弃；
   - 新栈 hub（sanqian-sdk）：触发式远程刷新，**只消费 `models[]`**（provider
     模板走 sdk 发版通道），版本单调 + 条目数骤降过半拒收（防误清库）；
   - sdk bundled 种子与 `/api/registry` 兼容面。
4. 发坏了怎么退：不能改回旧版本号（会被拒收），只能发一个更高版本的修复版。

## skills-registry.json / skill-curated

Skill 目录面，暂无远程直拉消费者，纪律从宽；进 sdk 仍走对应同步脚本。
