# Decision Tree

决策树是从根节点一步步走到叶子节点的过程。

所有的数据最终都会落在叶子节点，既可以做分类也可以做回归。

- 训练阶段：从给定的训练集构建出来一棵树，从根节点开始特征选择，如何进行特征切分。
- 测试阶段：根据构造出来的树模型从上到下去走一遍就好了
- 一旦构造好了决策树，那么分类或者预测任务就很简单了，只需要走一遍就可以了，那么难点就在于如何构造出来一棵树。

目标：选择最好的特征作为根节点。

---

# PA3

## A->B vs B->A

**A->B**:在一轮轮的迭代中，寻找到损失函数极小值点。此时各方向梯度为0（驻点）。

输入 → 前向传播 → 预测值
                      ↓
                损失函数 L
                      ↓
                反向传播（求梯度）
                      ↓
                优化器更新权重 w
                      ↓
                下一轮迭代 ↺

**B->A**:

优化器是通过迭代更新模型参数、以寻找损失函数极小值的方法。训练时先经前向传播得到预测值，再由反向传播计算损失对各参数的梯度，最后由优化器依据梯度调整权重。损失函数衡量的是模型预测值与真实标签之间的误差。


**反向传播**： ***链式法则的核心思想是将复杂函数的导数分解成简单函数的导数的乘积，逐层计算梯度。***

反向传播 → 求 $\frac{\partial L}{\partial w}$（梯度）

**优化器**是通过迭代更新模型参数来**最小化损失函数**的算法。

优化器 → 利用梯度更新 $w \leftarrow w - \eta \nabla L$


| 优化器 | 更新规则 | 特点 |
|---|---|---|
| SGD | $w \leftarrow w - \eta \nabla L$ | 简单，易震荡 |
| Momentum | 加入历史梯度方向 | 加速收敛，减少震荡 |
| Adam | 自适应学习率 + 动量 | 最常用，收敛稳定 |
| RMSProp | 自适应学习率 | 适合非平稳目标 |
| AdamW | Adam + 权重衰减(l2) | 更好的正则化效果 |


- Loss Function

> 损失函数衡量的是**模型预测值**与**真实标签**之间的误差，**与权重本身无直接关系**（除非加入正则化项）。
> 加入正则化项，则意味着损失函数不仅考虑预测误差，还会惩罚过大的权重值，以防止过拟合。

$$L = \underbrace{f(\hat{y},\ y)}_{\text{预测误差}} + \underbrace{\lambda \|w\|^2}_{\text{正则化项（可选）}}$$

常见损失函数：

| 任务类型 | 损失函数 | 公式 |
|---|---|---|
| 回归 | MSE | $\frac{1}{n}\sum(\hat{y}_i - y_i)^2$ |
| 二分类 | Binary Cross-Entropy | $-[y\log\hat{y} + (1-y)\log(1-\hat{y})]$ |
| 多分类 | Categorical Cross-Entropy | $-\sum y_k \log \hat{y}_k$ |


## implementation
加载预训练的 `distilbert-base-uncased`，并通过 `num_labels=3` 告诉它输出三分类——HuggingFace 会自动把原来的二分类头替换为三分类头。模型随后被移到 GPU（如果可用）。

优化器选用 `AdamW`，学习率 `2e-5` 是微调 BERT 系列模型的经验默认值。学习率调度器采用线性衰减（`linear`），从初始值线性下降到 0，防止后期训练时学习率过大导致模型在已收敛的参数上震荡。


每个 epoch 分两个阶段：

训练阶段调用 `model.train()` 开启 dropout 等正则化机制，对每个 batch 执行前向传播（`model(**batch)`）→ 计算交叉熵损失 → 反向传播（`loss.backward()`）→ 参数更新（`optimizer.step()`）→ 调度器步进（`lr_scheduler.step()`）→ 梯度清零（`optimizer.zero_grad()`）。每步结束后在进度条上打印当前 loss，epoch 结束后打印平均 loss。

评估阶段调用 `model.eval()` 关闭 dropout，并用 `torch.no_grad()` 跳过梯度计算以节省显存。对每个 batch 取 logits 最大值的索引作为预测类别，累计正确数与总数，最终计算 accuracy 并打印。

``` python
for epoch in range(num_epochs):
    model.train()
    total_loss = 0
    progress = tqdm(train_dataloader, desc=f"Epoch {epoch+1}/{num_epochs} [train]")
    for batch in progress:
        batch = {k: v.to(device) for k, v in batch.items()}
        outputs = model(**batch)
        loss = outputs.loss
        loss.backward()
        optimizer.step()
        lr_scheduler.step()
        optimizer.zero_grad()
        total_loss += loss.item()
        progress.set_postfix(loss=f"{loss.item():.4f}")

    avg_loss = total_loss / len(train_dataloader)
    print(f"Epoch {epoch+1} — avg train loss: {avg_loss:.4f}")

    model.eval()
    correct, total = 0, 0
    with torch.no_grad():
        for batch in tqdm(eval_dataloader, desc=f"Epoch {epoch+1}/{num_epochs} [eval]"):
            batch = {k: v.to(device) for k, v in batch.items()}
            outputs = model(**batch)
            preds = outputs.logits.argmax(dim=-1)
            correct += (preds == batch["labels"]).sum().item()
            total   += batch["labels"].size(0)

    accuracy = correct / total
    print(f"Epoch {epoch+1} — eval accuracy: {accuracy:.4f}\n")

```










----
训练网络是为了达到？局部最小值。（神经网络的损失函数是非凸函数）finding minima



寻找全局最小值的原因：为了得到结果，诸如检测/分类等，神经网络对错误结果的处理就是



-----
C (正则化系数的倒数)
C 值控制着模型的正则化强度（防止过拟合的机制）。它的实际含义是正则化惩罚项 λ 的倒数（即 C=1/λ）。

C 值较小（如 0.1, 0.01）：正则化极强。模型会变得更“简单保守”，不会去死记硬背训练集里的特定样本。对于众包数据（包含标注错误/噪声），推荐调小 C 值，防止模型跟着错误的标签跑。
C 值较大（如 10, 100）：正则化较弱。模型会尽全力去拟合训练集中的每一个样本。如果训练数据非常纯净准确，可以调大；但数据有噪声时极易过拟合。