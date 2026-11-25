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
.github/workflows/sync.yml  每 12 小时同步并提交
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

## 本地预览

```bash
python -m http.server 8080
```