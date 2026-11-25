# Sideload

LinkPass 应用的侧载入口页与 AltSource。

- Source: <https://linkpass888.github.io/Sideload/source.json>
- Apps: Kazumi、CarCanvas、AirBeep
- Support: AltStore、SideStore、Feather

## 结构

```
index.html            入口页（纯静态，无构建步骤）
source.json           AltSource 源，由脚本生成
assets/apps/*.png     应用图标
scripts/sync-source.mjs  从上游 Release 拉取最新 IPA
.github/workflows/sync.yml  定时 + 事件触发的同步
```

## 更新源

```bash
GITHUB_TOKEN=… node scripts/sync-source.mjs
```

脚本读取 `source.json` 的 `apps[].bundleIdentifier`，到下表对应仓库取最新的稳定 Release，
写入 `version` / `date` / `downloadURL` / `size` / `sha256` / `localizedDescription`。

| Bundle ID | 仓库 |
| --- | --- |
| `com.linkpass.kazumi` | LinkPass888/Kazumi-Swift |
| `com.carcanvas.wq` | LinkPass888/CarCanvas |
| `com.linkpass888.airbeep` | LinkPass888/AirBeep |

GitHub Actions 每 12 小时跑一次；检测到新版本会提交 `source.json`，
Pages 直接从 `main` 分支根目录发布。

## 更新延迟

| 路径 | 延迟 | 状态 |
| --- | --- | --- |
| Kazumi-Swift / CarCanvas 发布后 dispatch | 约 1 分钟 | 已接入，待配 `SIDELOAD_TOKEN` |
| AirBeep 发布后 dispatch | — | 不可用，见下 |
| 定时轮询 | 最长 12 小时 | 一直生效，是主要兜底 |

同步 workflow 本身监听 `repository_dispatch`（`upstream-release`），
上游发布 job 末尾调用一次即可。那段 yaml 已经加在
`Kazumi-Swift/.github/workflows/release.yaml` 和
`CarCanvas/.github/workflows/build-ipa.yml` 的 release job 末尾。

**AirBeep 无法接入**：`LinkPass888/AirBeep` 是 `frs0n/placard` 的 fork 且已归档（archived），
归档仓库的 workflow 不能修改。所以 AirBeep 只能靠定时轮询，最长 12 小时。
如果以后要接，得先取消归档，或者在本仓库里改用「观察 AirBeep release 时间」的方式主动拉取。

### 待配置

`SIDELOAD_TOKEN` 需同时存在于 Kazumi-Swift 和 CarCanvas 两个仓库的
Settings → Secrets and variables → Actions 里。同一个 fine-grained PAT：

- Repository access：只勾 `LinkPass888/Sideload`
- Permissions：`Contents: Read and write`，其余 No access

三个仓库内置的 `GITHUB_TOKEN` **不能跨仓库使用**，所以必须用 PAT。
这一步是 best effort（`continue-on-error: true`），没配也不影响发布，只是退回轮询。

## 本地预览

```bash
python -m http.server 8080
```