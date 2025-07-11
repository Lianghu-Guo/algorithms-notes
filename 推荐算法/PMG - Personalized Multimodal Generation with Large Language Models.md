### 一、研究背景与问题
#### 现有挑战
多模态生成（如图像、视频生成）已取得显著进展（如 Sora 视频生成模型），但个性化生成仍是空白。现有方法（如 Textual Inversion、DreamBooth）仅能基于少量图像微调模型风格，无法处理用户行为（如点击记录、对话历史）中的抽象偏好。
#### 应用价值
个性化多模态生成在推荐系统、广告设计、虚拟助手等场景中需求迫切。例如：
- 推荐系统中生成符合用户偏好的商品图片（如个性化服装预览）；
- 聊天工具根据用户历史使用的表情和对话内容生成专属表情；
- 电影推荐中生成融合用户偏好元素的个性化海报。
### 二、核心方法：PMG（个性化多模态生成）
PMG 的核心流程分为三步：用户偏好提取、生成条件构建、加权优化生成。
1. 用户偏好提取：从行为到语义表示
- 行为转换：将用户历史行为（点击记录、对话）中的文本、图像等多模态信息，通过 LLM（如 Llama2）和 caption 模型（如 BLIP-2）转换为自然语言摘要。例如，将用户点击的服装图片转为 “黑色卡通 T 恤、夏季棉料” 等描述。
- 双轨表示：
  - 显式关键词：通过 LLM 零样本生成偏好关键词（如 “卡通、黑色、夏季”），覆盖颜色、风格等属性；
  - 隐式嵌入：训练 “偏差校正 LLM” 生成软偏好嵌入（Soft Preference Embeddings），弥补自然语言表达的局限性（如抽象风格偏好无法用关键词完全描述）。
2. 生成条件构建：融合偏好与目标项目
- 目标项目处理：将待生成的目标项目（如 “黑色长袖衬衫”）转换为关键词（如 “衬衫、黑色、长袖”）。
- 条件融合：将用户偏好关键词、软嵌入与目标项目关键词结合，作为扩散模型（如 Stable Diffusion）或多模态 LLM 的输入 prompt。
3. 加权优化：平衡准确性与个性化
- 双指标评估：
  - 准确性分数（Accuracy Score）：生成内容与目标项目的相似度（如通过 CLIP 模型计算嵌入相似度）；
  - 偏好分数（Preference Score）：生成内容与用户偏好的匹配度。
- 加权优化：通过调整权重 α 平衡两者，公式为：
$$
z=α⋅log\ d_p + (1−α)⋅log\ d_t
​$$
其中 $d_p$是偏好分数，$d_t$ 是准确性分数，α 通常取 0.5，可根据场景调整。
### 三、实验验证：效果与优势
- 实验场景与数据集
  - 服装生成：使用 POG 数据集（2000 用户，1.6 万服装项目），生成个性化服装图片；
  - 电影海报生成：使用 MovieLens 数据集（9000 电影，600 用户），生成融合用户偏好的海报；
  - 表情生成：基于对话历史生成个性化表情（因无公开数据集，仅用关键词生成）。
- 评估指标
  - 客观指标：LPIPS（感知相似度，越低越好）、SSIM（结构相似度，越高越好）；
  - 主观评估：邀请 40 名志愿者对生成图像打分（1-3 分）。
- 关键结果
  - 个性化提升：相比无个性化基线，PMG 在 LPIPS 指标上提升 8%，且保持生成准确性；
  - 人类评估：PMG 生成的图像在电影海报和服装场景中平均得分 2.587 和 2.001，显著高于 Textual Inversion（1.952 和 1.725）和无个性化方案；
  - 消融实验：显式关键词和隐式嵌入结合效果最佳，缺少任意一项都会导致个性化程度下降（如表 2 所示）。
### 四、创新点与核心贡献
**首次将 LLM 用于个性化多模态生成**
- 突破传统方法依赖少量图像微调的限制，直接从用户行为（点击、对话）中提取偏好，适用于大规模用户场景。
**双轨偏好表示：关键词 + 嵌入**
- 显式关键词确保语义可解释性，隐式嵌入捕捉抽象偏好（如 “卡通风格” 的细微差异），两者结合提升生成准确性。
**加权优化框架**
- 通过平衡 “偏好匹配” 与 “目标相关性”，避免生成内容偏离原始项目（如不会将 “快乐表情” 生成 “哭泣猫”）。
### 五、应用场景与未来方向
- 实际应用
  - 推荐系统：生成个性化商品图片提升点击率；
  - 广告设计：根据用户兴趣定制产品海报；
  - 虚拟助手：生成符合用户习惯的表情、背景音等。
- 未来工作
  - 提升生成图像的真实感（如避免 “幻觉” 问题，确保角色、服装与真实物品一致）；
  - 扩展至视频、音频等更多模态，探索跨模态个性化生成。
### 六、总结
PMG 通过 LLM 赋能个性化多模态生成，为推荐系统、内容创作等领域提供了新范式。其核心价值在于将用户抽象偏好转化为可计算的生成条件，同时保证内容与目标项目的相关性，为 “千人千面” 的多模态内容生成奠定了基础。
