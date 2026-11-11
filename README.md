# AIP304 CW2 — Option 2, Day 1

主 Notebook 为 `CW2.ipynb`。本阶段只完成数据准备，不训练模型或计算测试性能。

## 运行

1. 在此目录启动 Jupyter，使用 Python 3.10+：`python -m pip install -r requirements.txt`。
2. 从上到下运行 `CW2.ipynb`。首次会下载约 1.1 GB 的 PlantSeg v7；CPU 足够。
3. 也可从 https://zenodo.org/records/17719108 下载 `plantseg.zip` 放到 `data/raw/`。官方 MD5：`9358a66dff88cdd15c4fe009763c40a3`。
4. Colab/Kaggle 可用环境变量 `CW2_PROJECT_ROOT` 指向可写输出目录，`PLANTSEG_DATA_ROOT` 指向含 Metadata.csv/images/annotations 的数据目录。保留压缩包可核验手动目录；未核验来源的数据不能冻结。
5. 可将 `DOWNLOAD=False` 用于离线运行。已校验压缩包会复用，原始文件不改写。

Notebook 所有处理函数都在单一文件内，无本地私有绝对路径。安装依赖与数据代码分开；实际运行版本记录于 `artifacts/metadata/environment.json`。

## 产物

- `artifacts/metadata/`：版本/API记录、环境、九宿主索引、原始到规范化标签映射、执行总结。
- `artifacts/audit/`：每宿主/类别原始分区计数、文件/mask异常、字节/像素重复及跨分区重复、排除作用范围。
- `artifacts/splits/`：A/B类别角色、独立known ID、每图manifest、各组支持数、协议及内容哈希。
- `artifacts/figures/`：尺寸统计、A/B共同known-training展示、同步resize演示。

## 使用边界

以 `split_protocol.json` 和 `execution_summary.json` 为真实运行状态依据。默认 draft；未解决跨分区重复、异常、空组和团队复核会阻止冻结。近重复未检查，不声称完全无泄漏。读取实验样本必须同时筛选 `included=True` 和对应 `experimental_split`。mask仅允许included known training使用；出现训练mask异常时两种比较方法使用相同排除清单。

保留原始约70/10/20分区，不重分或跨区移动图片。验证/测试预测只用RGB；不得输入真实mask、host元数据或含类别信息的文件名。原始数据和模型权重不入Git。数据集记录许可CC BY-NC 4.0，图片来源与许可保存在索引；对外发布示例前核对相关条款。

## 下一步

复核draft问题并冻结两套协议；填写成员信息；确定假设及容忍度、方法与匹配baseline、最终transforms和训练种子。尚未训练、调参或计算测试指标，也未上传GitHub。

## 来源与AI协助

课程任务书：`AIP304 CW2 Task Sheet.pdf`，页2、4–8。
数据：https://zenodo.org/records/17719108 （v7，2025-11-25）。
数据说明：https://doi.org/10.1038/s41597-025-06513-4 。
Mask编码参考：https://github.com/tqwei05/PlantSeg/blob/main/mmseg/datasets/plantseg115.py 。
Codex协助检查来源、编写数据准备代码及运行校验；具体结果见Notebook输出与执行总结。成员应复核并理解实现；不编造工具版本、成员贡献或未运行实验。

## 本次实际验证（2026-11-11）

已完成两次干净 kernel 顺序运行；九宿主索引和全部 A/B CSV 在两次运行中 SHA-256 一致。官方 MD5 核验通过。九宿主共 4,349 张图片、43 类，0 张不可读图片，10 条 mask/EXIF 异常，跨分区精确重复涉及 151 张图片。两套 manifest 均为 draft；下一步需复核重复与 mask 异常，再冻结协议。代码层面额外检查了 A/B 全部角色/分区组合、训练 mask 排除范围、安全解压、原始文件不覆盖及跨分区重复检测。
