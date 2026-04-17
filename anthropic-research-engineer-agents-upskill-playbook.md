# Upskill Playbook: Anthropic Research Engineer, Agents

**Created:** 2026-04-16
**Target role:** Research Engineer, Agents (Anthropic)
**Posting:** https://job-boards.greenhouse.io/anthropic/jobs/4017544008

---

## 1. Skills Gap Analysis

The role has five core responsibilities. Here is each one mapped to current strengths and gaps.

| Responsibility | Current Strength | Gap to Close |
|---|---|---|
| Agent harness design (memory, context compression, communication architectures) | **Strong.** Building multi-agent orchestration daily in Claude Code with MCP, sub-agents, memory files, context management. | Formalize knowledge of compression techniques (summarization, clipping, hierarchical memory). Study published harness architectures (OpenHands event-stream, SWE-agent, Sonar free-workflow). |
| Quantitative benchmarks for agentic tasks | **Moderate.** Understands eval concepts but hasn't built or run formal benchmark harnesses. | Build and run a benchmark harness against SWE-bench Verified or GAIA. Learn evaluation infrastructure (docker sandboxing, deterministic scoring, statistical analysis of pass rates). |
| Automated evaluation / model-based evaluation | **Moderate.** Uses LLMs to check outputs informally. | Study LLM-as-a-Judge methodology formally. Build a scaled evaluation pipeline using model-based grading with calibration against human labels. |
| Data mix optimization for training | **Low.** No direct experience with training data curation or mixing. | Study data mixing laws (ICLR 2025 paper), curriculum learning, and the mechanics of how data composition affects model behavior. Hands-on with small-scale experiments using TRL. |
| Large-scale RL on language models (RLHF, DPO, GRPO) | **Low.** Conceptual understanding only. No hands-on training experience. | This is the biggest gap. Requires structured learning path from theory through implementation. |

**Additional preferred qualifications:**
- Multi-agent systems: **Strong.** Daily practice.
- Finetuning for tool use: **Low-Moderate.** Understands the concept, hasn't done it hands-on.

---

## 2. Learning Paths by Priority

### Priority 1: RL on Language Models (Weeks 1-4)
This is the largest gap and the most important to close. The role lists "large-scale reinforcement learning experience on language models" as a preferred qualification.

**Learning sequence:**

1. **Foundations (Week 1)**
   - Watch Serrano.Academy's "Reinforcement Learning for LLMs" (covers RLHF, PPO, DPO, GRPO): https://www.classcentral.com/course/youtube-reinforcement-learning-for-llms-513015
   - Read the RLHF Book (free PDF): https://rlhfbook.com/book.pdf
   - Read Hugging Face blog on the RLHF pipeline: https://huggingface.co/blog/NormalUhr/rlhf-pipeline

2. **DPO Deep Dive (Week 2)**
   - Phil Schmid's practical guide "How to align open LLMs in 2025 with DPO": https://www.philschmid.de/rl-with-llms-in-2025-dpo
   - Run the Hugging Face TRL DPO tutorial end-to-end on a small model
   - TRL documentation: https://huggingface.co/docs/trl/en/index

3. **GRPO Implementation (Week 3)**
   - Hugging Face Learn Chapter 12.4, implementing GRPO in TRL: https://huggingface.co/learn/llm-course/chapter12/4
   - HF Cookbook: Post-training an LLM for reasoning with GRPO: https://huggingface.co/learn/cookbook/en/fine_tuning_llm_grpo_trl
   - Understand how DeepSeek R1 used GRPO to eliminate the critic network

4. **Scaling Understanding (Week 4)**
   - Technical survey of RL techniques for LLMs: https://arxiv.org/html/2507.04136v1
   - Red Hat post-training methods overview: https://developers.redhat.com/articles/2025/11/04/post-training-methods-language-models
   - Study TRL v1.0 architecture (released April 2026) as the production-ready framework: https://github.com/huggingface/trl

**Key concepts to internalize:** PPO vs DPO vs GRPO tradeoffs, reward model training, preference data construction, KL divergence regularization, on-policy vs off-policy methods, why GRPO cuts memory ~25% by eliminating the critic.

### Priority 2: Agent Benchmarks and Evaluation (Weeks 3-6)
This directly maps to two of the five core responsibilities.

**Learning sequence:**

1. **Benchmark Landscape (Week 3-4)**
   - Study SWE-bench ecosystem: SWE-bench Verified, SWE-bench Pro, SWE-bench Live, SWE-rebench
   - SWE-bench main site: https://www.swebench.com/
   - SWE-rebench: https://swe-rebench.com/
   - GAIA benchmark and leaderboard (Princeton HAL): https://hal.cs.princeton.edu/gaia
   - WebArena benchmark: study how OpAgent hit 71.6%
   - Phil Schmid's Agent Benchmark Compendium (50+ benchmarks cataloged): https://github.com/philschmid/ai-agent-benchmark-compendium

2. **Evaluation Harness Architecture (Week 4-5)**
   - Study OpenHands benchmark harness: https://github.com/OpenHands/benchmarks
   - Study OpenHarness (open agent harness with built-in agent): https://github.com/HKUDS/OpenHarness
   - Read the Claude Code harness paper: https://arxiv.org/html/2603.05344v1
   - Understand docker sandboxing, deterministic replay, pass@k metrics

3. **LLM-as-a-Judge (Week 5-6)**
   - Hugging Face Cookbook on LLM Judge: https://huggingface.co/learn/cookbook/en/llm_judge
   - Training an LLM-as-a-Judge paper: https://arxiv.org/html/2502.02988v1
   - DeepEval framework (pytest-like LLM testing): https://github.com/confident-ai/deepeval
   - Langfuse for evaluation observability: https://langfuse.com/docs/evaluation/evaluation-methods/llm-as-a-judge
   - KDD '25 survey on LLM agent evaluation: https://arxiv.org/html/2507.21504v1

### Priority 3: Agent Harness Architecture (Weeks 5-8)
You already build these. The goal is to formalize and extend your knowledge to match research-grade systems.

**Key papers and resources:**

1. **Harness Engineering**
   - "Externalization in LLM Agents: Memory, Skills, Protocols and Harness Engineering": https://arxiv.org/html/2604.08224
   - "Agentic Harness Engineering: LLMs as the New OS": https://www.decodingai.com/p/agentic-harness-engineering
   - "The Anatomy of an Agent Harness" (Avi Chawla): https://blog.dailydoseofds.com/p/the-anatomy-of-an-agent-harness
   - Sebastian Raschka's "Components of a Coding Agent": https://magazine.sebastianraschka.com/p/components-of-a-coding-agent

2. **Context Compression**
   - "Active Context Compression: Autonomous Memory Management in LLM Agents": https://arxiv.org/html/2601.07190
   - Phil Schmid's "Context Engineering for AI Agents: Part 2": https://www.philschmid.de/context-engineering-part-2

3. **Architecture Paradigms to Study**
   - Sonar Foundation Agent: free-workflow model, stateful bash execution, AST symbol searching, test-driven remediation (79.2% SWE-bench Verified)
   - OpenHands: event-stream architecture, typed actions/observations, Docker sandboxing (77.6% SWE-bench Verified)
   - Live-SWE-agent: self-evolving agent that modifies itself at runtime
   - Standard "Planner-Executor-Memory" modular architecture

### Priority 4: Finetuning for Tool Use (Weeks 7-9)
Maps directly to the representative project "Claude finetuning maximizing agent tool performance."

**Resources:**

1. **Tutorials**
   - Weights & Biases guide to fine-tuning for function calling: https://wandb.ai/wandb/function-calling-finetuning/reports/Fine-tuning-LLMs-for-function-calling--VmlldzoxMjgxMTgxMg
   - Hugging Face Cookbook, fine-tuning with xLAM dataset: https://huggingface.co/learn/cookbook/function_calling_fine_tuning_llms_on_xlam
   - Fine-tune LLM agent for tool use with Hugging Face: https://kyrylai.com/2025/04/01/fine-tune-llm-agent-tool-use-huggingface/
   - Parlance Labs video on fine-tuning for function calling: https://parlance-labs.com/education/fine_tuning/pawel.html

2. **Key Datasets and Approaches**
   - xLAM dataset (Salesforce): function calling training data
   - ToolACE: agent-based data collection for tool use
   - ToolLLM: 16,464 real-world APIs
   - Llama3-FunctionCalling repo: https://github.com/michaelnny/Llama3-FunctionCalling

3. **Research**
   - "Improving Large Language Models Function Calling" (EMNLP 2025): https://aclanthology.org/2025.emnlp-main.1242.pdf
   - RLVR two-step method (SFT then RL) for tool-calling enhancement

### Priority 5: Data Mix Optimization (Weeks 9-10)
Lower priority because this is more of a team-level skill, but worth understanding.

**Resources:**
- "Data Mixing Laws" (ICLR 2025): https://openreview.net/forum?id=jjCB27TMK3
- "Dynamic Data Mixing Maximizes Instruction Tuning" (NAACL 2025): https://aclanthology.org/2025.naacl-long.80.pdf
- "Curriculum Learning for LLM Pretraining" (2026): https://arxiv.org/pdf/2601.21698
- Google's AutoCurriculum system (RL-based dynamic data mixing, +9.3% on reasoning)
- Mid-training survey: https://arxiv.org/html/2510.06826v1

### Priority 6: ML Foundations Refresh (Parallel, ongoing)
If you need a refresher on core ML concepts that underpin all of the above.

- **Karpathy's Neural Networks: Zero to Hero** (free, YouTube): build from micrograd through GPT
- **Karpathy's nanochat** (Oct 2025): full-stack ChatGPT clone from scratch in ~8,000 lines, $100-$1000 training cost: https://github.com/karpathy/build-nanogpt
- **Stanford CS229** (if deeper foundations needed): https://cs229.stanford.edu/
- **Reasoning models resource collection**: https://github.com/rkinas/reasoning_models_how_to

---

## 3. Portfolio Projects

These map directly to the "representative projects" listed in the job posting.

### Project 1: Agent Benchmark Harness and Novel Evaluation
**Maps to:** "Novel agent harnesses outperforming existing agents on coding/knowledge benchmarks" + "Scaled evaluation frameworks utilizing model-based techniques"

**What to build:**
- Fork OpenHands benchmarks or build a lightweight harness from scratch
- Implement a benchmark runner that evaluates agents on SWE-bench Verified (even a subset of 50-100 instances)
- Add an LLM-as-a-Judge evaluation layer that scores agent reasoning quality, not just pass/fail
- Compare at least 2-3 different agent architectures (e.g., single-turn bash-only vs. multi-step planner-executor vs. your own design)
- Publish results with statistical analysis (confidence intervals, effect sizes)

**Why this is strong:** It demonstrates you can build evaluation infrastructure, run rigorous benchmarks, and analyze results quantitatively. This is exactly what the team does daily.

**Tech stack:** Python, Docker (for sandboxing), pytest or DeepEval for test framework, Claude API for the judge model.

**Estimated time:** 3-4 weeks part-time.

### Project 2: Tool-Use Finetuning Pipeline
**Maps to:** "Claude finetuning maximizing agent tool performance"

**What to build:**
- Take an open-weight model (Qwen 2.5 7B or Llama 3.x 8B)
- Create a synthetic dataset of tool-use interactions (function calling, bash commands, file operations)
- Fine-tune using TRL with both SFT and DPO/GRPO stages
- Evaluate before/after on a tool-use benchmark (BFCL or custom)
- Write up the data construction methodology, training curves, and ablation results

**Why this is strong:** It demonstrates hands-on RL training experience, data construction skills, and the ability to measure improvement rigorously. Directly addresses the "finetuning" representative project.

**Tech stack:** Hugging Face TRL, Unsloth (for efficient fine-tuning), Weights & Biases for tracking, 1-2 GPUs (rent from Lambda or RunPod).

**Estimated time:** 3-4 weeks part-time.

### Project 3: Multi-Agent Coordination Evaluation
**Maps to:** "Novel evaluations measuring group agent problem-solving interactions"

**What to build:**
- Design a benchmark that specifically measures multi-agent coordination quality
- Create scenarios where agents must collaborate (e.g., one agent writes code, another reviews, a third writes tests)
- Implement metrics for: communication efficiency, role adherence, conflict resolution, task decomposition quality
- Use LLM-as-a-Judge to score each dimension
- Compare centralized vs. decentralized coordination strategies
- Open-source the evaluation framework

**Why this is strong:** Multi-agent evaluation is explicitly called out as a representative project, and it's an open research problem. Most benchmarks today evaluate single agents. This would be genuinely novel.

**Tech stack:** Python, your existing multi-agent orchestration experience, Claude API.

**Estimated time:** 2-3 weeks part-time.

---

## 4. Key Resources Summary

### Papers (Must-Read)
| Paper | Topic | Link |
|---|---|---|
| Externalization in LLM Agents | Harness engineering taxonomy | https://arxiv.org/html/2604.08224 |
| Building AI Coding Agents for the Terminal | Claude Code architecture | https://arxiv.org/html/2603.05344v1 |
| Active Context Compression | Memory management | https://arxiv.org/html/2601.07190 |
| Data Mixing Laws | Training data optimization | https://openreview.net/forum?id=jjCB27TMK3 |
| Training an LLM-as-a-Judge | Model-based evaluation | https://arxiv.org/html/2502.02988v1 |
| KDD '25 Agent Evaluation Survey | Benchmark taxonomy | https://arxiv.org/html/2507.21504v1 |
| RL Techniques for LLMs Survey | RLHF/DPO/GRPO landscape | https://arxiv.org/html/2507.04136v1 |
| Multi-Agent Collaboration Survey | Coordination mechanisms | https://arxiv.org/pdf/2501.06322 |
| OpenHands Agent SDK | Agent platform architecture | https://arxiv.org/html/2511.03690v1 |
| LLM Agent Comprehensive Survey | Architectures and capabilities | https://www.preprints.org/manuscript/202512.2119 |

### Repos (Hands-On)
| Repo | Purpose |
|---|---|
| https://github.com/huggingface/trl | TRL v1.0: SFT, DPO, GRPO training |
| https://github.com/OpenHands/benchmarks | Agent evaluation harness |
| https://github.com/HKUDS/OpenHarness | Open agent harness framework |
| https://github.com/confident-ai/deepeval | LLM evaluation framework |
| https://github.com/philschmid/ai-agent-benchmark-compendium | 50+ benchmark catalog |
| https://github.com/karpathy/build-nanogpt | GPT from scratch |
| https://github.com/michaelnny/Llama3-FunctionCalling | Tool-use finetuning |
| https://github.com/rkinas/reasoning_models_how_to | RL training resources |

### Courses and Tutorials
| Resource | Format | Focus |
|---|---|---|
| Serrano.Academy RL for LLMs | Video (free) | RLHF, PPO, DPO, GRPO overview |
| RLHF Book | PDF (free) | Comprehensive RLHF theory |
| HF Learn Chapter 12 | Tutorial | GRPO implementation |
| HF Cookbook: DPO + GRPO | Notebooks | Hands-on training |
| Karpathy Zero to Hero | Video (free) | Neural net foundations |
| Phil Schmid's guides | Blog posts | DPO alignment, context engineering |
| W&B Function Calling Guide | Tutorial | Tool-use finetuning |

### Benchmarks to Know
| Benchmark | What It Measures | Current SOTA |
|---|---|---|
| SWE-bench Verified | Real GitHub issue resolution | Sonar 79.2%, OpenHands 77.6% |
| SWE-bench Pro | Harder coding tasks | Claude Opus 4.1 ~23%, GPT-5 ~23% |
| GAIA | General AI assistant tasks | Claude Sonnet 4.5 74.6% (HAL framework) |
| WebArena | Web navigation tasks | OpAgent 71.6% |
| BFCL | Function calling accuracy | Various |
| Tau-bench | Agentic task completion | Various |

---

## 5. Timeline: 10-Week Plan

### Weeks 1-2: RL Foundations
- [ ] Watch Serrano.Academy RL for LLMs series
- [ ] Read RLHF Book chapters 1-5
- [ ] Read HF blog on RLHF pipeline
- [ ] Read Phil Schmid's DPO practical guide
- [ ] Set up TRL environment locally or on RunPod
- [ ] Run DPO tutorial end-to-end on a small model

### Weeks 3-4: GRPO + Benchmark Landscape
- [ ] Complete HF Learn Chapter 12.4 (GRPO in TRL)
- [ ] Run GRPO cookbook tutorial
- [ ] Read RL techniques survey paper
- [ ] Study SWE-bench ecosystem (read the papers, explore leaderboards)
- [ ] Study GAIA and WebArena benchmarks
- [ ] Read Phil Schmid's benchmark compendium
- [ ] Read Claude Code harness paper

### Weeks 5-6: Evaluation and Harness Architecture
- [ ] Read harness engineering papers (Externalization, Anatomy, Active Context Compression)
- [ ] Study OpenHands event-stream architecture (read the paper + code)
- [ ] Study Sonar agent architecture
- [ ] Complete LLM-as-a-Judge tutorial (HF Cookbook)
- [ ] Read the KDD '25 agent evaluation survey
- [ ] Read Training an LLM-as-a-Judge paper
- [ ] **Start Project 1:** Set up benchmark harness infrastructure

### Weeks 7-8: Portfolio Project 1 (Benchmark Harness)
- [ ] Build docker-sandboxed evaluation runner
- [ ] Implement 2-3 agent strategies to compare
- [ ] Run evaluations on SWE-bench subset
- [ ] Add LLM-as-a-Judge scoring layer
- [ ] Analyze results with statistical rigor
- [ ] Write up findings

### Weeks 8-9: Tool-Use Finetuning (Project 2)
- [ ] Read W&B function calling guide + xLAM cookbook
- [ ] Construct synthetic tool-use dataset
- [ ] Run SFT stage on small open-weight model
- [ ] Run DPO or GRPO stage for tool-use alignment
- [ ] Evaluate before/after on BFCL or custom benchmark
- [ ] Document training curves and ablations

### Week 10: Multi-Agent Eval + Polish (Project 3)
- [ ] Design multi-agent coordination benchmark scenarios
- [ ] Implement coordination metrics
- [ ] Run evaluation comparing coordination strategies
- [ ] Polish all three project writeups
- [ ] Update resume and application materials with new projects

### Ongoing (parallel throughout)
- [ ] Karpathy Zero to Hero or nanochat if foundations need shoring up
- [ ] Read 1-2 papers per week from the must-read list
- [ ] Follow Anthropic research page for new publications
- [ ] Study data mixing laws paper when ready (lower priority)

---

## 6. What You Already Bring

Do not underestimate the practical agentic systems experience. The role posting says "Experience developing sophisticated agentic systems with LLMs" as a required qualification, and you exceed this:

- **Multi-agent orchestration:** 15+ specialized agents with routing, memory, and tool delegation
- **MCP server development:** Built and deployed custom MCP servers (FastMCP + httpx + Docker pattern)
- **Agent harness design:** Context management, sub-agent delegation, memory files, compaction strategies
- **Production agentic systems:** Claude Code workflows handling real infrastructure, deployment, security
- **Tool integration:** Extensive experience connecting LLMs to real-world tools (Docker, Git, APIs, databases)
- **Prompt engineering at scale:** Complex system prompts, agent routing tables, security boundaries

The upskill work focuses on the ML/training side and formalizing the benchmarking side. The systems engineering and agentic architecture experience is already strong.

---

## 7. Vocabulary

Terms you need to use fluently in interviews and application materials.

### Reinforcement Learning & Training

| Term | Definition |
|---|---|
| **RLHF** (Reinforcement Learning from Human Feedback) | Training pipeline where human preference labels train a reward model, which then guides policy optimization via PPO. The original alignment technique used by InstructGPT and early ChatGPT. |
| **PPO** (Proximal Policy Optimization) | RL algorithm that updates the policy in small, stable steps using a clipped objective function. Requires four models in memory (policy, reference, reward, critic). Standard but expensive. |
| **DPO** (Direct Preference Optimization) | Simplifies RLHF by skipping the reward model entirely. Treats the language model itself as an implicit reward model and optimizes directly on preference pairs. Off-policy (uses static data), cheaper than PPO. |
| **GRPO** (Group Relative Policy Optimization) | Introduced by DeepSeek. Eliminates the critic network by computing baselines from group scores across multiple sampled outputs. Cuts memory ~25% vs PPO. Used in DeepSeek R1's reasoning training. |
| **SFT** (Supervised Fine-Tuning) | Standard fine-tuning on input/output pairs. Usually the first stage before RL. Teaches format and basic task completion. |
| **Reward model** | A model trained on human preference data to score outputs. Used by PPO to provide training signal. DPO and GRPO bypass this. |
| **KL divergence** | A regularization term that penalizes the trained model for drifting too far from the reference (base) model. Prevents reward hacking and mode collapse. |
| **On-policy vs off-policy** | On-policy (PPO, GRPO): generates new samples during training. Off-policy (DPO): trains on a fixed dataset. On-policy is more expensive but can explore better. |
| **Preference data** | Pairs of outputs where one is labeled as preferred over the other. The core training signal for RLHF, DPO, and GRPO. |
| **Reward hacking** | When the model exploits patterns in the reward signal to get high scores without actually being more helpful. KL regularization and reward model ensembles help prevent this. |
| **Data mix / data composition** | The ratio and combination of different data types (code, math, conversation, etc.) used in training. Small changes in mix ratios can significantly shift model capabilities. |
| **Curriculum learning** | Ordering training data from simple to complex. Can improve convergence and final performance compared to random shuffling. |
| **Mid-training** | A training stage between pretraining and post-training (SFT/RLHF). Used for domain adaptation or capability injection at scale before alignment. |

### Agent Architecture

| Term | Definition |
|---|---|
| **Agent harness** | The scaffolding code that wraps an LLM to make it act as an agent: prompt construction, tool dispatch, memory management, error recovery, observation parsing. Sometimes called "scaffolding" or "agent framework." |
| **Affordance** | A capability or interface exposed to the agent. File editing, bash execution, web browsing, and tool calling are all affordances. Designing what the agent can do is as important as designing how it thinks. |
| **Context window management** | Strategies for fitting relevant information into the model's finite context: summarization, compression, sliding windows, hierarchical memory, retrieval. |
| **Context compression** | Reducing the token count of conversation history while preserving essential information. Techniques include summarization, dropping low-value turns, and selective extraction. |
| **Event-stream architecture** | OpenHands' pattern where all agent actions and environment observations are logged as typed events in a stream. Enables replay, debugging, and evaluation. |
| **Free-workflow architecture** | Sonar's approach where the agent decides its own workflow dynamically rather than following a fixed plan-execute loop. No predetermined sequence of steps. |
| **Planner-Executor pattern** | A two-stage architecture where one LLM call creates a plan, then separate calls execute each step. Common but rigid compared to free-workflow. |
| **Tool calling / function calling** | The model outputs structured JSON specifying which function to call and with what arguments, rather than generating free text. The harness executes the function and returns results. |
| **Observation** | The result returned to the agent after an action (tool output, error message, file contents). The agent's "sensory input" from the environment. |
| **Multi-agent coordination** | Multiple agents working together on a task. Key challenges: task decomposition, communication protocols, conflict resolution, and knowing when to escalate. |
| **Communication architecture** | How agents in a multi-agent system exchange information: shared memory, message passing, blackboard systems, hierarchical reporting. |

### Evaluation & Benchmarks

| Term | Definition |
|---|---|
| **SWE-bench** | Benchmark where agents resolve real GitHub issues from popular Python repos. SWE-bench Verified (500 human-validated instances) is the standard leaderboard. Current SOTA: ~79%. |
| **GAIA** | General AI Assistants benchmark from Meta/HuggingFace. Tests multi-step reasoning, tool use, and web browsing on questions with unambiguous answers. Three difficulty levels. |
| **WebArena** | Benchmark for web navigation agents. Agents must complete tasks on realistic self-hosted websites (shopping, forums, maps, GitLab). |
| **BFCL** (Berkeley Function Calling Leaderboard) | Evaluates how accurately models can generate correct function calls across different formats and complexity levels. |
| **Pass@k** | The probability that at least one of k generated solutions passes all tests. Pass@1 is the standard metric; higher k values show ceiling potential. |
| **LLM-as-a-Judge** | Using a language model to evaluate the quality of another model's output. Cheaper than human evaluation, but requires calibration against human labels to be trustworthy. |
| **Model-based evaluation** | Broader term for using models in the evaluation pipeline: as judges, as test generators, as adversarial probers, or as simulation environments. |
| **Ablation study** | Systematically removing or changing one component at a time to measure its contribution. "What happens if we remove memory?" is an ablation. |
| **Effect size** | How large a measured difference is, not just whether it's statistically significant. A 0.5% improvement with p<0.01 may not be practically meaningful. |
| **Eval harness** | Infrastructure for running evaluations reproducibly: sandboxed execution environments, scoring pipelines, result aggregation, statistical analysis. |

### Finetuning & Data

| Term | Definition |
|---|---|
| **LoRA** (Low-Rank Adaptation) | Efficient fine-tuning that freezes the base model and trains small rank-decomposition matrices. Drastically reduces GPU memory and storage requirements. |
| **QLoRA** | LoRA applied to a quantized (4-bit) base model. Enables fine-tuning large models on consumer GPUs. |
| **Synthetic data** | Training data generated by models rather than collected from humans. Used heavily for tool-use training (generate diverse function calling scenarios). Quality filtering is critical. |
| **xLAM** | Salesforce's dataset for training function calling capabilities. Contains diverse tool-use scenarios with verified correct outputs. |
| **Training curves** | Plots of loss/accuracy over training steps. Reading these is essential for diagnosing underfitting, overfitting, learning rate issues, and data quality problems. |
| **Unsloth** | Library for efficient LLM fine-tuning. 2x faster than standard HF training with lower memory usage. Popular for LoRA/QLoRA workflows. |
| **TRL** (Transformer Reinforcement Learning) | Hugging Face's library for post-training LLMs. Supports SFT, DPO, GRPO, PPO, and other alignment methods. v1.0 released April 2026. |

---

## 8. Application Strategy Notes

- The posting explicitly says "Not all strong candidates will meet every single qualification." The RL experience is preferred, not required.
- Lead with the agentic systems portfolio. The daily multi-agent orchestration work is directly relevant.
- Frame the RL upskilling as active investment, showing initiative and ability to learn quickly.
- The representative projects in Section 3 above, if completed, would directly demonstrate capability for 4 of the 6 listed representative projects in the posting.
- The comp range ($500K-$850K) signals this is a senior IC role. They want builders who ship, not just researchers who write papers.
