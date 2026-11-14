# 个人维护说明（ssacko510-stack 维护分支）

本仓库是 `dsh-purge 1.1.71` 的个人长期维护副本，来源链路：

| 环节 | 仓库 |
| --- | --- |
| 原始作者（设计 / 提示词 / skills） | `https://github.com/YuJunZhiXue/dsh-purge`（MIT） |
| 社区兼容版（1.6.x，上游活跃线） | `https://github.com/kingcharleslzy-ai/dsh-purge` |
| 1.1.71 兼容快照（本仓库直接来源） | `https://github.com/realMisakaMikoto/dsh-purge` |
| 本仓库（个人维护） | `https://github.com/ssacko510-stack/dsh-purge-fork` |

本仓库只承载 **DSH Desktop 0.2.1-rc.1 / 0.2.2-rc.2.1 兼容修复**，不含任何本机数据
（`$DSH_HOME/redteam/config.json`、凭据、engagement 记录都不在仓库里）。

## 本地三份副本的关系

| 路径 | 角色 |
| --- | --- |
| `D:\02_开发工程\工具\dsh-purge源码` | 原始导入快照（只读参考，不要在这里改） |
| `D:\09_成果输出\DSH成果\05-DSH与AI工具链\dsh-purge-个人维护版` | 工作副本 = 本仓库根 = 以后所有改动的落点 |
| `%USERPROFILE%\.dsh\profiles\desktop\node_modules\dsh-purge` | DSH 实际加载的已安装版本（由 tgz 装进去） |

改动流程：改工作副本 → `node --check` → `git commit && git push` → 需要装机时重新打包 tgz 覆盖安装。

## 日常操作

```bash
cd "D:/09_成果输出/DSH成果/05-DSH与AI工具链/dsh-purge-个人维护版"

# 1. 语法自检（必须全绿再提交）
NODE="C:/Users/Honor/.dsh/dsh-runtimes/dsh-primary-runtime/dependencies/node/bin/node.exe"
"$NODE" --check lib/index.js
"$NODE" --check lib/core.js
"$NODE" --check lib/reapply.js

# 2. 提交
git add -A
git commit -m "fix: <一句话说明>"
git push origin main
```

## 同步上游（可选）

上游 1.6.x 已经和 1.1.71 分叉得比较远，不要直接 merge，按需 cherry-pick 单点修复：

```bash
git remote add community https://github.com/kingcharleslzy-ai/dsh-purge.git
git fetch community main
git log --oneline community/main | head -30          # 看有哪些修复
git show <sha> --stat                                 # 看单个修复动了哪些文件
```

只挑 `lib/core.js`、`lib/host.js`、`lib/extract-asar.js` 这类宿主兼容补丁手动移植，
不要把上游整棵 `lib/redteam` 拖进来——1.6.x 的 store / schema 与本版不兼容。

## 已包含的兼容修复（1.1.71 基础 + 本分支）

1. `COMPLETE_PROMPT_KEEP_INJECT` 新增 rc.2 专用 pattern `COMPLETE_PROMPT_WATERFALL_INJECT_02`
   加兜底 pattern，避免官方改形后 `skipped` 导致永久取消重启。
2. `HOST_SYNC` 补 `closeSync / readSync / writeSync / fsyncSync / fstatSync / accessSync`，
   修复备份 0 条、`dsh.ps1` 不注入。
3. 红队 preset 在 rc.2 注册即 broken 的修复。
4. 仓库元数据指向本仓库，release workflow 分支从 `master` 改为 `main`。

## 版本策略

- 补丁版号写在 `package.json` 的 `version`，当前 `1.1.71`。
- 每次实改动提交，推 `main` 即可；release workflow 只在版本号变化时打 tag。
- 发布用的 zip 由 `.github/workflows/release.yml` 在 `main` 上自动产出。

## 打包安装（需要重新装机时）

```bash
cd "D:/09_成果输出/DSH成果/05-DSH与AI工具链/dsh-purge-个人维护版"
NODE="C:/Users/Honor/.dsh/dsh-runtimes/dsh-primary-runtime/dependencies/node/bin/node.exe"
"$NODE" "C:/Users/Honor/.dsh/dsh-runtimes/dsh-primary-runtime/dependencies/pnpm/bin/pnpm.mjs" pack
# 生成的 dsh-purge-1.1.71.tgz 复制到 profile 并在 package.json 里把依赖改成 file: 路径
```
