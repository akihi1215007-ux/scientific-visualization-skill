# 可移植性基线与验证记录

## 优化版本（2026-11-03，2.2-portable）

新增7项回归后，19项测试全部通过；计算包通过同级发布副本运行9项制图兼容测试。`screen_metadata` 拒绝非有限/非正尺寸与DPI、相反上下限。`configure_for_journal` 一次调用只读取一份期刊配置，先校验日期、尺寸、配色与样式，再修改全局状态。有效输入的接口、PNG量化容差与有效DPI判据保留。

以下2026-11-02记录属于历史基线，原脚本哈希不再表示本轮修改后的文件，也不作为接口兼容的证明。未改动的许可原文保留；本轮详情见仓库变更记录。

## 修改前基线（2026-11-02）

从既有科研制图技能复制 20 个有效文件：`SKILL.md` 1 个、`scripts/` 7 个、`assets/` 5 个、`references/` 6 个、`tests/` 1 个。排除 5 个已生成的 `.pyc` 缓存文件；原目录只读。

原 12 项 `unittest` 回归测试全部通过。环境：Python 3.12.14、Matplotlib 3.11.2、NumPy 2.5.4、Pillow 12.3.0、pypdf 6.20.1；`PYTHONUTF8=1`、`PYTHONDONTWRITEBYTECODE=1`、`MPLBACKEND=Agg`。

可移植性检查在修改前失败：`SKILL.md` 的 2 行、`references/diagram_and_templates.md` 的 11 行含本地绝对盘符路径，包括唯一解释器、同伴技能、Vivid 和表格素材入口。该检查排除了 `https://`。原入口还假定固定编号的工作链，且不存在独立安装依赖声明和完整随包许可证。

当日修复仅针对包装与说明，不修改既有脚本、资源和测试逻辑。原始 SHA-256（仅供核对历史文件身份）：

| 文件 | SHA-256 |
| --- | --- |
| `scripts/_common.py` | `334BEDAC12F899C4DFACE3A8F2740F75ADA51586948788B6D101F8431BBB5FCB` |
| `scripts/export_plan.py` | `06DFD7C4D5D7296CDC8728B4727F74AA9803AEFE438522DF6D42E041078364D9` |
| `scripts/figure_export.py` | `DA7A6DF1FE418B4916F3EA5DDE01A814B68291CB24E33B6A128A39706FB264D8` |
| `scripts/image_metadata.py` | `68111F17DBCA3185CE208BBC65AC8013AF29E82FD52FE612A89D1865766FED5F` |
| `scripts/palette_audit.py` | `6BD51ADD71388CD111E531CC60DEAB38A675DDF2F1845381546FE3BDBF923F13` |
| `scripts/style_presets.py` | `51B42DEB739EECB64027FF584E7160DF30A12B34D641D3063191FB02D4BB24A9` |
| `scripts/style_preview.py` | `5654AF7096AF50E107812D17659A2DCC0F551B17219B10CBEB450FEB4F0A0BB5` |
| `tests/test_metadata_and_styles.py` | `0E15787DCA138245D54AD213F235214F3A1CB89A6C7E1C257F4F3230704C6459` |

## 历史发布副本验证（2026-11-02）

本次发布使用上面列出的已安装环境，并未重新安装依赖或验证全部版本范围。在发布目录和另一个新建的临时目录中各运行一次完整测试，结果均为 **12/12 通过**：

```bash
python -m unittest discover -s skills/scientific-visualization/tests -v
```

临时目录只含从发布包复制出的完整技能，导出器不需要源机器的旧技能路径。验证结果：

| 检查 | 结果 |
| --- | --- |
| skill-creator `quick_validate.py` | `Skill is valid!` |
| 脚本、资源和测试文件哈希 | 原 7 脚本、5 资源、1 测试文件逐个一致 |
| 本地绝对路径/个人路径扫描 | 0 命中；排除 URL 和代码内非路径正则 |
| Markdown 本地链接解析 | 0 个缺失目标；未宣称遍历验证所有历史外部网页 |
| 6 个可执行 CLI 的 `--help` | 全部退出码 0；`_common.py` 是辅助模块 |
| 技能入口完整 Python 示例 | 在搬迁目录执行成功，生成 PDF/PNG |
| 独立 SVG 示例 | XML 解析成功，3 个矩形、3 个文本节点 |
| 随包 MIT 许可 | 与已核对的上游固定提交文本一致（忽略空白） |
| 缓存/二进制素材 | 0 个 `.pyc`/`__pycache__`；无捆绑字体或 Vivid 素材 |

实际导出使用临时输出目录，等效命令如下：

```bash
python scripts/figure_export.py --demo outputs/demo --formats pdf,png,svg --dpi 300 --manifest
python scripts/style_preview.py --output outputs/preview --formats png,svg --dpi 150
python scripts/image_metadata.py outputs/demo.png --format png --min-dpi 300
python scripts/image_metadata.py outputs/demo.pdf
python scripts/image_metadata.py outputs/demo.svg
```

- 演示 PNG 为 1050 × 750 像素，标称 DPI 299.9994（PNG pHYs 量化），300 DPI 下限检查通过。
- 演示 PDF/SVG 为 88.9 × 63.5 mm；PDF 首页面声明字体为已嵌入，SVG 保留 20 个文本元素且无外链图像。字体报告仅覆盖检查器的首页面资源范围，不等于全文件字体审计。
- 两张 PNG 已实际打开检查，标签、图例和画布边界未见裁切。样式预览的灰度启发式检查提示 4 对颜色需要关注；预览使用标记/线型/纹理冗余编码。此结果不是完整无障碍认证。
- 导出器无警告；临时样例和导出报告未放进仓库。原 20 个源文件未改动，不执行安装、Git 初始化、提交、上传或打包压缩。

## 未验证范围

没有证明每个 Python/依赖版本组合都兼容，也未执行全新环境安装、Linux/macOS、中文字体、Plotly/Kaleido、TeX、Cairo、Office、draw.io 或 Vivid 端到端测试。没有重新核验全部历史期刊网页，文件属性通过不表示科学结论、出版商要求或无障碍已全部合格。
