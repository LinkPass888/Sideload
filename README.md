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

GitHub Actions 每 10 分钟跑一次；检测到新版本会提交 `source.json`，
Pages 直接从 `main` 分支根目录发布。

## 让更新立刻生效

定时轮询有延迟，而且 GitHub 在高负载时会推迟 scheduled run。要几乎实时，
让上游的发布 workflow 在打完 Release 后直接通知本仓库。

1. 建一个 fine-grained PAT，权限只需 **Contents: Read and write**，仓库选本仓库。
2. 在本仓库 Settings → Secrets 里存成 `DISPATCH_TOKEN`。
3. 在上游仓库（Kazumi-Swift / CarCanvas / AirBeep）发布成功的 job 末尾加一步：

```yaml
      - name: Notify sideload source
        if: success()
        run: |
          curl -fsSL -X POST \
            -H "Authorization: Bearer ${{ secrets.SIDELOAD_TOKEN }}" \
            -H "Accept: application/vnd.github+json" \
            https://api.github.com/repos/LinkPass888/Sideload/dispatches \
            -d '{"event_type":"upstream-release"}'
```

上游的 `SIDELOAD_TOKEN` 就是第 1 步那个 PAT。加完这一步，发版后约 1 分钟源就会更新。

## 本地预览

```bash
python -m http.server 8080
```