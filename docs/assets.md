# External asset requirements / 外部资产

The decoder requires compatible caller-supplied model weights and LSF codebooks.
Some modes additionally require MCR codebooks or an HOA basis. None are included in
the repository or Cargo packages. The software does not download or extract them.
Only use assets you are entitled to use and distribute separately if permitted.

| Asset | API | Requirement |
|---|---|---|
| Neural model | `HyperModel::parse` | Required; bounded little-endian model records, at most 32 MiB |
| LSF | `LsfCodebooks::parse` | Required; 43,976 bytes, magic `AV3LSF01` |
| MCR | `McrCodebooks::parse`, `Decoder::with_mcr` | Additional 9,224-byte angle codebook, magic `AV3MCR01`, for low-rate MCR stereo |
| HOA | `HoaBasis::parse`, `Decoder::with_hoa` | Additional 85,960-byte basis, magic `AV3HOA01`, for reduced-transport third-order HOA |

All parsers validate dimensions, lengths and finite numeric values. Assets from a
different encoder/model configuration are not interchangeable merely because their
file names match. The normal decoder output requires no HRIR or room-processing asset.

## Binary formats

The following describes the supported binary layout for asset providers. It is not
an asset acquisition or extraction procedure.

仓库不内置训练权重、LSF/MCR 码本或 HOA 基系数。调用方需提供已经解封装的 little-endian 模型记录和码本资产；运行时不自动识别或解包来源格式。

MODEL.bin 依次包含 base/context 编码器、解码器、权重、偏置、归一化参数、中位数和概率表。支持 SAME padding、stride 1/2、ReLU/Linear/Sigmoid/Tanh/GDN/IGDN。模型最多 32 MiB，单张量及特征最多 1,048,576 元素，单次网络工作量最多 64 Mi；负尺寸、NaN/Inf、非法归一化参数、尺度或 CDF、尾随数据均拒绝。当前本地模型为 79,930 字节、14 层，base 潜变量 64×16、context 16×16。

LSF.bin 固定为 43,976 字节：8 字节 ASCII `AV3LSF01`，随后是 f32 little-endian 的 mean[16] 和以下十二个连续表。每表按 entry/vector 排列，数值必须有限且绝对值不超过 48,000。

| 顺序 | 表形状 |
|---|---|
| high-rate stage 1 | 256×9，256×7 |
| high-rate stage 2 | 128×3，128×3，64×3，32×3，32×4 |
| low-rate stage 1 | 256×9，256×7 |
| low-rate stage 2 | 128×5，128×4，64×7 |

MCR.bin 固定为 9,224 字节：8 字节 ASCII `AV3MCR01`，随后为 long 512×3、short 256×3 的 f32 little-endian 角度，按 entry/vector 排列。角度须有限且绝对值不超过 π。通过 `Decoder::new(model, lsf).with_mcr(mcr)` 启用；缺失码本时 MCR 帧明确报错且不推进状态，其他模式不要求此资产。

HOA.bin 固定为 85,960 字节：8 字节 ASCII `AV3HOA01`，随后为 1343×16 的 f32 little-endian 基系数，按基索引/输出声道排列。系数须有限且绝对值不超过 1。通过 `Decoder::new(model, lsf).with_hoa(basis)` 启用。仅三阶 256/320/384/512/640 kbps 要求该资产，其他 HOA 配置不需要。空间基索引延迟、时域缓存与二次合成重叠历史属于 Decoder 实例，reset 一并清空。
