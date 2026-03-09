# Task2 改进说明（任务2_Revised2Code_0306_modified）

## 目标
针对任务2中的两个瓶颈进行修正：
- ViT-B/16（不加载预训练）准确率偏低（约 72%）。
- D2NN 12层 + 全连接头准确率卡在约 60%。

## 关键修正点
1. **ViT 训练策略增强**
   - 数据增强改为：`RandomCrop + HorizontalFlip + RandAugment + RandomErasing`。
   - 训练技巧加入：`Mixup + LabelSmoothing + AdamW(分组权重衰减) + CosineAnnealing + AMP + 梯度裁剪`。
   - 训练轮数与学习率重新设定：`80 epochs`, `lr=3e-4`。

2. **D2NN 物理一致性与可训练性修正**
   - 输入在 D2NN 前向中先做**反归一化**回 `[0,1]` 强度域，再 `sqrt` 转振幅（更符合光场建模）。
   - 输出光强加 `log1p` 压缩动态范围，减轻高亮区域对分类头的支配。
   - 相位参数采用小随机初始化，避免全零初始化带来的对称性训练停滞。
   - 两阶段训练保留并强化：先训练 phase+fc，再全参数微调（第二阶段启用 mixup）。

3. **保持公平对比条件**
   - ViT 与 D2NN 都使用同结构 `SharedFCHead`（LayerNorm + MLP），满足“全连接部分一致”的对比要求。

## 你可以重点汇报的结论（方法论）
- 原 72% 的 ViT（from scratch）通常不是模型上限问题，更多是训练策略不充分。
- 原 60% 的 D2NN 很可能包含“输入不物理一致 + 光强动态范围过大 + 优化不稳定”的复合问题。
- 在不破坏任务2“同头公平对比”约束下，上述修改可显著提高上限并改善收敛稳定性。

## 文件说明
- 主实验 notebook：`task2_Revised2Code_0306_modified.ipynb`
- 本说明文档：`task2_revised_explanation.md`
