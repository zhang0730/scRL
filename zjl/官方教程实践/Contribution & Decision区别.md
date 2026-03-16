

**`adata.obs['Ery']` 和 `adata.obs['Ery_decision']` 虽然都与“红系命运”相关，但它们代表完全不同的生物学概念和计算逻辑**。下面详细解释它们的区别：

------

### **✅ 一、核心区别总结**

表格

| 特征                   | `Ery`                                                        | `Ery_decision`                                               |
| :--------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **含义**               | **实际贡献 / 分化程度**（*Contribution*）                    | **命运决策倾向 / 潜能**（*Decision*）                        |
| **计算方式**           | 基于 **到达终末状态的长期累积奖励**（value function for *contribution*） | 基于 **做出命运选择的早期信号**（value function for *decision*） |
| **时间特性**           | **回顾性 / 结果导向**：反映细胞“已经走得多远”                | **前瞻性 / 预测性**：反映细胞“有多可能走向红系”              |
| **依赖的 reward 设置** | `mode='Contribution'` → 奖励只在终末细胞（Ery_1/2, Mega）发放 | 默认（或 `mode='Decision'`）→ 奖励在**路径早期**发放，强调“决定点” |
| **数值分布**           | 在终末红系细胞中最高，在 HSC 中接近 0                        | 在 HSC/MPP 中就已升高，早于实际分化                          |

------



### **🔍 二、从代码看差异**

#### **1.** **`Ery` \**的生成（Contribution 模式）\****

python

```
gres = lineage_rewards(gres, [...], [...], mode='Contribution')
t_ery = trainer('ActorCritic', gres, reward_mode='Contribution', ...)
```

- ```
  mode='Contribution'
  ```

   意味着：

  - 只有当智能体（细胞）**真正到达目标状态**（如 `Ery_1`, `Mega`）时，才获得高奖励。
  - Critic 学习的是：**从当前状态出发，最终能“贡献”多少到终末红系细胞**。

- 因此，`Ery` 值 ≈ **该细胞在其分化轨迹上对红系终末群体的“归宿概率”或“通量”**。

- 类似于 **CellRank 的 "terminal state probability"**。

> 📌 **`Ery` 高 = 这个细胞几乎注定要变成红细胞/巨核细胞**

------

#### **2.** **`Ery_decision` \**的生成（默认 Decision 模式）\****

python

```
gres = lineage_rewards(gres, [...], [...])  # 没有指定 mode → 默认是 Decision
# 或隐含使用了早期奖励策略
```

- 在 Decision模式下：
  - 奖励可能在**靠近起点的分支点**就发放（例如，一旦细胞“决定”走向红系而非髓系，就给奖励）。
  - Critic 学习的是：**当前状态是否处于“红系决策路径”上**，即使还没分化。
- 因此，`Ery_decision` 反映的是 **早期命运承诺（fate commitment）的强度**。

> 📌 **`Ery_decision` 高 = 这个细胞刚刚“下定决心”要走红系，即使它现在看起来还是 HSC**