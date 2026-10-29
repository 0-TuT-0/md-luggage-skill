# md-baggage v0.2.0 使用说明

md-baggage 是“轻量提醒入口＋文档管理内核”，按需准备资料、保存有效结论、维护验证与经验。方法与边界见 [SKILL.md](SKILL.md)，本文用于安装和日常操作。

以下方法适用于 Codex。安装包是 `skills/md-baggage/` **整个目录**；`docs/md-baggage/` 是接入项目后的资料位置，不是 Skill 安装位置。

## 先区分安装与采用

| 事项 | 选择 | 含义 |
| --- | --- | --- |
| 安装范围 | 项目级／全局级 | 工具在哪里发现 Skill |
| 采用范围 | 个人／团队 | 项目资料如何管理、是否采用团队共同约定 |

默认个人采用。项目级安装副本也可以只留本地，不必提交；团队明确决定共享该副本时，才将它纳入项目 Git。个人模式中的 `share/` 表示适合后续分享，不表示已经发布或成为团队规范。

同一使用环境通常保留一份同名 Skill。Codex 不会合并项目级与用户级的同名技能。[官方发现位置与符号链接支持](https://learn.chatgpt.com/docs/build-skills)。其他工具按其自身规则安装，不能直接套用 Codex 路径。

## 准备源码

已有本地仓库直接使用。首次下载可运行：

```bash
mkdir -p "$HOME/coding"
git clone https://github.com/0-TuT-0/md-luggage-skill.git "$HOME/coding/md-baggage-skill"
```

私有仓库需要当前 Git 账号有读取权限。远程仓库名为 `md-luggage-skill`，本地目录为 `md-baggage-skill`，Skill 名为 `md-baggage`。

在同一终端设置实际源码位置：

```bash
MD_BAGGAGE_REPO="$HOME/coding/md-baggage-skill"
```

## 项目级安装＋个人采用

以下示例适用于已有 Git 仓库，包括 worktree。将项目路径替换为实际仓库根目录。初次安装前，先确认目标目录没有自己的旧安装或他人的资料；有则先核对版本、权威来源与更新范围。

安装副本和受管资料使用仓库本地 `info/exclude` 精确排除，不修改团队共享 `.gitignore`。已被 Git 跟踪的文件不受忽略规则影响，不能自动取消跟踪；也不要忽略整个 `.agents/`。[Git 排除规则](https://git-scm.com/docs/gitignore)。

```bash
MD_BAGGAGE_PROJECT="/你的项目绝对路径"

(
  set -eu
  [ -f "$MD_BAGGAGE_REPO/skills/md-baggage/SKILL.md" ] || {
    printf '%s\n' '请检查源码路径。'; exit 1;
  }
  MD_BAGGAGE_PROJECT=$(cd "$MD_BAGGAGE_PROJECT" && pwd -P)
  MD_BAGGAGE_GIT_ROOT=$(git -C "$MD_BAGGAGE_PROJECT" rev-parse --show-toplevel)
  [ "$MD_BAGGAGE_PROJECT" = "$(cd "$MD_BAGGAGE_GIT_ROOT" && pwd -P)" ] || {
    printf '%s\n' '请填写 Git 仓库根目录。'; exit 1;
  }
  MD_BAGGAGE_DEST="$MD_BAGGAGE_PROJECT/.agents/skills/md-baggage"
  MD_BAGGAGE_DOCS="$MD_BAGGAGE_PROJECT/docs/md-baggage"
  if [ -e "$MD_BAGGAGE_DEST" ] || [ -L "$MD_BAGGAGE_DEST" ] ||
     [ -e "$MD_BAGGAGE_DOCS" ] || [ -L "$MD_BAGGAGE_DOCS" ]; then
    printf '%s\n' '已有安装或受管资料，请先核对，勿直接覆盖。'; exit 1
  fi
  if [ -n "$(git -C "$MD_BAGGAGE_PROJECT" ls-files -- .agents/skills/md-baggage docs/md-baggage)" ]; then
    printf '%s\n' '目标范围已有 Git 跟踪文件，请先明确个人接入方案。'; exit 1
  fi
  MD_BAGGAGE_EXCLUDE=$(git -C "$MD_BAGGAGE_PROJECT" rev-parse --path-format=absolute --git-path info/exclude)
  mkdir -p "$(dirname "$MD_BAGGAGE_EXCLUDE")" "$MD_BAGGAGE_PROJECT/.agents/skills"
  printf '\n# md-baggage personal\n/.agents/skills/md-baggage/\n/docs/md-baggage/\n' >> "$MD_BAGGAGE_EXCLUDE"
  for MD_BAGGAGE_CHECK in .agents/skills/md-baggage/SKILL.md docs/md-baggage/share/README.md; do
    git -C "$MD_BAGGAGE_PROJECT" check-ignore -q -- "$MD_BAGGAGE_CHECK" || {
      printf '%s\n' '排除规则未生效，请先核对现有 Git 规则。'; exit 1;
    }
  done
  cp -R "$MD_BAGGAGE_REPO/skills/md-baggage" "$MD_BAGGAGE_DEST"
)
```

这一步只安装 Skill 并设置本地 Git 排除，不创建项目资料档案。随后在该项目会话中明确授权个人接入：

```text
使用本项目的 $md-baggage，按个人方式接入并建立必要资料入口。
允许保存本轮后续开发需要的资料到 docs/md-baggage/；
保留团队原有规范、规格和测试来源，不改公共 AGENTS.md／CLAUDE.md，
不迁移旧资料，不提交或发布项目文件。
```

这个请求已经明确授权建档，无需再重复询问。仅要求只读准备时，Skill 不应据此建档；首次需要接入而缺少授权时只询问一次，未获授权后不再主动询问或催促，用户主动提出后再按新范围处理。

个人资料默认全部留在本地，包括 `share/` 和独立个人验证用例。原有正式规格与测试继续引用原处，受管资料保存本轮实际差异、实施与验证记录，不复制另一套权威正文。

Git 排除不构成访问隔离；同一物理工作目录中的其他会话仍可能发现项目 Skill 和读取这些文件。需要独立工作空间时使用各自 checkout，而不是通过忽略规则声称文件对其他 AI 不可见。

## 全局级安装

跨项目使用时可在用户目录链接本地源码；源码目录需保留在原位置。目标已存在时停止核对，包括失效链接。

```bash
MD_BAGGAGE_DEST="$HOME/.agents/skills/md-baggage"

if [ ! -f "$MD_BAGGAGE_REPO/skills/md-baggage/SKILL.md" ]; then
  printf '%s\n' '请先检查源码路径。'
elif [ -e "$MD_BAGGAGE_DEST" ] || [ -L "$MD_BAGGAGE_DEST" ]; then
  printf '%s\n' "目标已存在，请先核对现有版本：$MD_BAGGAGE_DEST"
else
  mkdir -p "$HOME/.agents/skills" &&
    ln -s "$MD_BAGGAGE_REPO/skills/md-baggage" "$MD_BAGGAGE_DEST"
fi
```

全局安装不代表每个项目已经接入，也不代表团队采用。个人接入仍按上面的范围授权和项目本地排除规则处理；此时无需排除一个并不存在的项目安装副本。链接会直接使用源码工作区内容，包括未提交修改。

## 确认加载与实际执行

Codex 会发现 Skill 变化；未出现时重启后检查安装路径与 `SKILL.md`。[官方说明](https://learn.chatgpt.com/docs/build-skills)。在支持 `$` 的界面可点名 `$md-baggage`，也可从技能选择器中选择。

首次使用先让 AI 列出实际加载的 Skill 路径和版本，再核对以下结果：

- 加载 v0.2.0，且没有同名全局／项目副本混用。
- 个人安装副本与 `docs/md-baggage/` 被本地规则排除，没有进入待提交文件。
- 受管入口能定位原规范、当前需求与权威来源，必要正文已实际保存。
- `test.md` 能定位原测试及其执行入口；独立用例标为个人验证，不宣称团队 CI／正式回归已覆盖。
- 公共入口、旧资料和跟踪状态符合本次授权，没有因接入而被改写。

可在同一终端查看 Git 状态和实际排除来源：

```bash
git -C "$MD_BAGGAGE_PROJECT" status --short
git -C "$MD_BAGGAGE_PROJECT" check-ignore -v -- .agents/skills/md-baggage/SKILL.md docs/md-baggage/share/README.md
```

自动选择保持开启，匹配不保证每轮调用。Matt 技能独立使用；文档维护不扩大实现、运行测试、提交、合并或发布授权。

## 日常资料与本地状态

- 准备讨论：“使用 $md-baggage 为当前需求整理相关规范、依赖和未决问题。”临时重述已有资料可留在聊天；后续需要的新事实、决定、问题与交接内容写回受管正文。
- 同步变化：“回写刚确认的规则，保留待实现／未验证状态。”个人采用的工作目标不自动成为团队已批准规则。
- 经验复用：“提示可能相关的经验、适用条件和差异，先讨论采用范围。”经验留在项目，跨项目升级另行决定。
- 收尾与恢复：清理本任务已完成用途的临时探针和无效重复试验，先保存有用结论；稳定个人用例、未决试验及复现证据保留。无后台定时清理。

采用选择保存在用户目录的 `~/.md-baggage/projects.json`，只记录项目标识及 `asked`／`personal`／`team`；`asked` 表示已询问但未获授权，不再主动询问。Git 项目以规范绝对 Git 公共目录作标识，同一 clone 的 worktree 复用选择；非 Git 项目使用规范绝对项目根目录。记录不保存业务资料，也不代表未来操作已获授权。具体状态行为见 [接入规则](references/onboarding.md)。

## 更新与旧布局

先检查源码仓库的分支与工作区：

```bash
git -C "$MD_BAGGAGE_REPO" branch --show-current
git -C "$MD_BAGGAGE_REPO" status --short
```

确认在 `main`、工作区干净且要更新到远程最新版后，再运行：

```bash
git -C "$MD_BAGGAGE_REPO" pull --ff-only origin main
```

全局链接直接使用更新后的源码；项目副本需先比较本地定制与新版差异，再更新整个目录，避免套出嵌套的 `md-baggage/md-baggage/`。个人更新不提交本地安装副本；团队共享副本按其审查约定处理。

v0.2.0 的资料根固定为 `docs/md-baggage/`。发现旧版 `docs/share/`、`docs/private/` 或其他既有权威资料时，先说明当前来源与迁移方案，确认前不自动搬动或另建第二套现行正文。更新 Skill 包不会自动完成项目迁移。
