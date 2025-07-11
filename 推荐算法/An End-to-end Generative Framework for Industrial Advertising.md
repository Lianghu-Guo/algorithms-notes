### 论文核心概述
《EGA-V2: 面向工业广告的端到端生成框架》由美团团队提出，针对传统工业广告系统多阶段级联架构的局限性，首次提出将用户兴趣建模、广告点位（POI）与创意生成、广告分配及支付优化整合到单一生成模型中的端到端解决方案。该框架通过层次化标记、多标记预测、排列感知奖励模型及基于拍卖的偏好对齐机制，实现了广告系统的全局优化，在工业数据集上显著提升了平台 revenue、用户体验及广告主激励兼容性。
### 传统广告系统的痛点与生成式方案的突破
1. 传统多阶段级联架构的局限
- 早期过滤导致次优：召回、排序等阶段逐层筛选候选广告，高潜力广告可能在早期被丢弃，无法实现全局最优（如上游过滤的广告无法在后续阶段恢复）。
- 模块割裂决策分散：各阶段独立优化（如召回、排序、创意选择），缺乏全局协同，难以平衡用户体验与商业目标。
2. 生成式推荐的不足与广告场景的特殊需求
- 现有生成模型的缺陷：虽能端到端生成推荐序列，但未考虑广告特有的竞价规则、创意选择、广告位分配及支付计算等业务约束。
- 广告系统的核心需求：需满足激励兼容性（IC）、个体理性（IR），并同时优化平台 revenue、用户点击行为及广告主投放效益。
### EGA-V2 的核心架构与技术创新
1. 统一生成模型：从用户兴趣到广告序列的端到端生成
- 层次化标记与多模态生成：
  - 采用残差量化变分自编码器（RQ-VAE）将用户行为和广告特征编码为语义标记（Token），每个广告表示为 “POI 标记 + 创意标记” 对，捕捉高层意图与视觉细节。
  - 通过编码器 - 解码器架构，基于多标记预测（MTP）同时生成 POI 推荐序列与创意内容，实现联合优化。
- 架构对比：传统级联系统（图 1a）需多阶段筛选，而 EGA-V2（图 1b）直接生成最终广告序列，避免早期信息丢失。
2. 排列感知奖励模型与 token 级竞价机制
- 奖励模型捕捉全局依赖：通过自注意力机制处理生成的标记序列，预测点击率（pCTR）、转化率（pCVR）等指标，引导模型生成符合用户兴趣和商业价值的序列。
- token 级竞价与生成式分配：
  - 聚合与标记关联的所有广告竞价，以 “最大竞价” 作为标记权重（式 21），动态调整广告与自然内容的曝光比例。
  - 基于软 max 归一化生成候选序列，通过波束搜索筛选高价值序列，平衡多样性与优化目标。
3. 基于拍卖的偏好对齐与支付网络
- 解耦分配与支付：分配在标记级进行，而支付计算基于 POI 级，通过可微的事后遗憾最小化机制（Ex-post Regret）近似满足激励兼容性。
- 支付网络设计：输入广告表示、竞价分布及分配概率，通过 MLP 计算支付率，确保支付≤竞价（IR 约束），并通过拉格朗日优化最小化广告主遗憾（IC 约束）。
4. 多阶段训练策略
- 兴趣预训练：基于用户历史行为（含广告与自然内容），通过交叉熵损失训练模型预测下一个 POI 和创意标记，捕捉用户兴趣。
- 拍卖后训练：
- 奖励模型：基于真实点击、转化数据优化 pCTR/pCVR 预测。
- 生成式分配：通过策略梯度最大化预期 revenue，优化广告序列生成。
- 支付网络：通过迭代拉格朗日方法平衡 revenue 与 IC 约束。
### 实验验证与关键结论
1. 数据集与评估指标
- 数据集：美团 2024 年 9 月至 2025 年 4 月的 2 亿次用户请求，包含 1000 万 + 广告，分为预训练（200 天）、偏好对齐（10% 抽样）、测试（14 天）。
- 指标：
  - 平台 revenue（RPM）、用户点击率（CTR）。
  - 激励兼容性（IC Metric，Ψ）：衡量广告主通过虚假竞价获得的效用增益，值越低越好。
2. 对比基线与性能提升
- 基线：
  - MCA（多阶段级联架构）：采用 Tiger 召回、HSTU 排序等传统模块。
  - GR（生成式推荐）：整合 OneRec 生成与 GSP 支付。
- 结果（表 2）：
  - EGA-V2 的 RPM 达 230.41，较 GR 提升 11.4%，较 MCA 提升 19.7%。
  - CTR-poi 与 CTR-img 分别提升 6.3% 和 6.8%，IC 遗憾（Ψ）降至 2.7%，显著低于 GR 的 8.4% 和 MCA 的 3.6%。
3. 消融实验：各组件的有效性
- MTP 模块：移除后 RPM 下降 2.1%，CTR 下降 3.1%~3.6%，证明 POI 与创意联合建模的重要性。
- 多阶段训练：单阶段训练导致 RPM 下降 5.3%，说明预训练捕捉用户兴趣、后训练适配广告约束的必要性。
- token 级竞价：将 “最大竞价” 改为 “平均竞价” 后，IC 遗憾升至 4.1%，影响分配效率。
- 支付网络：替换为 GSP 支付后，IC 遗憾飙升至 8.2%，验证支付网络对激励兼容性的关键作用。
### 工业应用价值与未来方向
1. 核心贡献
- 技术突破：首个端到端生成式广告框架，统一多阶段决策，解决传统级联架构的全局次优问题。
- 业务落地：通过多阶段训练和拍卖机制，平衡用户兴趣与商业目标，适用于真实广告场景。
2. 未来研究方向
- 规模化扩展：探索模型在更大规模数据集和更多广告场景中的应用。
- 可解释性增强：提升生成式广告决策的透明度，便于广告主理解投放效果。
在线 A/B 测试：将离线验证的效果延伸至线上，进一步优化实时广告分配策略。
### 总结
EGA-V2 通过生成式建模与经济机制设计的结合，为工业广告系统提供了全新的技术范式。其核心价值在于打破传统级联架构的桎梏，通过统一模型实现从用户兴趣到广告分配的全局优化，同时满足广告业务的严格约束。该框架在离线实验中展现的显著优势，为下一代广告系统的发展指明了方向。