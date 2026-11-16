# dsh-plugin-publish

给 DeepSeek Harness（DSH）插件仓库用的**发布模板**：推 `v*` 标签就自动发到 npm，不需要 token、不会过期。

## 它解决什么

手工发版每次都要 npm 凭据，而「Bypass 2FA」token 从 2027 年 1 月起就不能直接发布了。这个模板用 **npm Trusted Publishing（OIDC）**：GitHub Actions 拿运行时签发的身份去发布，凭据不存在，也就不会过期。

## 一个包的一次性步骤

1. **让包先存在**：新包必须先在 npm 上发布一次（手工 `npm publish` 或网页上传）。Trusted Publisher 的设置在「已存在的包」页面里。
2. **登记 Trusted Publisher**：npm → 包页面 → Settings → Trusted Publisher → Add trusted publisher：
   - Publisher：GitHub Actions
   - Organization or user：你的 GitHub 用户名
   - Repository：插件仓库名
   - Workflow filename：`publish.yml`（必须完全一致）
   - Environment name：留空
   - Allowed actions：勾 **Allow npm publish**（要管理 tag 再勾 dist-tag）
3. **复制模板**：把 [`templates/publish.yml`](templates/publish.yml) 复制到你仓库的 `.github/workflows/publish.yml`。
4. **验证**：改个版本号 → `git tag vX.Y.Z && git push origin vX.Y.Z` → 看 Actions 跑绿、npm 上出现新版本。

完成之后，这个包以后每次发版都是「推 tag 即发布」。

## 为什么是复制而不是 reusable workflow

npm 的 OIDC 校验看的是 **仓库 + 工作流文件名**。跨仓库调用 `uses: owner/repo/.github/workflows/publish.yml@main` 时，GitHub 签发的身份里工作流路径变成了被调用的那个仓库，和 npm 上登记的 `publish.yml` 对不上，容易被拒。所以模板选择「每个仓库一份同名文件」——重复三四十行，但不会坏。

## 清单

```
[ ] 包已在 npm 存在
[ ] Trusted Publisher 已登记（workflow = publish.yml）
[ ] .github/workflows/publish.yml 已复制
[ ] 首次 tag 发布跑绿
[ ] 发布后确认 npm 有 Provenance 徽章
[ ] 删除旧的 Bypass 2FA token
```

## License

MIT