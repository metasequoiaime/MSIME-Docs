# 2026-09-05 版本方案回退记录

历史说明迁自[组织约定固定版本](https://github.com/metasequoiaime/.github/blob/012b5b9421102adffff4a25e29a8ccdd1411390b/AGENTS.md#版本号)。当前版本政策与变更流程以[组织约定](https://github.com/metasequoiaime/.github/blob/main/AGENTS.md#版本号)为准；本页不提出新版本方案。

这条是已经付出过代价的结论，不是风格偏好。2026-09-05 曾把三端统一切到 `2026.9.0`，当天回退：
release-please 没有 CalVer 策略，只能手工重置序列并钉死 `always-bump-patch`，而它据以生成
changelog 的序列一旦与 tag 历史脱节，就会把早已发布的功能重写成一整条新版本记录（Linux 64 行、
Apple 158 行）。更不可逆的是版本号本身——`v2026.9.1` 发布约二十分钟即删，但 dpkg 和 rpm 认为
`2026.9.1 > 0.7.0`，那个窗口里装过的用户不引入永久 epoch 前缀就收不到升级。

