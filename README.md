# 科研制图技能 · scientific-visualization

把已核验的数据制作成论文图，保留科学含义、可运行源文件和可靠导出。支持数据曲线、热图、多面板、简单方法示意图，以及尺寸、DPI、字体和配色检查。此仓库是可移植适配版，不是 K-Dense 官方发布。

版本 **2.2-portable**。核心功能可独立使用，不需要 Vivid 或其他科研技能。保留 7 个脚本与公开接口；测试由12项扩展至19项，覆盖尺寸/DPI边界与配置单次读取、失败不污染样式。本包不包含 Vivid 模板、字体或论文图像。

## 安装到 Codex

下载或克隆你实际上传的仓库后，在仓库根目录运行。无需初始化 Git、连接账号或上传任何数据。

按照 [Codex 当前技能说明](https://learn.chatgpt.com/docs/build-skills)，新环境推荐用户级 `~/.agents/skills/`，项目级可用 `.agents/skills/`。旧环境若已经识别 `$CODEX_HOME/skills` 或 `~/.codex/skills`，可继续使用该位置。只选择一个已识别的位置，避免重复发现同名技能。复制整个 `skills/scientific-visualization/`，不要只复制 `SKILL.md`。

PowerShell（默认新环境；先备份现有目标）：

```powershell
$skillSource = Join-Path (Get-Location) 'skills/scientific-visualization'
if (-not (Test-Path -LiteralPath (Join-Path $skillSource 'SKILL.md'))) { throw '请先进入仓库根目录' }
$skillBase = Join-Path $HOME '.agents/skills'
# 已确认使用旧目录时，可改为：
# $skillBase = if ($env:CODEX_HOME) { Join-Path $env:CODEX_HOME 'skills' } else { Join-Path $HOME '.codex/skills' }
$skillTarget = Join-Path $skillBase 'scientific-visualization'
New-Item -ItemType Directory -Path $skillBase -Force | Out-Null
if (Test-Path -LiteralPath $skillTarget) {
    $skillBackupBase = Join-Path (Split-Path $skillBase -Parent) 'skill-backups'
    New-Item -ItemType Directory -Path $skillBackupBase -Force | Out-Null
    $skillBackup = Join-Path $skillBackupBase "scientific-visualization-$([guid]::NewGuid().ToString('N'))"
    Move-Item -LiteralPath $skillTarget -Destination $skillBackup
    Write-Output "原技能已备份到 $skillBackup"
}
Copy-Item -LiteralPath $skillSource -Destination $skillTarget -Recurse
```

macOS/Linux（相同的整目录安装与备份原则）：

```bash
skill_base="$HOME/.agents/skills"
# 仅在已确认旧目录被识别时：skill_base="${CODEX_HOME:-$HOME/.codex}/skills"
[ -f skills/scientific-visualization/SKILL.md ] || { printf '请先进入仓库根目录\n'; exit 1; }
mkdir -p "$skill_base"
skill_target="$skill_base/scientific-visualization"
if [ -e "$skill_target" ]; then
  skill_backup_base="$(dirname "$skill_base")/skill-backups"
  mkdir -p "$skill_backup_base"
  skill_backup="$(mktemp -d "$skill_backup_base/scientific-visualization.XXXXXX")"
  mv "$skill_target" "$skill_backup/scientific-visualization"
  printf '原技能已备份到 %s\n' "$skill_backup/scientific-visualization"
fi
cp -R skills/scientific-visualization "$skill_target"
```

安装后新开会话，输入 `$scientific-visualization`，或自然语言说明绘图需求。若未发现技能，先检查当前运行环境识别的技能目录，不要修改全局配置碰运气。

## Python 环境与首次运行

需要 Python 3.11+。优先复用项目环境；仅当用户希望隔离环境时才新建虚拟环境。确认当前 `python` 指向选定环境后，在仓库根目录执行：

```bash
python -m pip install -r requirements.txt
python -m unittest discover -s skills/scientific-visualization/tests -v
python skills/scientific-visualization/scripts/figure_export.py --help
```

如果仅复制了技能目录，进入该目录后执行 `python -m pip install -r requirements.txt`，相对路径改为 `scripts/` 和 `tests/` 即可。Matplotlib 绘图、Pillow 栅格检查和 pypdf PDF 检查均由该依赖文件覆盖。版本范围不是完整兼容性矩阵；本次实际测试版本见 [PORTABILITY.md](PORTABILITY.md)。

无图形界面的终端设置 `MPLBACKEND=Agg`（PowerShell：`$env:MPLBACKEND='Agg'`；Bash：`export MPLBACKEND=Agg`）。

先建立一个新的输出目录，再运行真实导出冒烟测试：

```bash
mkdir outputs
python skills/scientific-visualization/scripts/figure_export.py --demo outputs/demo --formats pdf,png,svg --dpi 300 --manifest
python skills/scientific-visualization/scripts/image_metadata.py outputs/demo.png --format png --min-dpi 300
```

示例是解析生成的 sin/cos 数据，不是实测科研结果。默认不覆盖现有文件；需要重跑时换一个空目录。只有明确要替换同名目标时才用 `--force`。

## 使用示例

> 使用 `$scientific-visualization`，按这份已核验的频响结果绘制 89 mm 单栏图。缺失点保留断线，使用色盲友好配色与不同标记，导出可编辑 PDF 和 600 DPI PNG。不要重新估计置信区间，保留图源并检查裁切、单位和字体。

简单关系示意图可用随包的独立 SVG 示例；复杂 `.drawio` / PPTX 需求按用户已安装工具处理。细节见 [模板与示意图](skills/scientific-visualization/references/diagram_and_templates.md)。

## 主要文件

改良与版本见 [CHANGELOG.md](CHANGELOG.md)。README、LICENSE、CHANGELOG位于仓库根；唯一SKILL入口及其资源位于完整安装单元中。

| 文件 | 用途 |
| --- | --- |
| [SKILL.md](skills/scientific-visualization/SKILL.md) | 技能入口、科研含义与交付边界 |
| `scripts/figure_export.py` | 显式尺寸、多格式、拒绝意外覆盖的 Matplotlib 导出 |
| `scripts/image_metadata.py` | 栅格、SVG、PDF、EPS 属性检查 |
| `scripts/export_plan.py` | 带日期的出版商规格规划；不保证当前投稿合规 |
| `scripts/palette_audit.py` | 背景对比、灰度辅助检查 |
| `scripts/style_presets.py` / `style_preview.py` | 局部样式与可复现预览 |
| `scripts/_common.py` | 各工具共享的文件与参数校验 |

脚本和资源路径均相对技能目录。每个可执行工具的具体参数见 `python ... --help`。

## 重要边界

- 不补造数据、误差线、显著性、SHAP 值或模型关系；新统计分析不是此技能的自动附加任务。
- `statistical-analysis`、`research-writing-skill` 是可选同伴，不是必装依赖。
- Seaborn、Plotly/Kaleido、Chrome、TeX/PGF、Cairo、Office 和外部模板库均不是核心依赖，未由本次发布验证；明确需要时再单独评估、安装与测试。
- Vivid 是受单独限制性许可约束的外部项目，本包只链接其公开来源；未授权不得改编/打包。不要从本包 MIT 许可证推断外部资产授权。
- 期刊规则和文档中的包版本是带日期快照，不是今天的合规保证。投稿前必须检查具体期刊当前官方要求。
- 中文字体需使用本机有合法使用权的字体并实际渲染检查。本包不附带字体；元数据检查不能替代视觉检查或科学审核。

## 许可与验证

[MIT LICENSE](LICENSE) 保留 K-Dense Inc. 版权声明；[完整第三方说明](skills/scientific-visualization/THIRD_PARTY_NOTICES.md)解释历史来源、本次适配、配色数值及排除的素材。许可证和说明也放在技能目录内，独立安装时一并保留。

发布基线和本次验证结果见 [PORTABILITY.md](PORTABILITY.md)。仓库不含实际论文数据、环境缓存、个人配置或自动上传功能。
