# AIP304 CW2 Option 2：第一天数据准备任务书（供 Codex 执行）

## 1. 任务目标与执行范围

你正在协助完成 AIP304 Coursework 2 的 Option 2。团队尚未确定明确分工，当前只完成所有成员后续都能复用的数据准备工作。

**本阶段目标：下载正确版本的数据，理解图片与标注，完成基础审计，并生成 Assignment A/B 的可复现划分清单。**

后续主要工作在一个 Jupyter Notebook 中进行，再上传到用户指定的 GitHub 仓库。当前文档是一份执行说明，不表示数据已下载、审计已完成或实验已经运行。

执行要求：

- 先阅读现有仓库的 AGENTS.md、README、Notebook 与任务书；保留已有工作。
- 如果已有主 Notebook，就在其中补充第一天内容；否则创建 `AIP304_CW2_Option2.ipynb`。
- 使用一个主 Notebook 承载数据准备及后续实验，避免创建多个互相依赖的 Notebook。
- 本阶段不训练模型，不做超参数搜索，不计算测试性能，不开展完整文献综述。
- 不虚构实际目录、标签、数据数量、checksum 或运行结果。先检查真实文件与 metadata，再实现解析逻辑。
- 遇到下载受限时，保留可运行的下载或手动放置说明，完成不依赖数据的部分，并准确列出尚未执行的步骤。
- 完成可逆的常规操作，不要因普通目录或字段命名反复请求确认。真正缺失的信息和协议冲突应明确记录。

## 2. 依据与优先级

依据：用户提供的 `AIP304 CW2 Task Sheet(3).pdf`。

重点位置：

- 第 2 页：数据版本、九种宿主、原始分区、数据审计与防泄漏。
- 第 4–6 页：Option 2 类别角色、Assignment A/B、mask 使用限制与评估协议。
- 第 7–8 页：Notebook、复现、提交和评分要求。

本文中的目录、字段、函数名和第一天工作顺序是实施建议，不是任务书新增要求。若实际任务书有更新，以最新版原文为准并记录变化。

## 3. 必须遵守的实验约束

1. 使用 **PlantSeg v7**，发布于 2025-11-25，不混用其他版本。
2. 下载记录：<https://zenodo.org/records/17719108>；文件为 `plantseg.zip`，约 1.1 GB。
3. 只保留以下九种 host：`apple, banana, citrus, corn, potato, soybean, tomato, wheat, zucchini`。
4. 一个 class 是一个 host–disease pair；同一病害在不同宿主上是不同类别。
5. 任务书预期子集有 43 类，但必须从实际 metadata 导出并核对，不能制造或硬编码缺失类别。
6. 保留原始约 70% training、10% validation、20% test 分区；不得重新随机划分，也不得跨分区搬图片补样本。
7. 训练只使用 known classes 的原始 training 图片。
8. Development unknown 只使用其原始 validation 图片；test unknown 只使用其原始 test 图片；其余图片不进入实验。
9. Test-unknown 图片不得参与训练、特征拟合、校准或模型选择。
10. 二值病斑 mask 只用于 known training 数据的训练监督。Validation/test 预测只使用 RGB，不能输入真实 mask、host metadata 或含类别信息的文件名。
11. 今天可对所有分区做程序化文件完整性、数量与重复检查；人工浏览和预处理示例使用已确定的 known training 图片，不根据 test 外观调整方法。
12. 原始数据只读保留；排除通过 manifest 实现，不直接删除或改写原始图片。

## 4. 建议项目结构

| 路径 | 用途 | Git 管理 |
| --- | --- | --- |
| `AIP304_CW2_Option2.ipynb` | 唯一主 Notebook | 提交 |
| `AIP304_CW2_Option2_Day1_Codex.md` | 当前执行说明 | 提交 |
| `README.md` | 数据准备、运行顺序与阶段状态 | 提交 |
| `requirements.txt` | 实际使用的必要依赖 | 提交 |
| `.gitignore` | 排除原始数据、缓存、凭据和大文件 | 提交 |
| `data/raw/` | 下载压缩包与原始解压数据 | 不提交 |
| `artifacts/metadata/` | 数据版本信息与图片索引 | 提交小型文本文件 |
| `artifacts/splits/` | A/B 角色映射和 manifest | 提交 |
| `artifacts/audit/` | 审计统计与异常清单 | 提交小型文本文件 |
| `artifacts/figures/` | 可再生成的统计图 | 按需要提交小型图表 |
| `checkpoints/` | 后续模型权重 | 本阶段不创建权重，不直接入 Git |

若仓库已有合理结构，沿用它并说明路径对应关系。无需增加 Web 服务、复杂配置框架或独立训练工程。

## 5. Notebook 章节与每节交付物

每节先写简短 Markdown 说明目的，再放代码，最后显示关键结果。核心逻辑写为可重复调用的小函数；不要把结果数值直接写死。

### 0. 项目说明与阶段状态

- 写明 Option 2、当前为 Day 1 数据准备阶段。
- 给团队成员信息留占位符，不编造姓名、学号或分工。
- 简述研究目标：比较病斑监督与匹配 baseline 的未知拒绝能力，推理仅用 RGB。
- 记录执行日期、已完成步骤、未完成步骤与阻塞项。

### 1. 环境与路径

- 使用 `pathlib.Path` 和一个可配置的 `DATA_ROOT`，避免个人电脑绝对路径。
- 支持本地、Google Colab 或 Kaggle 中显式配置数据路径。
- 设置展示抽样 seed，记录 Python 和实际使用库的版本。
- 数据准备可在 CPU 运行，不要求 GPU。
- 安装依赖与主要数据处理分开；不要引入尚未使用的大型训练依赖。

### 2. 下载、校验与解压

- 从固定版本记录获取真实文件下载地址与提供的 checksum，不猜测 checksum。
- 记录数据版本、来源 URL、文件名、下载日期、文件大小、checksum 算法与校验状态。
- 已存在且验证通过的文件复用，不重复下载；已有文件必须先确认版本与完整性。
- 校验失败应停止使用该压缩包并报告，不将失败标记为通过。
- 如果来源 checksum 无法取得，记录 `unverified`；本地计算的 SHA-256 只能作为本地指纹，不能声称完成来源核验。
- 安全解压，检查条目路径不越过目标目录；不覆盖不同版本的已有数据。
- 提供手动下载后设置 `DATA_ROOT` 的备用方式。

输出建议：`artifacts/metadata/dataset_info.json`。

### 3. 数据格式探索

首先检查实际目录和标注格式，回答：

- 图片、mask 和 metadata 分别在哪里？
- 原始 training/validation/test 怎样表示？
- host 与 disease 标签来自哪里？
- image 和 mask 如何一一对应？
- 是否存在缺失或多重匹配？

以官方 metadata/分区文件为优先依据；若类别只能从官方目录约定解析，记录证据与规则，但不把路径字符串作为模型输入。

只打印必要的目录样例与字段，不输出数千条文件路径。

### 4. 九宿主索引表

建议生成 `artifacts/metadata/images.csv`，至少包含：

| 字段 | 含义 |
| --- | --- |
| `image_id` | 稳定且唯一的图片标识，优先使用官方 ID |
| `image_path` | 相对于配置数据根目录的 POSIX 路径 |
| `mask_path` | 对应 mask 的相对路径；缺失需标记 |
| `host_raw`, `disease_raw` | 原始标签 |
| `host`, `disease` | 实际用于匹配与排序的标签 |
| `class_name` | 可读的 host–disease 标识 |
| `original_split` | 原始 training / validation / test |
| `width`, `height`, `image_mode` | 图片尺寸和原始模式 |
| `readable`, `mask_exists` | 文件检查状态 |

要求：

- 使用 `(host, disease)` 作为可靠类别键，避免字符串拼接歧义。
- 保留原始标签；如去空格或规范化大小写，记录映射并检查是否合并了不同标签。
- `image_id` 不得仅取可能重复的 basename；缺少官方 ID 时可使用稳定相对路径。
- 不把本地绝对路径写入要提交的 CSV。
- 展示实际 host 数、类别数和图片数；与预期不一致时解释或标记待查。

### 5. 基础数据审计

生成以下结果：

1. 每个 host × original_split 的图片数量。
2. 每个 class × original_split 的图片数量，包括零计数。
3. 无法读取、损坏或缺失的图片清单。
4. 缺失、无法读取、尺寸异常的 mask 清单。
5. 图片尺寸和宽高比分布。
6. 文件级精确重复检查，以及必要的解码像素级重复检查。
7. 跨原始分区重复组的单独清单。

重复检查规则：

- 文件 SHA-256 检测字节相同文件。
- 如做像素级哈希，明确 EXIF 方向、RGB 转换和尺寸处理规则；在原始尺寸上检查，不先缩成 224×224。
- 不把感知哈希相似直接等同于完全重复。
- 发现重复不直接删除原始文件，输出重复组、标签、分区和处理状态。
- 未解决的跨分区精确重复必须阻止最终 manifest 标记为 frozen；不能为了消除重复而把图片移动到另一原始分区。
- 如果已有团队排除规则，统一执行并记录；否则生成 draft manifest 和待决项供核对。
- 如果未做 near-duplicate 检查，明确列为限制，不能声称数据完全无泄漏。
- Mask 异常不能只对病斑模型单独删样本。需要排除时，两个比较方法必须使用一致样本清单。

输出建议：`host_counts.csv`、`class_counts.csv`、`invalid_files.csv`、`mask_issues.csv`、`duplicate_groups.csv`、`exclusions.csv`、`audit_summary.json`。

空问题清单也保留列名，便于后续稳定读取。

### 6. Assignment A/B 类别映射

先在完整九宿主官方类别标签集合上构建角色，不因异常图片排除而改变类别排序。后续若某类可用样本为空，单独报告。

对其余七种 host 的 disease 标签按确定、可复现的字母排序规则排序。保存每个 host 的排序列表与任何规范化规则。

| 类别 | Assignment A | Assignment B |
| --- | --- | --- |
| 所有 citrus 类别 | development_unknown | development_unknown |
| 所有 potato 类别 | test_unknown | test_unknown |
| 其余 host 排序第 1 类 | development_unknown | known |
| 其余 host 排序第 2 类 | test_unknown | development_unknown |
| 其余 host 排序第 3 类 | known | test_unknown |
| 其余类别 | known | known |

Apple 校验样例：

| Disease | A | B |
| --- | --- | --- |
| black rot | development_unknown | known |
| mosaic virus | test_unknown | development_unknown |
| rust | known | test_unknown |
| scab | known | known |

如果真实标签/排序与示例冲突，报告原因，不静默修改数据以通过校验。

额外记录：每个 familiar-host unknown 的 disease 是否出现在该 assignment 其他 host 的 known 类别中。该字段用于后续 shared / previously-unseen disease 分组；只做标签比较，不浏览测试图片。

输出：`class_roles_A.csv`、`class_roles_B.csv`，以及每套 assignment 独立的连续 known-class ID 映射。Unknown 不加入 known 分类输出维度。

### 7. 图片级实验 manifest

规则：

| Class role | 原始 training | 原始 validation | 原始 test |
| --- | --- | --- | --- |
| known | train | validation | test |
| development_unknown | unused | validation | unused |
| test_unknown | unused | unused | test |

建议每张九宿主图片在 manifest 保留一行，记录：

- `assignment`、`image_id`、相对图片及 mask 路径。
- `host`、`disease`、`class_name`、`known_class_id`（未知类为空）。
- `original_split`、`class_role`、`experimental_split`。
- `unknown_group`：known / familiar_host / unfamiliar_host。
- `disease_shared_with_known`：只在相应 unknown 分组适用。
- `included`、`exclusion_reason`、`mask_supervision_allowed`。

对于 unused 或被排除记录，`included=False`。训练/验证/测试加载器必须同时依据 split 和 included 筛选，避免误读未使用样本。

只有 included 的 known train 记录可标记 `mask_supervision_allowed=True`。Manifest 存储路径和标签不等于允许模型将其作为输入。

输出：`manifest_A.csv`、`manifest_B.csv`、每套 assignment 的组大小统计，以及 `split_protocol.json`。

协议文件记录数据版本、排序规则、排除规则、状态 draft/frozen 和清单内容哈希。训练前两套划分都必须核对并冻结；第一天如存在未解决问题，保留 draft 并列出原因，不伪称 frozen。

### 8. 训练样本与 mask 可视化

从明确的 known training 数据中按固定 seed 抽取少量代表性样本；可优先选择 A/B 共同 known training 的图片，若不足则分别标注 assignment。

每组展示：RGB 原图、binary mask、透明叠加图。注明类别、图片 ID 和尺寸。

- 先检查 mask 唯一值、数据格式和背景定义，再决定二值化。
- 不盲目使用 `mask > 0`：如果存在 ignore/void 编码，必须依据数据说明处理。
- 检查 mask 与 image 的尺寸、方向和几何对齐。
- 当发现尺寸错位时先调查，不能通过任意 resize 掩盖对应错误。
- 不使用 test 图片来挑选裁剪策略或增强方法。

### 9. 最小预处理演示

今天只实现读取函数和少量示例，不批量生成永久处理后的图片。

- 保留原图，说明 RGB 转换与 EXIF 处理。
- 可演示同步 resize；约 224×224 是起点建议，不是已确定的最终配置。
- 对 binary label mask 使用 nearest-neighbour 插值。
- 图像和 mask 的几何变换必须共享同一组参数；颜色变换只作用于图像。
- 展示前后结果，确认没有破坏 mask 二值性和对应关系。
- 最终 resize/padding/crop、augmentation、normalisation 和 class weighting 留待方法确定。
- 若计算数据相关统计量，只使用相应 assignment 的 known training 图片；不得用 validation/test 拟合。

### 10. 阶段总结与交接

Notebook 末尾输出：

- 已验证的数据版本及实际数量。
- 两套 assignment 的角色与分区统计。
- 异常、重复与排除项的状态。
- 生成文件的位置。
- 实际运行时间及环境信息。
- 尚未确定的事项：团队分工、方法、最终 transforms、训练超参数、随机种子等。
- 一份基于实际协助内容的 AI-use 工作记录，供后续正式披露使用；不要伪造工具版本或检查过程。

## 6. 建议函数划分

以下仅是接口建议，不要在尚未查看数据时写死字段和目录：

- `inspect_dataset_layout(data_root)`
- `load_metadata(data_root)`
- `build_image_index(metadata, selected_hosts)`
- `audit_images(index)`
- `audit_masks(index)`
- `find_exact_duplicates(index)`
- `build_class_roles(class_table, assignment)`
- `build_manifest(index, roles, exclusions)`
- `validate_manifest(manifest)`
- `plot_training_examples(manifest, seed)`

这些函数可以先写在同一个 Notebook 中。不必为第一天任务建立复杂 Python package。

## 7. 必须执行的有效性检查

使用 Notebook 内的有意义 assertions 或检查报告，重点验证协议，不必另建庞大测试工程：

- [ ] 数据来源与版本明确；checksum 核验状态真实。
- [ ] 筛选后只含九种指定 host；实际类别数已核对。
- [ ] 图片 ID 唯一，路径可以由配置的数据根目录解析。
- [ ] 原始分区没有被随机重分或迁移。
- [ ] 每个类别在每套 assignment 中恰有一个 role。
- [ ] Citrus/potato 角色及 apple 示例符合要求。
- [ ] 每个训练样本都是 known + 原始 training。
- [ ] Development unknown 只进入 validation；test unknown 只进入 test。
- [ ] 所有 included 样本通过适用的完整性检查。
- [ ] 训练 mask 权限只适用于 known training。
- [ ] 跨分区精确重复已处理，或明确阻止冻结。
- [ ] 所有排除有原因，对后续所有方法一致。
- [ ] Known class ID 连续，未知类别不占用 known 输出 ID。
- [ ] 在排除后检查 known 类的训练支持数及各评估组大小；空类/空组被明确报告。
- [ ] Notebook 在干净 kernel 中按顺序运行，无隐藏状态依赖。
- [ ] 再运行不会重复下载、重复追加记录或修改原始数据。
- [ ] 真实运行输出已保存；没有执行的内容清楚标注。

## 8. GitHub 准备与上传边界

当前先把 Notebook 和配套小型文件准备为可提交状态。用户之后提供或确认目标仓库与分支并要求上传时，再执行对应 Git 操作；不要推断 FYP 等其他仓库就是本次目标。

建议 `.gitignore` 至少覆盖：

```gitignore
data/raw/
checkpoints/
.ipynb_checkpoints/
__pycache__/
.venv/
.env
.env.*
!.env.example
*.zip
*.pt
*.pth
```

根据实际目录调整，保留已有规则。不要写过宽规则误排除 CSV、JSON、Notebook 或需要提交的结果表。

提交准备要求：

- 不提交原始数据集、下载压缩包、凭据或模型大权重。
- Notebook 保留必要的统计输出与少量示例，避免嵌入大量高分辨率图片导致文件膨胀；不一概清空课程要求的输出。
- README 说明数据来源、下载方式、目录配置、依赖安装、运行顺序和目前完成范围。
- 只记录实际验证过的依赖版本，不编造版本锁定。
- 可提交完整小型 manifest、类别映射与审计表，使队友使用一致协议。
- 后续确需发布额外数据或大量原图示例时，先核对数据使用与再分发条款。
- 上传前检查 `git status`、差异和待提交文件大小；不覆盖队友的未提交工作，不 force-push。
- GitHub 上传不是本 Markdown 生成阶段已经完成的事项，不在交付说明中声称已上传。

## 9. 完成标准与最终反馈

第一天完成后，团队应能用一个 Notebook：定位正确版本数据，重新生成九宿主索引、数据审计、A/B 类别映射、实验清单和训练样本展示。

Codex 最终向用户反馈：

1. 修改了哪些文件及用途。
2. 实际运行了哪些检查、结果如何。
3. 实际数据量与异常摘要，不虚构数值。
4. Manifest 是 draft 还是 frozen，原因是什么。
5. 尚未执行的步骤或需团队确定的问题。
6. 下一步可以基于这些产物开始的工作。

若今天时间只有两小时，优先完成：版本明确的下载、数据格式探索、九宿主数量统计及少量合规训练样本展示。其余工作标记待完成，不用省略检查来宣称任务全部通过。

## 10. 用户可直接复制的启动指令

> 请阅读 `AIP304_CW2_Option2_Day1_Codex.md`，按其中要求完成第一天数据准备。先检查当前仓库已有文件与任务书，在同一个 `AIP304_CW2_Option2.ipynb` 中开展工作，完成数据获取、审计、Assignment A/B 映射和 manifest。不要开始模型训练。保存真实输出，给出检查结果与待决项，并准备好后续 GitHub 提交需要的 README、依赖说明和忽略规则。如果已有 Notebook 或目录约定，优先沿用。
