# Autonomous Optimization Arena

没有延续之前Harness Arena的想法，仅评测对harness的优化，感觉这样把这个题目做小了，尤其比如翁jiayi的RL game问题也不能将其归结到Harness优化上，刚好结合我最近写intro的想法，我们提了“AutonomousOptimization”这个概念，感觉非常合适

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NzNmNWRhMzA0Y2Q3ZmIyZWNjOGQ2M2E5MTdmOWQ3OTVfMmY0OTFjNmY3YjUwZDhlMWQ4ZDUwYWQ1ZmQ0NDAyNTJfSUQ6NzY0NDA5Njk4NjYxODU5NjU1Nl8xNzc5Nzg4NzU2OjE3Nzk4NzUxNTZfVjM)



> We formalize this class of problems as ***Autonomous Optimization \(AO\)***: given an artifact to improve, an objective, and an evaluation procedure, an agent must autonomously produce increasingly better versions without step\-level human supervision\.
> 
> 

在这个topic下，自主优化harness仅仅是其中的一个track（独立出来也方便，如果有的工作只做了harness，可以只测这个track，受众更多），把概念重点更放在agent的“Autonomous”上，比仅仅harness感觉更promising。

每个 AO task 可以形式化为：

```Plain Text
AO Task = (
  initial artifact,
  editable space,
  objective,
  evaluator,
  budget,
  held-out / final split
)
```

其中：

- **Artifact**：被优化的对象，例如 `policy\.py`、`train\.py`、`harness/`、`kernel\.py`、`rubric\.md`、`data\_synthesis\_pipeline\.py`。

- **Editable space**：agent 允许改哪些文件/模块。

- **Objective**：优化目标，例如 reward、validation loss、pass rate、speedup、judge\-human agreement。

- **Evaluator**：自动评测流程，例如 rollout、训练脚本、benchmark harness、unit tests、LLM judge。

- **Budget**：时间、API calls、GPU hours、环境交互步数、实验次数等。

- **Held\-out / final split**：防止 agent 只 overfit public dev evaluator。

[resource](https://jwolpxeehx.feishu.cn/wiki/XFHgwl12yiQwL5kUVDWc1W2enWe)

集合到Harbor框架里，初步觉得这些track可以归进来：

# Track A: Harness Engineering Optimization

优化 agent harness，比如工具调用、搜索策略、反思/投票/缓存/memory组件等，固定 backbone model，只允许优化 harness/scaffold。

- **Search Agent Family：**以Browsecomp [https://openai\.com/index/browsecomp/](https://openai.com/index/browsecomp/)为主bench，在上面sample后划分测试集/验证集，优化后在OOD的bench上（GAIA/HLE/WideSearch）上sample一些题测试harness的泛化性

- **Code Agent Family：**以Terminal Bench 2\.0 https://www\.tbench\.ai/为主bench，优化后在SWE\-Bench上测harness的泛化性，这个setting在Agentic Harness Engineering这篇工作里就是这么测的

- **Browser Agent Family：**以WebArena为主bench，在其他webbrowser上测泛化（这种bench非常多）



# Track B: Game / RL Policy Optimization

Learning Beyond Gradients（wengjiayi）里的任务，让 agent 迭代修改 heuristic policy / controller code，然后通过环境 rollout 得到 reward

- **Atari Family**：以 Atari 游戏为 task，优化 `policy\.py`，输入可以是 RAM 或 image observation，输出是离散动作。

    - 具体 task：

        - Pong

        - Breakout

        - Montezuma’s Revenge

        - Atari57 subset / full Atari57

- **MuJoCo Family**：以连续控制任务为 task，优化 controller code，输出连续动作。

    - 具体 task：

        - Ant

        - HalfCheetah

- **VizDoom Family**：以视觉 FPS\-like 环境为 task，优化基于图像和 game variables 的 heuristic policy。

    - 具体 task：

        - VizDoom D1 Basic

        - VizDoom D3 Battle





# Track C: Model Training Optimization

优化训练 recipe，包括 hyperparameters、optimizer、architecture、schedule、loss、data order、augmentation、training loop这些，AK的autoresearch就是在这种task上做的

- **modded\-nanogpt Family**：以 [KellerJordan/modded\-nanogpt](https://github.com/KellerJordan/modded-nanogpt) 为主 task。该 repo 是 NanoGPT speedrun，目标是在 8×H100 上尽快训练到 FineWeb validation 3\.28 cross\-entropy；另有 optimization track，在固定 architecture/data/batch size 下最小化 steps。 [oai\_citation:3‡GitHub](https://github.com/KellerJordan/modded-nanogpt?utm_source=chatgpt.com) **后面这个track不需要八张H100，一张卡就行**

- **Karpathy AutoResearch Family**：以 [karpathy/autoresearch](https://github.com/karpathy/autoresearch) 为 task。它的核心就是让 agent 修改小型 LLM training setup，训练 5 分钟，检查结果是否提升，保留或丢弃修改，然后循环。 [oai\_citation:4‡GitHub](https://github.com/karpathy/autoresearch?utm_source=chatgpt.com)



# Track D: Algorithm Optimization

优化算法，通过code实现，具体的task非常多，有点类似那种toy example，但感觉很合适我们的bench

- **AlphaEvolve的task**：以 AlphaEvolve\-style algorithm discovery https://github\.com/google\-deepmind/alphaevolve\_results为核心，用 [OpenEvolve](https://github.com/algorithmicsuperintelligence/openevolve) / CodeEvolve 类开源实现构造 task。

    - 可用 task：

        - matrix multiplication search

        - scheduling heuristic discovery

        - packing / covering heuristic discovery

        - numerical algorithm optimization

        - algorithmic competition problems

- **HeuriGym / CO Heuristics Family**：以 [HeuriGym](https://github.com/cornell-zhang/heurigym) 为核心。HeuriGym 评估 LLM 是否能通过 code\-driven interaction 生成并 refine combinatorial optimization heuristics。 [oai\_citation:6‡GitHub](https://github.com/cornell-zhang/heurigym?utm_source=chatgpt.com)

    - task：

        - TSP

        - SAT

        - Knapsack

        - Bin Packing

        - Set Cover

        - MaxCut

        - Graph Coloring

        - Job\-shop Scheduling

    - 可优化内容：

        - construction heuristic

        - local search neighborhood

        - restart strategy

        - greedy rule

        - metaheuristic parameters



# Track E: Kernel / Compiler Optimization

优化底层系统 artifact，比如 CUDA/Triton kernel、compiler flags、pass sequence、scheduler、serving config。这个 track 的 evaluator 最硬：先 correctness gate，再测 speed / memory / binary size。这是hj师兄提到的，灵感是之前隔壁组一直在用cc优化国产算子

- **GPU Kernel Optimization Family**：以 [KernelBench](https://github.com/ScalingIntelligence/KernelBench) 为主 bench。KernelBench 评估 LLM 生成 correct and efficient CUDA / DSL kernels 的能力，任务是为 PyTorch programs 写更快的 GPU kernels。 [oai\_citation:7‡GitHub](https://github.com/ScalingIntelligence/KernelBench?utm_source=chatgpt.com)

- **Triton Operator Optimization Family**：以 [TritonBench](https://github.com/thunlp/TritonBench) 为 complementary bench。TritonBench 是 performance\-aware Triton generation benchmark，包含 TritonBench\-G 和 TritonBench\-T 两个 channel。 [oai\_citation:8‡arXiv](https://arxiv.org/html/2502.14752v1?utm_source=chatgpt.com)

- **Compiler Flag / Pass Tuning Family**：优化 compiler flags 或 pass sequences。



# Track F: Rubric Optimization

这个我最开始想的是prompt optimization，后来想了想感觉rubric更合适些，优化 evaluator artifact，而不是优化被评价模型

- **Judge Prompt Calibration Family**：用 human\-labeled evaluation data 优化 judge prompt / rubric，使 LLM judge 更贴近 human labels。

- **LLM\-Rubric / Autorubric Family**：用 rubric\-based evaluation frameworks 构造 task。Autorubric 是 rubric\-based LLM evaluation framework；PaperBench 也使用 hierarchical rubrics 来评价论文复现结果。 [oai\_citation:9‡GitHub](https://github.com/openai/preparedness/blob/main/project/paperbench/README.md?utm_source=chatgpt.com)

- **Reward / Verifier Rubric Family**：把 rubric 用作 reward/verifier，特别适合 math、code、search agent。

    - 可用 task：

        - math verifier rubric

        - code verifier rubric

        - citation verifier rubric

        - instruction\-following reward rubric

        - search\-answer reward rubric



# Track G: Paper Reproduction Optimization

这是刘老师提到的优化论文复现过程，paper reproduction。Agent 从 paper、partial repo、baseline implementation 或 broken reproduction package 出发，自动改进 reproduction artifact。

主要是Paper Bench 和 ResearchGym这两个，但我觉得复现完整流程成本稍高，可能可以截取片段来复现

和Harness优化有重复

或者复现算法





# ~~Track H: Data Synthesis Optimization~~

~~优化数据合成 pipeline，使得用合成数据训练后的模型在下游 metric 上更高。这个 track 放最后，因为 evaluator 最贵，我们自己跑实验的解决方案是我们不优化直接下游metric，以math reasoning的数据合成为例，我们优化pass@4\-pass@1这个指标，作为衡量题目质量的指标，可以类似的思路~~

~~或者就用posttrain bench上的task~~

- ~~Search agent data~~

- ~~Math Reasoning data~~

- ~~Code Agent data~~

- ~~Tool\-Use data~~





- 题少一点

- 动态榜单

