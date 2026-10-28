# md-baggage 使用说明

md-baggage 是“轻量提醒入口＋文档管理内核”：按需准备需求资料，维护已确认的决定、进度、验证和经验。有方法支持需要时提醒使用独立的 Matt 技能；具体规则见 [SKILL.md](SKILL.md)。

以下安装方法适用于 Codex。安装包是 `skills/md-baggage/` **整个目录**，包含 `SKILL.md`、`agents/`、`references/`、`assets/` 和本说明。

## 选择安装范围

| 方式 | 安装位置 | 适合场景 |
| --- | --- | --- |
| 项目级：复制目录 | `<项目根目录>/.agents/skills/md-baggage/` | 仅供当前项目使用；团队可将该副本纳入项目 Git |
| 全局级：符号链接 | `~/.agents/skills/md-baggage` | 当前用户跨项目使用；适合维护本 Skill 源码的人 |

同一使用环境通常选一种即可。Codex 对项目级和全局级的同名 Skill 不做合并，两份可能同时出现。[官方安装位置与链接支持](https://learn.chatgpt.com/docs/build-skills)。其他 AI 工具请按其自身的 Skill 发现规则安装。

## 准备源码

已有本地仓库时直接使用，不必重复克隆。首次下载可运行：

```bash
mkdir -p "$HOME/coding"
git clone https://github.com/0-TuT-0/md-luggage-skill.git "$HOME/coding/md-baggage-skill"
```

私有仓库需要当前 Git 账号具有读取权限。远程仓库名是 `md-luggage-skill`，本地目录和 Skill 名分别是 `md-baggage-skill`、`md-baggage`。

在终端设置实际源码位置；下文命令在同一个终端执行：

```bash
MD_BAGGAGE_REPO="$HOME/coding/md-baggage-skill"
```

## 项目级安装

将下面的项目路径换成你的项目根目录，再运行。目标已有文件、目录或链接时，命令会停止，保留现有安装。

```bash
MD_BAGGAGE_PROJECT="/你的项目绝对路径"
MD_BAGGAGE_DEST="$MD_BAGGAGE_PROJECT/.agents/skills/md-baggage"

if [ ! -d "$MD_BAGGAGE_PROJECT" ] || [ ! -f "$MD_BAGGAGE_REPO/skills/md-baggage/SKILL.md" ]; then
  printf '%s\n' '请先检查项目路径和源码路径。'
elif [ -e "$MD_BAGGAGE_DEST" ] || [ -L "$MD_BAGGAGE_DEST" ]; then
  printf '%s\n' "目标已存在，请先核对现有版本：$MD_BAGGAGE_DEST"
else
  mkdir -p "$MD_BAGGAGE_PROJECT/.agents/skills" &&
    cp -R "$MD_BAGGAGE_REPO/skills/md-baggage" "$MD_BAGGAGE_DEST"
fi
```

在 Codex 中打开这个项目。团队需要复用时，将复制后的 `.agents/skills/md-baggage/` 随项目提交；它不依赖你的个人源码路径。

## 全局级安装

以下命令建立链接，指向上述本地仓库中的 Skill 目录；源码目录需要保留在原位置。

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

“全局”指当前用户的各个项目。链接会直接使用源码工作区内容，包括尚未提交的修改；个人源码路径不应作为团队共享安装配置提交。

## 确认可用与首次接入

在对应项目会话中查看技能列表，确认出现 `md-baggage`／“开发文档管家”。Codex 会自动发现 Skill 变化；未出现时重启后再检查安装目录和 `SKILL.md`。[官方发现说明](https://learn.chatgpt.com/docs/build-skills)。

在支持 `$` 语法的 Codex 界面中可用 `$md-baggage` 点名；提供技能选择器的界面也可直接选择该 Skill。首次接入可以这样请求：

```text
使用 $md-baggage 接入当前项目：先盘点现有文档和 AI 规范，
给出最小接入方案；暂不迁移旧文档。
```

安装只让 Codex 找到 Skill。项目接入另行维护根 `AGENTS.md`、`docs/share/README.md` 等实际入口，沿用已有规则；个人过程放 `docs/private/` 并排除版本控制。已有项目的迁移仍先给方案，确认后再执行。详见 [接入规则](references/onboarding.md)。

## 日常使用

- 准备讨论：“使用 $md-baggage 为报名需求准备 grill 所需资料，只读取相关规范、依赖和未决问题。”
- 同步决定：“使用 $md-baggage 回写刚确认的规则，标明尚未实现、尚未验证的部分。”
- 经验复用：“使用 $md-baggage 提示本需求可能复用的经验和适用差异，先讨论是否采用。”

自动选择已开启，实际触发取决于当前请求与工具的匹配机制；需要明确使用时直接点名。项目接入后的简短提醒规则由当前 AI 遵守，有实际需要才提醒；安装不意味着后台监听或每轮必定调用。

Matt 技能保持独立，需要时另行准备并直接使用。md-baggage 的文档维护授权不包含自动实现、执行测试、提交、合并或发布。

## 更新

先检查源码仓库的分支和工作区：

```bash
git -C "$MD_BAGGAGE_REPO" branch --show-current
git -C "$MD_BAGGAGE_REPO" status --short
```

确认分支为 `main`、状态无输出，并且要更新到远程最新版后再运行：

```bash
git -C "$MD_BAGGAGE_REPO" pull --ff-only origin main
```

全局链接直接使用更新后的源码；源码移动后需要修正链接。项目副本需先比较本地定制与新版差异，再更新整个 Skill 目录，并通过项目 Git 审查。避免把新目录再次复制进已有 `md-baggage/` 中而形成嵌套目录。更新 Skill 包不会自动迁移或重写项目文档。
