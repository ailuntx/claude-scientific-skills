# COBRApy 综合工作流程

本文档为代谢建模中常见的 COBRApy 任务提供了详细的逐步工作流程。

## 工作流程1：完整的基因敲除研究与可视化

本工作流程演示如何执行全面的基因敲除研究并可视化结果。

```python
import pandas as pd
import matplotlib.pyplot as plt
from cobra.io import load_model
from cobra.flux_analysis import single_gene_deletion, double_gene_deletion

# 步骤1：加载模型
model = load_model("ecoli")
print(f"Loaded model: {model.id}")
print(f"Model contains {len(model.reactions)} reactions, {len(model.metabolites)} metabolites, {len(model.genes)} genes")

# 步骤2：获取基准生长速率
baseline = model.slim_optimize()
print(f"Baseline growth rate: {baseline:.3f} /h")

# 步骤3：执行单基因敲除
print("Performing single gene deletions...")
single_results = single_gene_deletion(model)

# 步骤4：按影响程度分类基因
essential_genes = single_results[single_results["growth"] < 0.01]
severely_impaired = single_results[(single_results["growth"] >= 0.01) &
                                   (single_results["growth"] < 0.5 * baseline)]
moderately_impaired = single_results[(single_results["growth"] >= 0.5 * baseline) &
                                     (single_results["growth"] < 0.9 * baseline)]
neutral_genes = single_results[single_results["growth"] >= 0.9 * baseline]

print(f"\nSingle Deletion Results:")
print(f"  Essential genes: {len(essential_genes)}")
print(f"  Severely impaired: {len(severely_impaired)}")
print(f"  Moderately impaired: {len(moderately_impaired)}")
print(f"  Neutral genes: {len(neutral_genes)}")

# 步骤5：可视化分布
fig, ax = plt.subplots(figsize=(10, 6))
single_results["growth"].hist(bins=50, ax=ax)
ax.axvline(baseline, color='r', linestyle='--', label='Baseline')
ax.set_xlabel("生长速率 (/h)")
ax.set_ylabel("基因数量")
ax.set_title("单基因敲除后生长速率分布")
ax.legend()
plt.tight_layout()
plt.savefig("single_deletion_distribution.png", dpi=300)

# 步骤6：确定双基因敲除目标
# 聚焦非必需基因寻找合成致死对
target_genes = single_results[single_results["growth"] >= 0.5 * baseline].index.tolist()
target_genes = [list(gene)[0] for gene in target_genes[:50]]  # 性能限制

print(f"\nPerforming double deletions on {len(target_genes)} genes...")
double_results = double_gene_deletion(
    model,
    gene_list1=target_genes,
    processes=4
)

# 步骤7：寻找合成致死基因对
synthetic_lethals = double_results[
    (double_results["growth"] < 0.01) &
    (single_results.loc[double_results.index.get_level_values(0)]["growth"].values >= 0.5 * baseline) &
    (single_results.loc[double_results.index.get_level_values(1)]["growth"].values >= 0.5 * baseline)
]

print(f"Found {len(synthetic_lethals)} synthetic lethal gene pairs")
print("\nTop 10 synthetic lethal pairs:")
print(synthetic_lethals.head(10))

# 步骤8：导出结果
single_results.to_csv("single_gene_deletions.csv")
double_results.to_csv("double_gene_deletions.csv")
synthetic_lethals.to_csv("synthetic_lethals.csv")
```

## 工作流程2：培养基设计与优化

本工作流程展示如何系统设计生长培养基并寻找最小培养基组成。

```python
from cobra.io import load_model
from cobra.medium import minimal_medium
import pandas as pd

# 步骤1：加载模型并检查当前培养基
model = load_model("ecoli")
current_medium = model.medium
print("当前培养基组成:")
for exchange, bound in current_medium.items():
    metabolite_id = exchange.replace("EX_", "").replace("_e", "")
    print(f"  {metabolite_id}: {bound:.2f} mmol/gDW/h")

# 步骤2：获取基准生长速率
baseline_growth = model.slim_optimize()
print(f"\nBaseline growth rate: {baseline_growth:.3f} /h")

# 步骤3：计算不同生长目标的最小培养基
growth_targets = [0.25, 0.5, 0.75, 1.0]
minimal_media = {}

for fraction in growth_targets:
    target_growth = baseline_growth * fraction
    print(f"\nCalculating minimal medium for {fraction*100:.0f}% growth ({target_growth:.3f} /h)...")

    min_medium = minimal_medium(
        model,
        target_growth,
        minimize_components=True,
        open_exchanges=True
    )

    minimal_media[fraction] = min_medium
    print(f"  必需组分: {len(min_medium)}")
    print(f"  组分: {list(min_medium.index)}")

# 步骤4：比较培养基组成
media_df = pd.DataFrame(minimal_media).fillna(0)
media_df.to_csv("minimal_media_comparison.csv")

# 步骤5：测试有氧与厌氧条件
print("\n--- 有氧与厌氧条件比较 ---")

# 有氧条件
model_aerobic = model.copy()
aerobic_growth = model_aerobic.slim_optimize()
aerobic_medium = minimal_medium(model_aerobic, aerobic_growth * 0.9, minimize_components=True)

# 厌氧条件
model_anaerobic = model.copy()
medium_anaerobic = model_anaerobic.medium
medium_anaerobic["EX_o2_e"] = 0.0
model_anaerobic.medium = medium_anaerobic
anaerobic_growth = model_anaerobic.slim_optimize()
anaerobic_medium = minimal_medium(model_anaerobic, anaerobic_growth * 0.9, minimize_components=True)

print(f"有氧生长: {aerobic_growth:.3f} /h (需要 {len(aerobic_medium)} 种组分)")
print(f"厌氧生长: {anaerobic_growth:.3f} /h (需要 {len(anaerobic_medium)} 种组分)")

# 步骤6：识别独特需求
aerobic_only = set(aerobic_medium.index) - set(anaerobic_medium.index)
anaerobic_only = set(anaerobic_medium.index) - set(aerobic_medium.index)
shared = set(aerobic_medium.index) & set(anaerobic_medium.index)

print(f"\n共享组分: {len(shared)}")
print(f"有氧独有: {aerobic_only}")
print(f"厌氧独有: {anaerobic_only}")

# 步骤7：测试自定义培养基
print("\n--- 测试自定义培养基 ---")
custom_medium = {
    "EX_glc__D_e": 10.0,  # 葡萄糖
    "EX_o2_e": 20.0,       # 氧气
    "EX_nh4_e": 5.0,       # 铵盐
    "EX_pi_e": 5.0,        # 磷酸盐
    "EX_so4_e": 1.0,       # 硫酸盐
}

with model:
    model.medium = custom_medium
    custom_growth = model.optimize().objective_value
    print(f"自定义培养基生长速率: {custom_growth:.3f} /h")

    # 检测限制性营养物
    for exchange in custom_medium:
        with model:
            # 双倍摄取速率
            medium_test = model.medium
            medium_test[exchange] *= 2
            model.medium = medium_test
            test_growth = model.optimize().objective_value
            improvement = (test_growth - custom_growth) / custom_growth * 100
            if improvement > 1:
                print(f"  {exchange}: 加倍后生长提升 +{improvement:.1f}% (限制性)")
```

## 工作流程3：基于采样的通量空间探索

本工作流程演示使用通量变异性分析（FVA）和采样进行全面的通量空间分析。

```python
from cobra.io import load_model
from cobra.flux_analysis import flux_variability_analysis
from cobra.sampling import sample
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# 步骤1：加载模型
model = load_model("ecoli")
baseline = model.slim_optimize()
print(f"基准生长速率: {baseline:.3f} /h")

# 步骤2：在最优生长下执行FVA
print("\n在最优生长下执行通量变异性分析...")
fva_optimal = flux_variability_analysis(model, fraction_of_optimum=1.0)

# 步骤3：识别具有灵活性的反应
fva_optimal["range"] = fva_optimal["maximum"] - fva_optimal["minimum"]
fva_optimal["relative_range"] = fva_optimal["range"] / (fva_optimal["maximum"].abs() + 1e-9)

flexible_reactions = fva_optimal[fva_optimal["range"] > 1.0].sort_values("range", ascending=False)
print(f"\n发现 {len(flexible_reactions)} 个灵活性>1.0 mmol/gDW/h的反应")
print("\n前10个最灵活反应:")
print(flexible_reactions.head(10)[["minimum", "maximum", "range"]])

# 步骤4：在次优生长下执行FVA（90%）
print("\n在90%最优生长下执行通量变异性分析...")
fva_suboptimal = flux_variability_analysis(model, fraction_of_optimum=0.9)
fva_suboptimal["range"] = fva_suboptimal["maximum"] - fva_suboptimal["minimum"]

# 步骤5：比较不同最优性水平的灵活性
comparison = pd.DataFrame({
    "range_100": fva_optimal["range"],
    "range_90": fva_suboptimal["range"]
})
comparison["range_increase"] = comparison["range_90"] - comparison["range_100"]

print("\n次优条件下灵活性增加最大的反应:")
print(comparison.sort_values("range_increase", ascending=False).head(10))

# 步骤6：执行通量采样
print("\n执行通量采样（1000个样本）...")
samples = sample(model, n=1000, method="optgp", processes=4)

# 步骤7：分析关键反应的采样结果
key_reactions = ["PFK", "FBA", "TPI", "GAPD", "PGK", "PGM", "ENO", "PYK"]
available_key_reactions = [r for r in key_reactions if r in samples.columns]

if available_key_reactions:
    fig, axes = plt.subplots(2, 4, figsize=(16, 8))
    axes = axes.flatten()

    for idx, reaction_id in enumerate(available_key_reactions[:8]):
        ax = axes[idx]
        samples[reaction_id].hist(bins=30, ax=ax, alpha=0.7)

        # 叠加FVA边界
        fva_min = fva_optimal.loc[reaction_id, "minimum"]
        fva_max = fva_optimal.loc[reaction_id, "maximum"]
        ax.axvline(fva_min, color='r', linestyle='--', label='FVA最小值')
        ax.axvline(fva_max, color='r', linestyle='--', label='FVA最大值')

        ax.set_xlabel("通量 (mmol/gDW/h)")
        ax.set_ylabel("频率")
        ax.set_title(reaction_id)
        if idx == 0:
            ax.legend()

    plt.tight_layout()
    plt.savefig("flux_distributions.png", dpi=300)

# 步骤8：计算反应间相关性
print("\n计算通量相关性...")
correlation_matrix = samples[available_key_reactions].corr()

fig, ax = plt.subplots(figsize=(10, 8))
sns.heatmap(correlation_matrix, annot=True, fmt=".2f", cmap="coolwarm",
            center=0, ax=ax, square=True)
ax.set_title("关键糖酵解反应通量相关性")
plt.tight_layout()
plt.savefig("flux_correlations.png", dpi=300)

# 步骤9：识别反应模块（高度相关组）
print("\n高度相关反应对 (|r| > 0.9):")
for i in range(len(correlation_matrix)):
    for j in range(i+1, len(correlation_matrix)):
        corr = correlation_matrix.iloc[i, j]
        if abs(corr)

```markdown
if len(knockout_df) > 0:
    print("\n测试敲除组合...")
    top_genes = knockout_df.head(3)["gene"].tolist()

    with model:
        for gene_id in top_genes:
            model.genes.get_by_id(gene_id).knock_out()

        solution = model.optimize()
        if solution.status == "optimal":
            combined_production = solution.objective_value
            combined_growth = solution.fluxes["BIOMASS_Ecoli_core_w_GAM"]
            combined_improvement = (combined_production / max_production - 1) * 100

            print(f"\n组合敲除结果:")
            print(f"  基因: {', '.join(top_genes)}")
            print(f"  产量: {combined_production:.3f} mmol/gDW/h")
            print(f"  生长率: {combined_growth:.3f} /h")
            print(f"  提升幅度: {combined_improvement:.1f}%")

# 步骤7：分析生产菌株中的通量分布
if len(knockout_df) > 0:
    best_gene = knockout_df.iloc[0]["gene"]

    with model:
        model.genes.get_by_id(best_gene).knock_out()
        solution = model.optimize()

        # 获取活跃通路
        active_fluxes = solution.fluxes[solution.fluxes.abs() > 0.1]
        active_fluxes.to_csv(f"production_strain_fluxes_{best_gene}_knockout.csv")

        print(f"\n生产菌株中的活跃反应数: {len(active_fluxes)}")
```

## 工作流程5：模型验证与调试

本工作流程展示验证和调试代谢模型的系统方法。

```python
from cobra.io import load_model, read_sbml_model
from cobra.flux_analysis import flux_variability_analysis
import pandas as pd

# 步骤1：加载模型
model = load_model("ecoli")  # 或使用 read_sbml_model("your_model.xml")
print(f"模型: {model.id}")
print(f"反应数: {len(model.reactions)}")
print(f"代谢物数: {len(model.metabolites)}")
print(f"基因数: {len(model.genes)}")

# 步骤2：检查模型可行性
print("\n--- 可行性检查 ---")
try:
    objective_value = model.slim_optimize()
    print(f"模型可行 (目标值: {objective_value:.3f})")
except:
    print("模型不可行")
    print("排查步骤:")

    # 检查阻塞反应
    from cobra.flux_analysis import find_blocked_reactions
    blocked = find_blocked_reactions(model)
    print(f"  阻塞反应数: {len(blocked)}")
    if len(blocked) > 0:
        print(f"  前10个阻塞反应: {list(blocked)[:10]}")

    # 检查培养基
    print(f"\n  当前培养基: {model.medium}")

    # 尝试开放所有交换反应
    for reaction in model.exchanges:
        reaction.lower_bound = -1000

    try:
        objective_value = model.slim_optimize()
        print(f"\n  开放交换反应后模型可行 (目标值: {objective_value:.3f})")
        print("  问题：培养基约束过严")
    except:
        print("\n  开放交换反应后仍不可行")
        print("  问题：结构性问题（缺失反应、质量不平衡等）")

# 步骤3：检查质量和电荷平衡
print("\n--- 质量与电荷平衡检查 ---")
unbalanced_reactions = []
for reaction in model.reactions:
    try:
        balance = reaction.check_mass_balance()
        if balance:
            unbalanced_reactions.append({
                "reaction": reaction.id,
                "imbalance": balance
            })
    except:
        pass

if unbalanced_reactions:
    print(f"发现 {len(unbalanced_reactions)} 个不平衡反应:")
    for item in unbalanced_reactions[:10]:
        print(f"  {item['reaction']}: {item['imbalance']}")
else:
    print("所有反应均质量平衡")

# 步骤4：识别死端代谢物
print("\n--- 死端代谢物检查 ---")
dead_end_metabolites = []
for metabolite in model.metabolites:
    producing_reactions = [r for r in metabolite.reactions
                          if r.metabolites[metabolite] > 0]
    consuming_reactions = [r for r in metabolite.reactions
                          if r.metabolites[metabolite] < 0]

    if len(producing_reactions) == 0 or len(consuming_reactions) == 0:
        dead_end_metabolites.append({
            "metabolite": metabolite.id,
            "producers": len(producing_reactions),
            "consumers": len(consuming_reactions)
        })

if dead_end_metabolites:
    print(f"发现 {len(dead_end_metabolites)} 个死端代谢物:")
    for item in dead_end_metabolites[:10]:
        print(f"  {item['metabolite']}: {item['producers']}个生产者, {item['consumers']}个消费者")
else:
    print("未发现死端代谢物")

# 步骤5：检查重复反应
print("\n--- 重复反应检查 ---")
reaction_equations = {}
duplicates = []

for reaction in model.reactions:
    equation = reaction.build_reaction_string()
    if equation in reaction_equations:
        duplicates.append({
            "reaction1": reaction_equations[equation],
            "reaction2": reaction.id,
            "equation": equation
        })
    else:
        reaction_equations[equation] = reaction.id

if duplicates:
    print(f"发现 {len(duplicates)} 对重复反应:")
    for item in duplicates[:10]:
        print(f"  {item['reaction1']} == {item['reaction2']}")
else:
    print("未发现重复反应")

# 步骤6：识别孤立基因
print("\n--- 孤立基因检查 ---")
orphan_genes = [gene for gene in model.genes if len(gene.reactions) == 0]

if orphan_genes:
    print(f"发现 {len(orphan_genes)} 个孤立基因（未关联任何反应）:")
    print(f"  前10个: {[g.id for g in orphan_genes[:10]]}")
else:
    print("未发现孤立基因")

# 步骤7：检查热力学不可行循环
print("\n--- 热力学循环检查 ---")
fva_loopless = flux_variability_analysis(model, loopless=True)
fva_standard = flux_variability_analysis(model)

loop_reactions = []
for reaction_id in fva_standard.index:
    standard_range = fva_standard.loc[reaction_id, "maximum"] - fva_standard.loc[reaction_id, "minimum"]
    loopless_range = fva_loopless.loc[reaction_id, "maximum"] - fva_loopless.loc[reaction_id, "minimum"]

    if standard_range > loopless_range + 0.1:
        loop_reactions.append({
            "reaction": reaction_id,
            "standard_range": standard_range,
            "loopless_range": loopless_range
        })

if loop_reactions:
    print(f"发现 {len(loop_reactions)} 个可能参与循环的反应:")
    loop_df = pd.DataFrame(loop_reactions).sort_values("standard_range", ascending=False)
    print(loop_df.head(10))
else:
    print("未检测到热力学不可行循环")

# 步骤8：生成验证报告
print("\n--- 生成验证报告 ---")
validation_report = {
    "model_id": model.id,
    "feasible": objective_value if 'objective_value' in locals() else None,
    "n_reactions": len(model.reactions),
    "n_metabolites": len(model.metabolites),
    "n_genes": len(model.genes),
    "n_unbalanced": len(unbalanced_reactions),
    "n_dead_ends": len(dead_end_metabolites),
    "n_duplicates": len(duplicates),
    "n_orphan_genes": len(orphan_genes),
    "n_loop_reactions": len(loop_reactions)
}

validation_df = pd.DataFrame([validation_report])
validation_df.to_csv("model_validation_report.csv", index=False)
print("验证报告已保存至 model_validation_report.csv")
```

这些工作流程为常见COBRApy任务提供了全面模板。请根据具体研究问题和模型进行调整。
