# miaobi-site

妙笔（Miaobi）官网 + 共享规则池 + 许可名单 + 发布下载点。**软件本体闭源**
（源码在私有仓库），本仓库承担四件事：

1. **官网**：`index.html` 单页静态站（GitHub Pages 托管，`main` 分支根目录）
2. **规则池**：`rules/pool.json` —— 客户端合规检测的共享词库/教训池，
   全体用户每 24h 自动拉取（jsdelivr CDN → raw 回退）
3. **许可名单**：`licenses.json` —— 激活码吊销通道（只存哈希，不泄码）
4. **发布下载**：Releases 挂安装包/压缩包，客户端更新检查指向本仓库
   `releases/latest`

## 发码（人工签发）

发行机上运行（主密钥 `tools/keys/` 在私有仓库，gitignore）：

```
python tools/keygen.py code <用户机器码> [天数]
```

输出三样：机器码、有效期、`MB.` 激活码（发给用户），以及一条**名单条目**。
把条目追加进本仓库 `licenses.json` 的 `licenses` 数组并推送——即完成发码登记。

吊销：把对应条目的 `revoked` 改为 `true` 推送，用户端 24h 内生效
（断网用户在下次联网时生效；签名校验始终在线下兜底）。

## 更新规则池

直接编辑 `rules/pool.json` 后推送到 `main`：

| 字段 | 说明 |
|---|---|
| `version` | 每次必 +1（客户端按此判断新旧） |
| `rules.absolute_words` | 极限词（广告法风险，high 级） |
| `rules.ai_transitions` | AI 腔短语（累计 ≥3 触发 mid 级） |
| `patterns` | 预留：联系方式等正则 |
| `lessons` | 共享教训 `[{category, detail, time}]`，并入 LLM 写作提示 |

用户贡献流程：客户端「导出贡献包」（本地 lessons JSON）→ 发维护者 →
审核合并进本文件 → 推送后 24h 内全体生效。

## 发版

1. 私有仓库 bump `secrets.CURRENT_VERSION` + 提交推送
2. 打安装包（zip），本仓库 `gh release create vX.Y.Z` 挂包并写更新日志
3. 客户端启动即检查到新版本

## 启用官网（一次性）

GitHub → Settings → Pages → Source: `main` / root → Save。
访问地址：`https://wei-zhang09.github.io/miaobi-site/`
