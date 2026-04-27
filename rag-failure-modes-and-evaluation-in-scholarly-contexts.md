---
title: "Retrieval-Augmented Generation: Documented Failure Modes and Evaluation Methodologies in Scholarly and Academic-Library Contexts (2024-2026)"
date: 2026-04-27
updated: 2026-04-27 5:45 PM ET / 9:45 PM UTC
summary: "Peer-reviewed and arXiv literature 2024-2026 documents persistent RAG failure modes (retrieval miss, citation misgrounding, context-window crowding, conflicting-source handling) at rates of 17-33% even in production legal-research systems. Evaluation frameworks (RAGAS, RGB, CRUD-RAG, ARES, Ragnarök, FACTS Grounding, ChatRAG-Bench) measure these failures along orthogonal dimensions, but no framework eliminates them. RAG should be positioned in lit reviews as prior art with documented limits, not as a solved hallucination problem."
---

## Current Status

- The canonical engineering taxonomy is Barnett et al.'s **seven failure points** (CAIN 2024), spanning retrieval, consolidation, extraction, formatting, specificity, and completeness failures ([source](https://arxiv.org/abs/2401.05856)).
- **Stanford RegLab's preregistered empirical evaluation (Magesh et al. 2024, JELS 2025)** found that production legal-research RAG tools hallucinate on 17-33% of queries despite vendor claims of "hallucination-free" output, and introduced the critical distinction between **factual incorrectness** and **misgrounding** (citing a real source that does not support the claim) ([source](https://arxiv.org/abs/2405.20362)) ([source](https://onlinelibrary.wiley.com/doi/full/10.1111/jels.12413)).
- Foundational evaluation benchmarks now exist: **RAGAS** (EACL 2024), **RGB** (AAAI 2024), **CRUD-RAG** (TOIS 2024), **ARES** (NAACL 2024), **Ragnarök / TREC RAG 2024**, **FACTS Grounding** (Google DeepMind, December 2024), and **ChatRAG-Bench** (NeurIPS 2024). Each captures a different slice of failure.
- The **"lost in the middle"** phenomenon (Liu et al., TACL 2024) demonstrates that even when correct evidence is retrieved, LLMs systematically under-attend to mid-context content, producing a U-shaped accuracy curve that explains a class of grounding failures invisible to retrieval-only metrics ([source](https://arxiv.org/abs/2307.03172)).
- Library-deployed RAG systems (SJSU's KingbotGPT, EBSCO AI Insights, ProQuest Research Assistant, Elicit, Consensus, Scite, Scopus AI) ship with explicit "may be inaccurate" disclaimers; comparative evaluations from HKUST Library and others show variable reliability and confirm that source grounding does not equal source support ([source](https://library.hkust.edu.hk/sc/trust-ai-lit-rev/)) ([source](https://library.sjsu.edu/librarychatbot/kingbot)).
- An open question across the field: RAG mitigates but does not eliminate hallucination. The 2025 evaluation literature (Yu et al. survey, FACTUM, VeriCite) increasingly treats **citation hallucination despite grounding** as a distinct failure class requiring constraint-side rather than retrieval-side fixes.

## Findings

### 1. RAG Failure Taxonomies

**Barnett et al. — "Seven Failure Points When Engineering a Retrieval Augmented Generation System"** (CAIN 2024, ACM/IEEE 3rd International Conference on AI Engineering, Lisbon). The paper draws on three case studies (research, education, biomedical) and identifies seven failure points that any RAG implementation must address ([source](https://arxiv.org/abs/2401.05856)):

- **FP1: Missing Content** — "asking a question that cannot be answered from the available documents" ([source](https://arxiv.org/html/2401.05856v1)).
- **FP2: Missed the Top Ranked Documents** — "The answer to the question is in the document but did not rank highly enough to be returned" ([source](https://arxiv.org/html/2401.05856v1)).
- **FP3: Not in Context — Consolidation strategy** — relevant documents are retrieved but excluded during consolidation ([source](https://arxiv.org/html/2401.05856v1)).
- **FP4: Not Extracted** — "the answer is present in the context, but the large language model failed to extract out the correct answer" ([source](https://arxiv.org/html/2401.05856v1)).
- **FP5: Wrong Format** — "the large language model ignored the instruction" for output format ([source](https://arxiv.org/html/2401.05856v1)).
- **FP6: Incorrect Specificity** — "The answer is returned in the response but is not specific enough or is too specific to address the user's need" ([source](https://arxiv.org/html/2401.05856v1)).
- **FP7: Incomplete** — "Incomplete answers are not incorrect but miss some of the information even though that information was in the context" ([source](https://arxiv.org/html/2401.05856v1)).

Two conclusions are load-bearing for the IJoL argument: (a) "validation of a RAG system is only feasible during operation" — i.e., bench evaluation underestimates production failure — and (b) "the robustness of a RAG system evolves rather than designed in at the start" ([source](https://arxiv.org/abs/2401.05856)). Robustness emerges from operational discipline, not architecture.

**Liu et al. — "Lost in the Middle: How Language Models Use Long Contexts"** (TACL 2024). Authored by Nelson F. Liu, Kevin Lin, John Hewitt, Ashwin Paranjape, Michele Bevilacqua, Fabio Petroni, Percy Liang ([source](https://arxiv.org/abs/2307.03172)) ([source](https://aclanthology.org/2024.tacl-1.9/)). The paper documents a U-shaped performance curve: language models perform best when relevant information is at the beginning or end of the input context and substantially worse when it appears in the middle. This is a **context-grounding failure** that occurs after retrieval has succeeded — the evidence is present and the model still misses it. For RAG with k=5+ retrieved passages, this means correct passages buried at positions 3-4 are systematically under-used.

**Citation hallucination as a distinct class.** Recent work treats citation fabrication despite retrieval grounding as a separate failure mode requiring its own detection and mitigation machinery. The 2025-2026 literature includes FACTUM (mechanistic detection of citation hallucination in long-form RAG) and VeriCite (rigorous verification at SIGIR-AP 2025), both predicated on the observation that a model can output a real-looking citation that does not actually support the surrounding claim ([source](https://dl.acm.org/doi/10.1145/3767695.3769505)).

### 2. Evaluation Frameworks

**RAGAS — "Automated Evaluation of Retrieval Augmented Generation"** (Es, James, Espinosa-Anke, Schockaert, EACL 2024 System Demonstrations, pp. 150-158) ([source](https://aclanthology.org/2024.eacl-demo.16/)) ([source](https://arxiv.org/abs/2309.15217)). Reference-free framework with three core metrics, each LLM-judged ([source](https://arxiv.org/html/2309.15217v1)):

- **Faithfulness**: F = |V|/|S|. Statements are extracted from the answer and verified against context; the score is the fraction of supported statements. Operationalizes "the answer should be grounded in the given context."
- **Answer Relevance**: AR = (1/n)∑sim(q, qᵢ). The LLM generates n alternative questions from the answer; embedding similarity to the original question is averaged. Penalizes incomplete or redundant answers.
- **Context Relevance**: CR = (extracted critical sentences)/(total sentences in context). Measures whether retrieved context is focused.

RAGAS's reference-free design is its strength (no human-annotated ground truth required) and its weakness (the LLM judge can repeat the generator's biases). Faithfulness is a necessary but not sufficient measure — a fully "faithful" answer can still hallucinate citations that fall outside the statements being verified.

**RGB — "Benchmarking Large Language Models in Retrieval-Augmented Generation"** (Chen, Lin, Han, Sun, AAAI 2024) ([source](https://arxiv.org/abs/2309.01431)) ([source](https://ojs.aaai.org/index.php/AAAI/article/view/29728)). Evaluates four fundamental abilities required for RAG: **noise robustness**, **negative rejection**, **information integration**, **counterfactual robustness**. Bilingual (English and Chinese). Evaluation of six representative LLMs found that "while LLMs exhibit a certain degree of noise robustness, they still struggle significantly in terms of negative rejection, information integration, and dealing with false information" ([source](https://arxiv.org/abs/2309.01431)). Negative rejection — knowing when to abstain — is consistently the weakest dimension. Directly relevant to library use: a system that cannot abstain when sources are inadequate is not safe for reference work.

**CRUD-RAG — "A Comprehensive Chinese Benchmark for Retrieval-Augmented Generation of Large Language Models"** (Lyu et al., arXiv 2401.17043, published in ACM Transactions on Information Systems) ([source](https://arxiv.org/abs/2401.17043)) ([source](https://dlnext.acm.org/doi/10.1145/3701228)). Categorizes RAG applications as Create, Read, Update, Delete and constructs four tasks: text continuation, single- and multi-document QA, hallucination modification, open-domain multi-document summarization. Argues that prior benchmarks "predominantly assess question-answering applications while neglecting broader RAG scenarios."

**ARES — "An Automated Evaluation Framework for Retrieval-Augmented Generation Systems"** (Saad-Falcon, Khattab, Potts, Zaharia, NAACL 2024) ([source](https://aclanthology.org/2024.naacl-long.20/)) ([source](https://arxiv.org/abs/2311.09476)). Evaluates context relevance, answer faithfulness, and answer relevance. Generates synthetic training data, fine-tunes lightweight LM judges, then uses prediction-powered inference (PPI) with a small set of human annotations to bound prediction error. Claims robustness across domain shifts. Open source ([source](https://github.com/stanford-futuredata/ARES)).

**Ragnarök — "A Reusable RAG Framework and Baselines for TREC 2024 Retrieval-Augmented Generation Track"** (Pradeep, Thakur, Sharifymoghaddam, Zhang, Nguyen, Campos, Craswell, Lin, arXiv June 2024) ([source](https://arxiv.org/abs/2406.16828)). Frames the TREC 2024 RAG track around the MS MARCO V2.1 collection with industrial baselines (OpenAI GPT-4o, Cohere Command R+) and a head-to-head arena for crowdsourced pairwise judgments. Important because it standardizes I/O definitions for cross-system comparison.

**FACTS Grounding** (Google DeepMind & Google Research, December 17, 2024) ([source](https://deepmind.google/blog/facts-grounding-a-new-benchmark-for-evaluating-the-factuality-of-large-language-models/)). Benchmark of 1,719 examples (860 public, 859 private), each requiring a long-form response grounded in a context document up to 32,000 tokens spanning finance, technology, retail, medicine, and law. Two-phase judging: (1) eligibility — does the response address the request — and (2) factuality — is it fully grounded with no hallucinations. Three frontier judge LLMs (Gemini 1.5 Pro, GPT-4o, Claude 3.5 Sonnet) to mitigate within-family bias. Initial leaderboard had Gemini 2.0 Flash at 83.6% factuality — meaning roughly **one in six responses still contained ungrounded content** even from frontier models on a benchmark explicitly designed for grounded long-form generation.

**ChatRAG-Bench / ChatQA** (Liu, Ping et al., NVIDIA, NeurIPS 2024) ([source](https://arxiv.org/abs/2401.10225)) ([source](https://huggingface.co/datasets/nvidia/ChatRAG-Bench)). Built from ten existing datasets (Doc2Dial, QuAC, QReCC, TopioCQA, INSCIT, CoQA, HybriDialogue, DoQA, SQA, ConvFinQA). Evaluates conversational RAG, table reasoning, arithmetic, and — critically — unanswerable question detection. Adds the multi-turn dimension absent from RAGAS/RGB.

**Yu et al. — "Evaluation of Retrieval-Augmented Generation: A Survey"** (arXiv 2405.07437, May 2024) ([source](https://arxiv.org/abs/2405.07437)). Proposes the RGAR (Retrieval, Generation, Additional Requirement) framework. Reviews RAGAS, RGB, CRUD-RAG, ARES, MultiHop-RAG. Identifies three systemic failure-mode categories: (a) retrieval failures driven by dynamic/temporal knowledge bases, (b) generation failures driven by faithfulness/correctness gaps on subjective open-ended tasks, and (c) system-level failures where component-wise evaluation does not predict end-to-end behavior ([source](https://arxiv.org/html/2405.07437v1)).

### 3. Empirical RAG Performance on Citation-Critical Tasks

**Stanford RegLab — Magesh, Surani, Dahl, Suzgun, Manning, Ho — "Hallucination-Free? Assessing the Reliability of Leading AI Legal Research Tools"** (arXiv 2405.20362, May 2024; Journal of Empirical Legal Studies, 2025, jels.12413) ([source](https://arxiv.org/abs/2405.20362)) ([source](https://onlinelibrary.wiley.com/doi/full/10.1111/jels.12413)). The first preregistered empirical evaluation of proprietary legal RAG. Constructed dataset of over 200 open-ended legal queries spanning four legal-question categories. Tested LexisNexis Lexis+ AI, Thomson Reuters Westlaw AI-Assisted Research, Thomson Reuters Ask Practical Law AI, and GPT-4 as a non-RAG baseline.

Headline findings:

- **Lexis+ AI**: ~65% accurate; hallucinates on >17% of queries ([source](https://hai.stanford.edu/news/ai-trial-legal-models-hallucinate-1-out-6-or-more-benchmarking-queries)).
- **Westlaw AI-Assisted Research**: ~42% accurate; hallucinates ~33% of the time — roughly twice the Lexis+ rate ([source](https://hai.stanford.edu/news/ai-trial-legal-models-hallucinate-1-out-6-or-more-benchmarking-queries)).
- **Ask Practical Law AI**: refused to answer outright on more than 60% of queries — high abstention used as a hallucination-avoidance strategy ([source](https://hai.stanford.edu/news/ai-trial-legal-models-hallucinate-1-out-6-or-more-benchmarking-queries)).
- **GPT-4 baseline**: hallucinated at higher rates than the RAG tools, confirming RAG reduces but does not eliminate hallucination.

Critically, the study introduces a typology distinguishing two hallucination kinds ([source](https://hai.stanford.edu/news/ai-trial-legal-models-hallucinate-1-out-6-or-more-benchmarking-queries)):

1. **Factual incorrectness** — "describing the law wrongly or making a factual error."
2. **Misgrounding** — "the AI cited a real source, but the source did not actually support the claim being made."

The paper notes misgrounded citations are "harder to detect than invented ones" because the source genuinely exists. This is the central evidence that retrieval grounding ≠ semantic support. For Pete's argument, this is the cleanest empirical support that RAG's retrieval guarantee is upstream of, and orthogonal to, the constraint problem of "is the claim entailed by the source it points to."

**Watson — "Hallucinated Citation Analysis: Delving into Student-Submitted AI-Generated Sources at the University of Mississippi"** (The Serials Librarian, Vol. 85, No. 5-6, 2024) ([source](https://www.tandfonline.com/doi/abs/10.1080/0361526X.2024.2433640)). Library-side empirical study. Most suspect citations were hallucinations but "often included some real information, such as actual author names, book titles, or scholarly journal names" — i.e., remixes of authentic scholarly elements rather than pure inventions. This is the same failure mode the Stanford study found in legal RAG, observed in the wild on student work. Important because it confirms misgrounding is not a legal-domain artifact; it is a generation-time failure mode that surfaces wherever citation is required.

**Reference Hallucination Score (medical AI chatbots)** ([source](https://pmc.ncbi.nlm.nih.gov/articles/PMC11325115/)). In comparative evaluation, Elicit and SciSpace produced the lowest reference hallucination scores while ChatGPT 3.5 and Bing produced critical-level hallucination. Tools whose architecture restricts to indexed scholarly literature outperformed open-web tools by a wide margin — additional evidence that retrieval-side discipline reduces but does not eliminate fabrication.

### 4. Library-Specific RAG Deployments and Reported Issues

**KingbotGPT (San José State University Library)**, public launch September 2024 ([source](https://library.sjsu.edu/librarychatbot/kingbot)) ([source](https://github.com/sjsu-library/kingbotgpt)). Built on Streamlit, LlamaIndex, GPT-4o Mini, with a Chroma vector database over the library website and a local Q&A dataset. Documented in "Library-Led AI: Building a Library Chatbot as Service and Strategy" (ACRL 2025 Conference Proceedings) ([source](https://www.ala.org/sites/default/files/2025-03/Library-LedAI.pdf)). The library's chatbot policy is explicit: "Kingbot may provide incorrect and biased information... responses are automated and can be unpredictable. The accuracy, relevancy, and completeness of chatbot responses is not guaranteed" ([source](https://www.sjlibrary.org/policy/chatbot-policy)). RAG is named as the architecture; reliability is not claimed.

**EBSCO AI Insights** (beta released April-May 2024 to 50 libraries; 20-user UX study; June 2024 focus groups) ([source](https://about.ebsco.com/blogs/ebscopost/AI-insights-library-research)). Generates 3-5-point article summaries within EBSCOhost. EBSCO marks output as AI-generated with a verification disclaimer. 85% of end-users in the beta felt AI Insights would have a positive workflow impact — a usability signal, not an accuracy signal.

**ProQuest Research Assistant** (launched 2025 in ProQuest Central) ([source](https://about.proquest.com/en/blog/2025/academic-ai-powered-research-assistance-now-available-in-proquest-central/)). Provides "document insights instead of simple generative answers" — design choice explicitly steering away from open-ended generation toward extractive summarization.

**Comparative scholarly-search RAG tools** evaluated by HKUST Library ([source](https://library.hkust.edu.hk/sc/trust-ai-lit-rev/)):

- **Scite Assistant**: "generated arguments were generally well-supported by the sources"; full-text access improved accuracy.
- **Consensus**: extracted original text for verification but had fewer open-access sources.
- **Elicit**: retrieved fewer studies (4-8) but cited contradictory evidence others missed.
- **Scopus AI**: "the sources did not support the arguments" in some sections, with near-verbatim copying that "could result in plagiarism."

The HKUST evaluation explicitly cautions: "These tools should be used as a starting point for research and **not as a replacement for a thorough literature review**. The information they generate should **always be verified**" ([source](https://library.hkust.edu.hk/sc/trust-ai-lit-rev/)). Reliability degrades on "new or less documented subjects" — i.e., when retrieval coverage is thin, the generation step compensates by fabricating.

**Bevara, Lund, Mannuru et al. — "Prospects of Retrieval Augmented Generation (RAG) for Academic Library Search and Retrieval"** (Information Technology and Libraries, June 2025) ([source](https://ital.corejournals.org/index.php/ital/article/view/17361)). Surveys technical requirements (embedding pipelines, vector DBs, middleware) and frames RAG as "novel approach to academic information discovery" — but stops short of empirical failure-mode evaluation. Useful as a citation for "RAG is being actively integrated into academic library infrastructure"; does not refute the failure-mode literature.

### 5. Known Limitations Synthesized for the Lit Review

For an IJoL reviewer who needs RAG positioned as **prior art with documented limits, not a solved problem**, the load-bearing limitations are:

1. **Retrieval miss** — Barnett FP1/FP2; the answer simply is not retrieved.
2. **Consolidation loss** — Barnett FP3; retrieved but cut.
3. **Context-window crowding / lost in the middle** — Liu et al. TACL 2024; retrieved, in context, but under-attended.
4. **Extraction failure** — Barnett FP4; in context, attended, but the model returns the wrong span.
5. **Citation misgrounding** — Magesh et al. 2024; the cited source exists but does not support the claim. **This is the failure mode that retrieval cannot fix because retrieval has already succeeded.**
6. **Conflicting-source handling** — RAMDocs / RAG with Conflicting Evidence (arXiv 2504.13079, 2025) ([source](https://arxiv.org/abs/2504.13079)); multiple retrieved sources disagree, no constraint on resolution.
7. **Outdated index** — Yu et al. survey "temporal aspect of information"; the index lags ground truth.
8. **No quality judgment** — RAG retrieves "clinically-relevant evidence but remains methodologically blind, unable to judge study quality (e.g., retractions, underpowered analyses)" — applies equally to scholarly RAG.
9. **Failure mode masquerading as success** — Watson 2024; remixed real authors and titles produce plausible-looking false citations.

The lit-review framing for the IJoL paper writes itself: RAG addresses the *open-domain* hallucination problem (the model has no grounding) but not the *grounded-claim-correctness* problem (the model has grounding but does not reliably constrain its output to it). Constraint discipline operates at the layer that RAG, by construction, does not.

### 6. Timeline of Key Publications

| Date | Paper / Release | Significance |
|------|-----------------|--------------|
| 2024-12-17 | FACTS Grounding (Google DeepMind) | Long-form grounding benchmark; top model 83.6% factuality |
| 2024-09 | KingbotGPT launch (SJSU Library) | Production library RAG; explicit accuracy disclaimer |
| 2024-06-24 | Ragnarök / TREC RAG 2024 (Pradeep et al.) | Standardized I/O for TREC RAG track |
| 2024-05-30 | Hallucination-Free? (Magesh et al.) | 17-33% hallucination in production legal RAG; misgrounding typology |
| 2024-05-13 | Yu et al. RAG Evaluation Survey | RGAR framework synthesizing prior benchmarks |
| 2024-04 | EBSCO AI Insights beta | First commercial library RAG at scale |
| 2024-03 | RAGAS (EACL demos) | Reference-free faithfulness/relevance metrics |
| 2024-02 | RGB / Chen et al. (AAAI 2024) | Four fundamental abilities for RAG |
| 2024-01-30 | CRUD-RAG (Lyu et al.) | CRUD-typed multi-task benchmark |
| 2024-01-18 | ChatQA / ChatRAG-Bench (NVIDIA) | Conversational RAG with unanswerable detection |
| 2024-01-11 | Seven Failure Points (Barnett et al., CAIN) | Canonical engineering failure taxonomy |
| 2023-11-15 | ARES (Saad-Falcon et al., NAACL 2024) | Synthetic-trained LM judges with PPI bounds |
| 2023-09-26 | RAGAS preprint | Faithfulness/AR/CR metrics |
| 2023-09-04 | RGB preprint | Negative rejection identified as weakest dimension |
| 2023-07-06 | Lost in the Middle (Liu et al., TACL 2024) | U-shaped position curve in long-context RAG |
| 2020-05-22 | Lewis et al. — original RAG paper (NeurIPS 2020) | Defines the architecture being evaluated ([source](https://arxiv.org/abs/2005.11401)) |

## Confidence Assessment

### High Confidence

- The Seven Failure Points taxonomy (Barnett et al., CAIN 2024) names and definitions are quoted directly from the arXiv HTML version ([source](https://arxiv.org/html/2401.05856v1)).
- Magesh et al. hallucination rates (Lexis+ AI 17%+, Westlaw 33%, GPT-4 baseline higher) and the misgrounding-vs.-factual-incorrectness typology are confirmed across the arXiv preprint, the JELS publication, and Stanford HAI's summary ([source](https://arxiv.org/abs/2405.20362)) ([source](https://hai.stanford.edu/news/ai-trial-legal-models-hallucinate-1-out-6-or-more-benchmarking-queries)) ([source](https://onlinelibrary.wiley.com/doi/full/10.1111/jels.12413)).
- RAGAS metric definitions (Faithfulness F=|V|/|S|, Answer Relevance via embedded similarity, Context Relevance) are extracted verbatim from the arXiv HTML ([source](https://arxiv.org/html/2309.15217v1)).
- RGB's four abilities (noise robustness, negative rejection, information integration, counterfactual robustness) and the finding that negative rejection is the weakest dimension are confirmed against the arXiv abstract ([source](https://arxiv.org/abs/2309.01431)).
- FACTS Grounding parameters (1,719 examples, 860/859 split, 32k token docs, three frontier judges) are from the Google DeepMind blog post ([source](https://deepmind.google/blog/facts-grounding-a-new-benchmark-for-evaluating-the-factuality-of-large-language-models/)).
- Lost in the Middle author list and U-shaped finding are confirmed against the arXiv abstract ([source](https://arxiv.org/abs/2307.03172)).

### Medium Confidence

- The exact sample size for the Magesh et al. study is reported as "over 200 open-ended legal queries" via Stanford HAI; the original PDF was not directly fetchable in this session due to access controls.
- HKUST Library's comparative findings on Scite/Elicit/Consensus/Scopus AI are summarized from a library guide rather than a peer-reviewed paper; the rank-order of tools is consistent with the broader literature but the specific quotes are not from a refereed venue.
- The Watson 2024 Serials Librarian paper details (Vol. 85, No. 5-6) and characterization of "remixes of real authors, real titles, real publications" come from search-result summaries; the journal page is paywalled and could not be directly verified line-by-line.

### Low Confidence

- The exact factuality scores per model on the FACTS Grounding leaderboard beyond the headline 83.6% for Gemini 2.0 Flash. These shift as models update; the leaderboard is the source of truth ([source](https://www.kaggle.com/benchmarks/google/facts-grounding)).
- Specific failure-rate breakdowns for EBSCO AI Insights, ProQuest Research Assistant, and other library commercial deployments. Vendors publish satisfaction metrics and disclaimers but not error rates on standardized benchmarks.

## Open Questions

- What is the misgrounding rate (as distinct from full fabrication) in scholarly-RAG tools like Elicit, Consensus, and Scite? The HKUST evaluation suggests it is non-zero but does not quantify it.
- Do any peer-reviewed library-science journals (JAL, JASIST, College & Research Libraries, Information Technology and Libraries) have empirical evaluations of EBSCO AI Insights or ProQuest Research Assistant? The 2024-2025 ITAL piece by Bevara et al. is conceptual rather than empirical.
- How much of the gap between Lexis+ AI and Westlaw AI-Assisted Research (17% vs. 33% hallucination) is attributable to retrieval differences versus constraint/post-processing differences? The Magesh et al. paper does not attribute.
- Has any post-2025 work demonstrated a constraint-side mitigation that materially reduces misgrounding without depending on retrieval improvements? FACTUM and VeriCite are detection frameworks, not mitigations.

## Sources

### Foundational and Failure Taxonomy

- Barnett, S., Kurniawan, S., Thudumu, S., Brannelly, Z., & Abdelrazek, M. (2024). "Seven Failure Points When Engineering a Retrieval Augmented Generation System." *Proceedings of the IEEE/ACM 3rd International Conference on AI Engineering — Software Engineering for AI* (CAIN 2024), Lisbon. [arXiv:2401.05856](https://arxiv.org/abs/2401.05856) — [HTML](https://arxiv.org/html/2401.05856v1) — [ACM DL](https://dl.acm.org/doi/10.1145/3644815.3644945).
- Lewis, P., Perez, E., Piktus, A., Petroni, F., Karpukhin, V., Goyal, N., Küttler, H., Lewis, M., Yih, W., Rocktäschel, T., Riedel, S., & Kiela, D. (2020). "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks." NeurIPS 2020. [arXiv:2005.11401](https://arxiv.org/abs/2005.11401).
- Liu, N. F., Lin, K., Hewitt, J., Paranjape, A., Bevilacqua, M., Petroni, F., & Liang, P. (2024). "Lost in the Middle: How Language Models Use Long Contexts." *Transactions of the Association for Computational Linguistics*. [arXiv:2307.03172](https://arxiv.org/abs/2307.03172) — [ACL Anthology](https://aclanthology.org/2024.tacl-1.9/).

### Evaluation Frameworks

- Es, S., James, J., Espinosa-Anke, L., & Schockaert, S. (2024). "RAGAs: Automated Evaluation of Retrieval Augmented Generation." *EACL 2024 System Demonstrations*, pp. 150-158. [arXiv:2309.15217](https://arxiv.org/abs/2309.15217) — [ACL Anthology](https://aclanthology.org/2024.eacl-demo.16/) — [HTML](https://arxiv.org/html/2309.15217v1).
- Chen, J., Lin, H., Han, X., & Sun, L. (2024). "Benchmarking Large Language Models in Retrieval-Augmented Generation." *AAAI 2024*. [arXiv:2309.01431](https://arxiv.org/abs/2309.01431) — [AAAI proceedings](https://ojs.aaai.org/index.php/AAAI/article/view/29728).
- Lyu, Y. et al. (2024). "CRUD-RAG: A Comprehensive Chinese Benchmark for Retrieval-Augmented Generation of Large Language Models." *ACM Transactions on Information Systems*. [arXiv:2401.17043](https://arxiv.org/abs/2401.17043) — [ACM TOIS](https://dlnext.acm.org/doi/10.1145/3701228) — [GitHub](https://github.com/IAAR-Shanghai/CRUD_RAG).
- Saad-Falcon, J., Khattab, O., Potts, C., & Zaharia, M. (2024). "ARES: An Automated Evaluation Framework for Retrieval-Augmented Generation Systems." *NAACL 2024*. [arXiv:2311.09476](https://arxiv.org/abs/2311.09476) — [ACL Anthology](https://aclanthology.org/2024.naacl-long.20/) — [GitHub](https://github.com/stanford-futuredata/ARES).
- Pradeep, R., Thakur, N., Sharifymoghaddam, S., Zhang, E., Nguyen, R., Campos, D., Craswell, N., & Lin, J. (2024). "Ragnarök: A Reusable RAG Framework and Baselines for TREC 2024 Retrieval-Augmented Generation Track." [arXiv:2406.16828](https://arxiv.org/abs/2406.16828) — [Springer](https://link.springer.com/chapter/10.1007/978-3-031-88708-6_9).
- Google DeepMind & Google Research (2024-12-17). "FACTS Grounding: A new benchmark for evaluating the factuality of large language models." [DeepMind blog](https://deepmind.google/blog/facts-grounding-a-new-benchmark-for-evaluating-the-factuality-of-large-language-models/) — [Kaggle leaderboard](https://www.kaggle.com/benchmarks/google/facts-grounding).
- Liu, Z., Ping, W. et al. (2024). "ChatQA: Surpassing GPT-4 on Conversational QA and RAG." *NeurIPS 2024*. [arXiv:2401.10225](https://arxiv.org/abs/2401.10225) — [ChatRAG-Bench dataset](https://huggingface.co/datasets/nvidia/ChatRAG-Bench).
- Yu, H., Gan, A., Zhang, K., Tong, S., Liu, Q., & Liu, Z. (2024). "Evaluation of Retrieval-Augmented Generation: A Survey." [arXiv:2405.07437](https://arxiv.org/abs/2405.07437) — [HTML v1](https://arxiv.org/html/2405.07437v1) — [Awesome-RAG-Evaluation repo](https://github.com/YHPeter/Awesome-RAG-Evaluation).

### Empirical Evaluations on Citation-Critical Tasks

- Magesh, V., Surani, F., Dahl, M., Suzgun, M., Manning, C. D., & Ho, D. E. (2024/2025). "Hallucination-Free? Assessing the Reliability of Leading AI Legal Research Tools." *Journal of Empirical Legal Studies*. [arXiv:2405.20362](https://arxiv.org/abs/2405.20362) — [Wiley JELS](https://onlinelibrary.wiley.com/doi/full/10.1111/jels.12413) — [Stanford RegLab](https://reglab.stanford.edu/publications/hallucination-free-assessing-the-reliability-of-leading-ai-legal-research-tools/) — [HAI summary](https://hai.stanford.edu/news/ai-trial-legal-models-hallucinate-1-out-6-or-more-benchmarking-queries).
- Watson, A. P. (2024). "Hallucinated Citation Analysis: Delving into Student-Submitted AI-Generated Sources at the University of Mississippi." *The Serials Librarian*, 85(5-6). [Tandfonline](https://www.tandfonline.com/doi/abs/10.1080/0361526X.2024.2433640) — [eGrove showcase](https://egrove.olemiss.edu/ored_showcase/2025/posters/62/).
- "Reference Hallucination Score for Medical Artificial Intelligence Chatbots: Development and Usability Study" (2024). PMC. [PMC11325115](https://pmc.ncbi.nlm.nih.gov/articles/PMC11325115/).
- "Retrieval-Augmented Generation with Conflicting Evidence" (2025). [arXiv:2504.13079](https://arxiv.org/abs/2504.13079).
- VeriCite (SIGIR-AP 2025). [ACM DL](https://dl.acm.org/doi/10.1145/3767695.3769505).

### Library-Specific RAG Deployments

- Bevara, R. V. K., Lund, B. D., Mannuru, N. R., Karedla, S. P., Mohammed, Y., Kolapudi, S. T., & Mannuru, A. (2025). "Prospects of Retrieval Augmented Generation (RAG) for Academic Library Search and Retrieval." *Information Technology and Libraries*. [ITAL article](https://ital.corejournals.org/index.php/ital/article/view/17361).
- Mueller, D. M. (Ed.) (2025). "Library-Led AI: Building a Library Chatbot as Service and Strategy." *ACRL 2025 Conference Proceedings*. [ALA PDF](https://www.ala.org/sites/default/files/2025-03/Library-LedAI.pdf).
- San José State University Library. "Kingbot: A.I. Reference — KingbotGPT." [SJSU Library page](https://library.sjsu.edu/librarychatbot/kingbot) — [Source code](https://github.com/sjsu-library/kingbotgpt) — [Chatbot policy](https://www.sjlibrary.org/policy/chatbot-policy).
- EBSCO. "AI in Library Research and AI in Academic Research Platforms — Findings from EBSCO's Recent Beta Release." [EBSCO post](https://about.ebsco.com/blogs/ebscopost/AI-insights-library-research) — [Library Technology Guides](https://librarytechnology.org/document/30504).
- ProQuest. "AI-powered research assistance now available in ProQuest Central" (2025). [ProQuest blog](https://about.proquest.com/en/blog/2025/academic-ai-powered-research-assistance-now-available-in-proquest-central/).
- HKUST Library. "Trust in AI: Evaluating Scite, Elicit, Consensus, and Scopus AI for Generating Literature Reviews." [HKUST Library](https://library.hkust.edu.hk/sc/trust-ai-lit-rev/).

### Surveys and Tertiary Sources

- "Retrieval-Augmented Generation (RAG) Chatbots for Education: A Survey of Applications" (2025). *Applied Sciences* (MDPI). [MDPI](https://www.mdpi.com/2076-3417/15/8/4234).
- "A Systematic Review of Key Retrieval-Augmented Generation (RAG) Systems: Progress, Gaps, and Future Directions" (2025). [arXiv:2507.18910](https://arxiv.org/abs/2507.18910).
- "Mitigating Hallucination in Large Language Models (LLMs): An Application-Oriented Survey on RAG, Reasoning, and Agentic Systems" (2025). [arXiv:2510.24476](https://arxiv.org/html/2510.24476v1).

## Update History

- **2026-04-27 5:45 PM ET** — Initial report. Fresh Mode. Built from arXiv, ACL Anthology, AAAI, NAACL, EACL, NeurIPS, JELS, *Information Technology and Libraries*, *The Serials Librarian*, ACRL 2025 proceedings, Google DeepMind, Stanford RegLab/HAI, HKUST Library, EBSCO, ProQuest, and SJSU Library primary sources.

## How This Report Was Generated

Generated by the Research agent following the `/research` skill protocol. SearXNG (Tier 1) was attempted first but returned heavily polluted Chinese-language Zhihu mirrors for English-language RAG queries; Cornell Gateway Gemini search (Tier 2) was unavailable in this environment due to missing auth token; the report therefore relied on built-in WebSearch + WebFetch (Tier 3) with explicit verification of every cited URL. Direct quotes were extracted from arXiv HTML versions where possible; PDF-only sources (Stanford legal study, ACRL proceedings) were verified against multiple reputable secondary summaries (Stanford HAI, Sedona Conference references, journal landing pages) where the primary PDF could not be parsed. All claims are cited inline. Confidence levels are flagged. Timestamps are Eastern Time.
