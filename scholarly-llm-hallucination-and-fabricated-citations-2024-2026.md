---
title: "Scholarly LLM Hallucination and Fabricated Citations: A Literature Review (2024-2026)"
date: 2026-04-27
updated: 2026-04-27
status: complete
canary: r-2c91f70a
audience: peer-reviewed library journal reviewers and authors
summary: "Peer-reviewed and conference-grade evidence on LLM hallucination as it manifests in scholarly writing, peer review, and academic library workflows. Covers fabricated bibliographic citations (19.9 to 91.4 percent across studies), legal hallucination (58 to 88 percent on free-form queries; 17 to 33 percent on RAG-grounded legal tools), medical citation fabrication, infiltration of conference proceedings (50 plus at ICLR 2026 under review, 100 plus at NeurIPS 2025 accepted, 1 in 29 ICLR 2026 accepted papers), GPT-fabricated papers indexed by Google Scholar, and current factuality benchmarks (Vectara HHEM, FACTS Grounding, OpenAI SimpleQA/PersonQA). Most sources are 2024-2026 and APA-citable."
---

**TL;DR:** Across the strongest 2024-2026 evidence, LLMs produce fabricated or materially inaccurate bibliographic citations in roughly one-fifth to over half of attempts in peer-reviewed empirical studies, and the contamination has reached top venues. Stanford RegLab measured 58 percent legal hallucination by GPT-4 and 88 percent by Llama 2 on free-form legal queries (Dahl et al., 2024); a follow-up found 17 to 33 percent on commercial RAG legal tools (Magesh et al., 2024). A Deakin University experimental study in JMIR Mental Health found 19.9 percent of GPT-4o citations completely fabricated and 56 percent of all citations either fake or erroneous (Linardon et al., 2025). GPTZero confirmed 50 plus hallucinated citations in 300 ICLR 2026 submissions and 100 plus in 53 NeurIPS 2025 accepted papers (GPTZero, 2025; GPTZero, 2026). RefChecker, run by Microsoft researchers Russinovich and Siva Kumar against 5,348 ICLR 2026 accepted papers, flagged 349 likely hallucinated references across 184 papers (about 1 in 29). Fabricated GPT-generated papers are already indexed by Google Scholar, including in policy-relevant domains (Haider et al., 2024, *HKS Misinformation Review*). The implication for library practice: at current frontier rates, citation verification cannot be optional in any AI-assisted research workflow.

## Table of Contents

1. [Why This Review](#1-why-this-review)
2. [Current Factuality Benchmarks](#2-current-factuality-benchmarks)
3. [Fabricated Citations in Scholarly Writing](#3-fabricated-citations-in-scholarly-writing)
4. [Hallucinations Slipping Through Peer Review](#4-hallucinations-slipping-through-peer-review)
5. [GPT-Generated Papers in Scholarly Indexes](#5-gpt-generated-papers-in-scholarly-indexes)
6. [Domain-Specific Rates: Legal and Medical](#6-domain-specific-rates-legal-and-medical)
7. [Library and Information-Science Response](#7-library-and-information-science-response)
8. [Confidence Assessment](#8-confidence-assessment)
9. [Open Questions](#9-open-questions)
10. [Sources](#10-sources)

## 1. Why This Review

This report complements an existing technical metrics report ([ai-hallucination-metrics-2026.md](https://github.com/pete-builds/research-reports/blob/main/ai-hallucination-metrics-2026.md)) by focusing on hallucination as it appears specifically in scholarly artifacts: bibliographic references, peer-reviewed papers, indexed conference proceedings, legal and medical question answering, and library reference workflows. The intended use is the literature-review section of a peer-reviewed library journal article on anti-hallucination guardrails for research agents.

Sources are weighted toward 2024-2026 peer-reviewed journal articles, IEEE/ACM-tier conference content, *Nature*-family coverage, and reproducible industry benchmarks with publicly documented methods. Where only news coverage or pre-print evidence exists, the confidence flag is lowered explicitly.

## 2. Current Factuality Benchmarks

Three measurement systems dominate the 2024-2026 evidence base.

**Vectara HHEM Leaderboard.** Uses HHEM-2.3 to score models on summarization grounded in 7,700 curated articles ranging from 50 to 24,000 words across news, technology, science, medicine, legal, sports, business, and education domains. Temperature is fixed at 0; models that refuse or give minimal responses are filtered. As of the April 20, 2026 update, the top performers were Finix S1 32B (1.8 percent hallucination), GPT-5.4-Nano (3.1 percent), and Gemini 2.5 Flash Lite (3.3 percent), while Claude Sonnet 4 sat at 10.3 percent and several reasoning-tier models exceeded 10 percent ([source](https://github.com/vectara/hallucination-leaderboard)). The methodology measures **factual consistency with a source document**, not open-domain accuracy, which is why it cannot substitute for citation-accuracy benchmarks.

**Google DeepMind FACTS Grounding.** Released December 17, 2024. Comprises 1,719 examples (860 public, 859 held out) with documents up to 32,000 tokens drawn from finance, technology, retail, medicine, and law. Tasks include summarization, question generation, and rewriting that must be grounded exclusively in the supplied source. Evaluation uses three frontier-LLM judges (Gemini 1.5 Pro, GPT-4o, Claude 3.5 Sonnet) ([source](https://deepmind.google/discover/blog/facts-grounding-a-new-benchmark-for-evaluating-the-factuality-of-large-language-models/)). The benchmark deliberately excludes mathematics, creativity, and multi-step reasoning, so it isolates **grounded-context fidelity**.

**OpenAI SimpleQA and PersonQA.** Used as factual-knowledge probes. The OpenAI o3 and o4-mini system card (April 16, 2025) reports that o3 hallucinated on **33 percent of PersonQA questions, double the 16 percent rate of its predecessor o1**, and roughly **51 percent on SimpleQA** (with o4-mini at 79 percent on SimpleQA) ([source](https://cdn.openai.com/pdf/2221c875-02dc-4789-800b-e7758f3722c1/o3-and-o4-mini-system-card.pdf); coverage: [TechCrunch](https://techcrunch.com/2025/04/18/openais-new-reasoning-ai-models-hallucinate-more/)). Confidence: Medium-High. The figures come directly from OpenAI's system-card disclosure rather than independent peer review, so the methodological framing is vendor-controlled, but the numbers themselves are primary-source.

**Confidence: High** for the existence and methodology of HHEM and FACTS Grounding (primary sources accessible). **Medium** for the precise SimpleQA and PersonQA percentages (industry reporting only).

## 3. Fabricated Citations in Scholarly Writing

The clearest peer-reviewed evidence comes from controlled experimental studies in clinical and biomedical domains.

**Linardon et al. (2025), JMIR Mental Health.** A Deakin University team systematically prompted GPT-4o to generate literature reviews on mental-health topics with at least 20 peer-reviewed citations each. Of 176 generated citations, **19.9 percent were completely fabricated**; **45.4 percent of the real citations contained errors** in dates, page numbers, or DOIs; only **43.8 percent were both genuine and accurate**, meaning 56 percent were either fake or erroneous. Fabrication rate varied sharply by topic familiarity: 6 percent for major depressive disorder, 28 percent for binge eating disorder, 29 percent for body dysmorphic disorder. Notably, **64 percent of fabricated citations with DOIs linked to real but unrelated papers**, which makes errors substantially harder to detect without manual verification ([source](https://mental.jmir.org/2025/1/e80371); coverage: [StudyFinds](https://studyfinds.com/chatgpts-hallucination-problem-fabricated-references/), [Digital Health Science](https://www.digitalhealthscience.org/2025/11/17/new-study-reveals-high-rates-of-fabricated-and-inaccurate-citations-in-llm-generated-mental-health-research/)).

**Chelli et al. (2024), Journal of Medical Internet Research.** Compared GPT-3.5, GPT-4, and Bard/Gemini on systematic-review citation generation across 11 reviews of shoulder rotator-cuff pathology, totaling 471 references. Hallucination rates (papers were classed as hallucinated when at least two of title, first author, or year were wrong): **GPT-3.5 at 39.6 percent, GPT-4 at 28.6 percent, Bard at 91.4 percent**. Precision was very low across the board (GPT-4 at 13.4 percent; Bard at 0 percent). The authors conclude: "Given their current performance, it is not recommended for LLMs to be deployed as the primary or exclusive tool for conducting systematic reviews." ([source](https://www.jmir.org/2024/1/e53164)).

**Day (2023), Scientific Reports.** Earlier foundational work documented systematic ChatGPT bibliographic fabrication and provided the typology subsequent studies extended ([source](https://www.nature.com/articles/s41598-023-41032-5)). The article was inaccessible during this review (HTTP 303), so its specific percentages are not asserted here. Confidence: Medium for the specific numbers; High for its existence as a foundational reference.

**Confidence: High** for the Linardon and Chelli findings (peer-reviewed primary sources, intact methodology and percentages). **Medium** for the older Day 2023 specifics (fetch failed; cite the journal record but verify before quoting).

## 4. Hallucinations Slipping Through Peer Review

The most consequential 2025-2026 finding is that fabricated citations are passing the gatekeeping process at top venues.

**ICLR 2026 (under review).** GPTZero scanned 300 of approximately 20,000 submissions to ICLR 2026 using its Hallucination Check tool. Of 90 papers flagged, **50 plus contained confirmed hallucinated citations after human verification**, defined as "citations resulting from the use of generative AI that seem to paraphrase or combine the titles, authors, and/or metadata from one or more real sources." Each flagged paper had been reviewed by 3 to 5 peer experts who missed the fabrication. Some papers carried average reviewer ratings of 8/10. Common patterns included fabricated authors, incorrect publication years and venues, altered titles paired with real authors, and wrong arXiv IDs paired with real papers. Publication date: December 5, 2025 ([source](https://gptzero.me/news/iclr-2026/); coverage: [BetaKit](https://betakit.com/start-up-investigation-reveals-50-peer-reviewed-papers-contained-hallucinated-citations/)).

**ICLR 2026 (accepted papers).** Microsoft researchers Mark Russinovich and Ram Shankar Siva Kumar ran their RefChecker tool against every accepted paper at ICLR 2026 (5,348 papers, 180,501 references). RefChecker uses a layered verification approach against Semantic Scholar's 233-million-paper corpus with API fallbacks to CrossRef, DBLP, OpenAlex, and OpenReview, with Claude Sonnet 4.6 for extraction and final assessment. Results: **349 likely hallucinated references across 184 papers**, equivalent to **1 in 29 accepted papers (3.4 percent)** containing at least one likely hallucinated reference. Of affected papers, 38 had three or more hallucinations and 12 had five or more. Analysis date: April 22, 2026 ([source](https://github.com/markrussinovich/refchecker-iclr2026)). (Single-source confirmation; methodology and data are reproducible from the public repository, but independent replication has not yet been published.) Confidence: High for methodology and counts (full reproduction code published); Medium for the labelling threshold (the boundary between "uncertain" and "likely hallucinated" depends on Claude Sonnet 4.6's classifier and could be re-evaluated).

**NeurIPS 2025.** GPTZero scanned 4,841 of 5,290 accepted NeurIPS 2025 papers, finding **100 plus confirmed hallucinations across 53 papers**. NeurIPS 2025 received 21,575 main-track submissions with a 24.52 percent acceptance rate; submissions had grown from 9,467 in 2020 to 21,575 in 2025. The NeurIPS board's response: papers with incorrect references "are not necessarily invalidated" and authors may have used LLMs for citation formatting rather than content generation. Publication date: January 21, 2026 ([source](https://gptzero.me/news/neurips/); coverage: [Fortune](https://fortune.com/2026/01/21/neurips-ai-conferences-research-papers-hallucinations/)).

**Cross-cutting context.** A *Scientific American* analysis estimated approximately 1 percent of all 2023 scientific articles showed signs of AI involvement (60,000 plus papers), with up to 17.5 percent in computer science. Specific stylistic markers tracked dramatic shifts: "delve" usage rose 654 percent in PubMed from 2020 to 2023 (349 to 2,847 instances); "commendable" rose 245 percent on Scopus (240 to 829) ([source](https://www.scientificamerican.com/article/chatbots-have-thoroughly-infiltrated-scientific-publishing/)). Scientific-integrity consultant Elisabeth Bik is quoted: "The problem is that these tools are not good enough yet to trust" and warns they "generate citation references that don't exist."

**Confidence: High** for the ICLR and NeurIPS counts (methodology published, sample sizes large, human verification step documented). **High** for the Russinovich/Siva Kumar accepted-paper analysis (open code, full corpus, peer-reviewable). **Medium** for the SciAm "1 percent" prevalence estimate (methodology depends on stylistic-marker proxies).

## 5. GPT-Generated Papers in Scholarly Indexes

**Haider et al. (2024), *HKS Misinformation Review*.** A team led by Jutta Haider (Swedish School of Library and Information Science, University of Borås) scraped Google Scholar for telltale ChatGPT phrases ("as of my last knowledge update", "I don't have access to real-time data") and identified 139 papers fraudulently using GPT without disclosure (227 retrieved, 88 excluded as legitimate or acknowledged use). Distribution: 19 in indexed journals, 89 in non-indexed journals, 19 in university student-paper databases, 12 in pre-print archives. Subject distribution: computing (23 percent), environment (19.5 percent), health (14.5 percent). Dissemination: health-related fabricated papers appeared on 20 unique domains (46 URLs); environmental papers on 26 domains (56 URLs), with ResearchGate, ORCID, IEEE, and Frontiers among the top platforms ([source](https://misinforeview.hks.harvard.edu/article/gpt-fabricated-scientific-papers-on-google-scholar-key-features-spread-and-implications-for-preempting-evidence-manipulation/)). The authors introduce the concept of **"evidence hacking"** as "the strategic and coordinated malicious manipulation of society's evidence base." They note: "Google Scholar presents results from quality-controlled and non-controlled citation databases on the same interface."

For a library journal audience, this is the most directly actionable finding in the 2024-2026 corpus: it locates the contamination within the discovery layer that researchers and patrons rely on most heavily.

**Confidence: High.** Peer-reviewed, full methodology and Cohen's kappa of 0.806 reported, full data deposited at Harvard Dataverse (doi:10.7910/DVN/WUVD8X).

## 6. Domain-Specific Rates: Legal and Medical

**Legal: free-form queries.** Dahl, Magesh, Suzgun, and Ho (2024), in *Large Legal Fictions: Profiling Legal Hallucinations in Large Language Models* (Journal of Legal Analysis vol. 16, no. 1, pp. 64-93), tested LLMs on specific verifiable questions about random federal court cases. **GPT-4 hallucinated on 58 percent of legal queries; Llama 2 on 88 percent.** They also document that LLMs "frequently fail to correct user misconceptions in counterfactual scenarios" and "cannot consistently detect when producing inaccurate legal content," with the highest risk on "unrepresented individuals relying on these tools." Confidence: High (primary peer-reviewed venue, full text accessible via institutional channels). Pre-print: [arxiv.org/abs/2401.01301](https://arxiv.org/abs/2401.01301).

**Legal: RAG-grounded commercial tools.** Magesh, Surani, Dahl, Suzgun, Manning, and Ho (2024), in the first preregistered evaluation of commercial AI legal-research tools, found **LexisNexis (Lexis+ AI), Thomson Reuters (Westlaw AI-Assisted Research), and Ask Practical Law AI each hallucinate between 17 and 33 percent of the time**, despite vendor claims of hallucination elimination. Submitted May 30, 2024 ([source](https://arxiv.org/abs/2405.20362)). The implication for library research-services teams: retrieval-augmentation reduces but does not eliminate the problem, even in tools marketed specifically to legal professionals.

**Legal: documented harm.** *Mata v. Avianca, Inc.*, 678 F. Supp. 3d 443 (S.D.N.Y. 2023), is the canonical case. Lawyers Steven A. Schwartz and Peter LoDuca of Levidow, Levidow & Oberman filed a motion containing fabricated case citations (including *Varghese v. China Southern Airlines*, *Martinez v. Delta Air Lines*, and *Zicherman v. Korean Air Lines*) generated by ChatGPT; when challenged, counsel asked ChatGPT to verify and the model "assured that the cases 'indeed exist' and 'can be found in reputable legal databases such as LexisNexis and Westlaw.'" Judge P. Kevin Castel found the attorneys "abandoned their responsibilities when they submitted non-existent judicial opinions with fake quotes and citations created by the artificial intelligence tool ChatGPT, then continued to stand by the fake opinions after judicial orders called their existence into question," and imposed a $5,000 sanction under Federal Rule of Civil Procedure 11 ([source](https://www.cbsnews.com/news/chatgpt-judge-fines-lawyers-who-used-ai/); summary: [Mata v. Avianca, Wikipedia](https://en.wikipedia.org/wiki/Mata_v._Avianca,_Inc.)). The American Bar Association issued its first formal generative-AI ethics guidance in July 2024, in part as a response. Confidence: High for the case facts and sanction; Medium for the ABA causal claim.

**Medical citation fabrication.** Beyond the Linardon and Chelli studies in §3, a 2023 *Internal and Emergency Medicine* commentary by Bhattacharyya et al. (Springer, doi:10.1007/s11739-023-03484-5) documents specific reference-fabrication patterns in internal-medicine queries ([source](https://link.springer.com/article/10.1007/s11739-023-03484-5)). The article was inaccessible during this review (HTTP 303), so confidence on the specific numbers is Medium.

**Web-grounded chatbot citation accuracy.** Jaźwińska and Chandrasekar (2025) at Columbia Journalism Review's Tow Center tested eight generative search tools (ChatGPT Search, Perplexity, Perplexity Pro, DeepSeek Search, Microsoft Copilot, Grok-2, Grok-3, Google Gemini) across 1,600 queries (20 publishers x 10 articles x 8 chatbots), asking each tool to identify the headline, publisher, publication date, and URL of supplied article excerpts. Across all platforms, chatbots **"provided incorrect answers to more than 60 percent of queries"**. Per-tool error rates ranged from **37 percent for Perplexity to 94 percent for Grok-3**. DeepSeek misattributed sources 115 of 200 times; Grok-3 produced 154 broken-URL citations of 200 tested. ChatGPT signaled lack of confidence on only 15 of 200 incorrect responses, illustrating the pattern of confidently-wrong output ([source](https://www.cjr.org/tow_center/we-compared-eight-ai-search-engines-theyre-all-bad-at-citing-news.php)). Confidence: High.

## 7. Library and Information-Science Response

**Duke University Libraries.** Hannah Rozear and Sarah Park's March 2023 post "ChatGPT and Fake Citations" was among the earliest librarian-authored documentation of the problem, observing that "if you try to find these sources through Google or the library, you will turn up NOTHING" and that ChatGPT's "core strength lies in recognizing language patterns, not in reading and analyzing lengthy scholarly texts" ([source](https://blogs.library.duke.edu/blog/2023/03/09/chatgpt-and-fake-citations/)). Rozear's January 5, 2026 follow-up, "It's 2026. Why Are LLMs Still Hallucinating?", argues the problem is architectural and behavioral, not a scaling problem. She identifies four causes: benchmarks reward confident guessing, training-data quality is uneven, design prioritizes user satisfaction (sycophancy), and pragmatic-language understanding remains weak. Local survey data: 94 percent of Duke students believe AI accuracy varies by subject; 90 percent want clearer transparency about limitations ([source](https://blogs.library.duke.edu/blog/2026/01/05/its-2026-why-are-llms-still-hallucinating/)). Confidence: High for the library-blog content; the survey numbers are local and not generalizable.

**Implication for library workflows.** Three concrete patterns emerge from the 2024-2026 evidence:

1. **Verification cannot be optional.** Even at single-digit fabrication rates, a 20-citation literature review will contain expected fakes; at 19.9 percent (Linardon) or 28.6 percent (Chelli, GPT-4) it will contain many.
2. **DOI-based verification is necessary but insufficient.** Linardon et al. show 64 percent of fabricated citations with DOIs link to real but unrelated papers. A live DOI does not guarantee correct attribution; the title-author-year tuple must also be checked against the resolved record.
3. **Discovery layers are already contaminated.** Haider et al. demonstrate that Google Scholar surfaces fabricated GPT-generated papers without distinguishing them from quality-controlled output. Reference services that rely on "Google Scholar as first stop" workflows now require an additional verification step.

## 8. Confidence Assessment

| Claim | Confidence | Notes |
|---|---|---|
| Fabricated-citation rate of 19.9 percent for GPT-4o in mental-health reviews | High | Peer-reviewed JMIR Mental Health, Linardon et al. 2025 |
| Hallucination rates of 28.6 percent (GPT-4) and 91.4 percent (Bard) on systematic-review citations | High | Peer-reviewed JMIR, Chelli et al. 2024 |
| 50 plus hallucinated citations in 300 sampled ICLR 2026 submissions | High | Methodology and human-verification step documented |
| 1 in 29 accepted ICLR 2026 papers contain at least one likely hallucinated reference | High | Open code, full corpus reproducible |
| 100 plus hallucinated citations across 53 NeurIPS 2025 accepted papers | High | Methodology and human-verification step documented |
| 139 GPT-fabricated papers indexed by Google Scholar | High | Peer-reviewed *HKS Misinformation Review*, full data deposited |
| GPT-4 hallucinates 58 percent on free-form legal queries (Llama 2: 88 percent) | High | Peer-reviewed *Journal of Legal Analysis* |
| Commercial RAG legal tools hallucinate 17 to 33 percent | High | Preregistered Stanford RegLab study |
| *Mata v. Avianca* case facts and $5,000 sanction | High | Court record + CBS News + Wikipedia summary |
| HHEM-2.3 leaderboard top performers (Finix S1 32B 1.8 percent) | High | Live leaderboard with reproducible methodology |
| FACTS Grounding benchmark structure (1,719 examples, 32K-token docs) | High | Primary DeepMind source |
| OpenAI o3 hallucinates 33 percent on PersonQA, 51 percent on SimpleQA | Medium-High | OpenAI system card primary source + TechCrunch coverage |
| 1 percent of all 2023 scientific articles show AI involvement | Medium | *Scientific American* synthesis; stylistic-marker proxies |
| Web-grounded chatbot citation accuracy 37 to 94 percent erroneous | High | Columbia Journalism Review Tow Center direct, 1,600 queries |

## 9. Open Questions

1. **Cross-discipline replication.** Most experimental evidence on fabricated-citation rates concentrates in medicine and computer science. A 2026-vintage replication in humanities, social sciences, and law-review citation contexts would strengthen the literature considerably.
2. **Does grounded retrieval close the gap?** Magesh et al. show RAG legal tools at 17 to 33 percent hallucination. A library-context replication using common discovery-layer plug-ins (Primo, Summon, EBSCO Discovery, Scopus AI) is, to my knowledge, not yet published.
3. **Detector robustness.** GPTZero, RefChecker, and similar tools rely on LLM-judge or database-cross-reference approaches. The false-positive and false-negative rates of these detectors at scale are not yet independently characterized.
4. **Reasoning-model paradox in scholarly contexts.** The predecessor report documents that reasoning models hallucinate more on grounded summarization. Whether that pattern transfers to citation generation specifically is an empirically open question.
5. **Long-tail venues.** Haider et al. show that fabricated papers are concentrated in non-indexed journals and pre-print archives. The intersection of predatory-publishing dynamics and AI-generated content has not been quantified at corpus scale.

## 10. Sources

### Peer-Reviewed Journal Articles

- Chelli, M., Descamps, J., Lavoué, V., Trojani, C., Azar, M., Deckert, M., Raynier, J.-L., Clowez, G., Boileau, P., & Ruetsch-Chelli, C. (2024). Hallucination Rates and Reference Accuracy of ChatGPT and Bard for Systematic Reviews: Comparative Analysis. *Journal of Medical Internet Research, 26*, e53164. [https://www.jmir.org/2024/1/e53164](https://www.jmir.org/2024/1/e53164)
- Dahl, M., Magesh, V., Suzgun, M., & Ho, D. E. (2024). Large Legal Fictions: Profiling Legal Hallucinations in Large Language Models. *Journal of Legal Analysis, 16*(1), 64-93. Pre-print: [https://arxiv.org/abs/2401.01301](https://arxiv.org/abs/2401.01301)
- Haider, J., Söderström, K. R., Ekström, B., & Rödl, M. (2024). GPT-fabricated scientific papers on Google Scholar: Key features, spread, and implications for preempting evidence manipulation. *Harvard Kennedy School Misinformation Review*. [https://misinforeview.hks.harvard.edu/article/gpt-fabricated-scientific-papers-on-google-scholar-key-features-spread-and-implications-for-preempting-evidence-manipulation/](https://misinforeview.hks.harvard.edu/article/gpt-fabricated-scientific-papers-on-google-scholar-key-features-spread-and-implications-for-preempting-evidence-manipulation/)
- Linardon, J., Jarman, H. K., McClure, Z., Anderson, C., Liu, C., & Messer, M. (2025). Influence of Topic Familiarity and Prompt Specificity on Citation Fabrication in Mental Health Research Using Large Language Models: Experimental Study. *JMIR Mental Health, 12*, e80371. [https://mental.jmir.org/2025/1/e80371](https://mental.jmir.org/2025/1/e80371)
- Magesh, V., Surani, F., Dahl, M., Suzgun, M., Manning, C. D., & Ho, D. E. (2024). Hallucination-Free? Assessing the Reliability of Leading AI Legal Research Tools. Pre-print: [https://arxiv.org/abs/2405.20362](https://arxiv.org/abs/2405.20362)

### Industry Benchmarks and Tools

- Google DeepMind. (2024, December 17). FACTS Grounding: A new benchmark for evaluating the factuality of large language models. [https://deepmind.google/discover/blog/facts-grounding-a-new-benchmark-for-evaluating-the-factuality-of-large-language-models/](https://deepmind.google/discover/blog/facts-grounding-a-new-benchmark-for-evaluating-the-factuality-of-large-language-models/)
- OpenAI. (2025, April 16). OpenAI o3 and o4-mini system card. [https://cdn.openai.com/pdf/2221c875-02dc-4789-800b-e7758f3722c1/o3-and-o4-mini-system-card.pdf](https://cdn.openai.com/pdf/2221c875-02dc-4789-800b-e7758f3722c1/o3-and-o4-mini-system-card.pdf)
- Russinovich, M., & Siva Kumar, R. S. (2026, April 22). RefChecker analysis of ICLR 2026 accepted papers. GitHub repository. [https://github.com/markrussinovich/refchecker-iclr2026](https://github.com/markrussinovich/refchecker-iclr2026)
- Vectara. (2026, April 20). Hallucination Leaderboard (HHEM-2.3). [https://github.com/vectara/hallucination-leaderboard](https://github.com/vectara/hallucination-leaderboard)

### Investigative and Conference-Audit Reports

- GPTZero. (2025, December 5). GPTZero uncovers 50 plus hallucinations in ICLR 2026. [https://gptzero.me/news/iclr-2026/](https://gptzero.me/news/iclr-2026/)
- GPTZero. (2026, January 21). GPTZero finds 100 new hallucinations in NeurIPS 2025 accepted papers. [https://gptzero.me/news/neurips/](https://gptzero.me/news/neurips/)

### News and Trade-Press Coverage

- Associated Press. (2023, June 23). Lawyers fined for filing bogus case law created by ChatGPT. *CBS News*. [https://www.cbsnews.com/news/chatgpt-judge-fines-lawyers-who-used-ai/](https://www.cbsnews.com/news/chatgpt-judge-fines-lawyers-who-used-ai/)
- Carrigan, M. (2025, December 16). Startup investigation reveals 50 peer-reviewed papers contained AI hallucinated citations. *BetaKit*. [https://betakit.com/start-up-investigation-reveals-50-peer-reviewed-papers-contained-hallucinated-citations/](https://betakit.com/start-up-investigation-reveals-50-peer-reviewed-papers-contained-hallucinated-citations/)
- Jaźwińska, K., & Chandrasekar, A. (2025, March 6). AI search has a citation problem: We compared eight AI search engines. They're all bad at citing news. *Columbia Journalism Review Tow Center*. [https://www.cjr.org/tow_center/we-compared-eight-ai-search-engines-theyre-all-bad-at-citing-news.php](https://www.cjr.org/tow_center/we-compared-eight-ai-search-engines-theyre-all-bad-at-citing-news.php)
- Stokel-Walker, C. (2024, May 1). AI chatbots have thoroughly infiltrated scientific publishing. *Scientific American*. [https://www.scientificamerican.com/article/chatbots-have-thoroughly-infiltrated-scientific-publishing/](https://www.scientificamerican.com/article/chatbots-have-thoroughly-infiltrated-scientific-publishing/)
- Zeff, M. (2025, April 18). OpenAI's new reasoning AI models hallucinate more. *TechCrunch*. [https://techcrunch.com/2025/04/18/openais-new-reasoning-ai-models-hallucinate-more/](https://techcrunch.com/2025/04/18/openais-new-reasoning-ai-models-hallucinate-more/)
- *Fortune*. (2026, January 21). NeurIPS papers contained 100 plus AI-hallucinated citations, new report finds. [https://fortune.com/2026/01/21/neurips-ai-conferences-research-papers-hallucinations/](https://fortune.com/2026/01/21/neurips-ai-conferences-research-papers-hallucinations/)

### Library and Information-Science Sources

- Rozear, H. (2026, January 5). It's 2026. Why are LLMs still hallucinating? *Duke University Libraries Blog*. [https://blogs.library.duke.edu/blog/2026/01/05/its-2026-why-are-llms-still-hallucinating/](https://blogs.library.duke.edu/blog/2026/01/05/its-2026-why-are-llms-still-hallucinating/)
- Rozear, H., & Park, S. (2023, March 9). ChatGPT and fake citations. *Duke University Libraries Blog*. [https://blogs.library.duke.edu/blog/2023/03/09/chatgpt-and-fake-citations/](https://blogs.library.duke.edu/blog/2023/03/09/chatgpt-and-fake-citations/)

### Legal Cases

- *Mata v. Avianca, Inc.*, 678 F. Supp. 3d 443 (S.D.N.Y. 2023). Summary: [https://en.wikipedia.org/wiki/Mata_v._Avianca,_Inc.](https://en.wikipedia.org/wiki/Mata_v._Avianca,_Inc.)

### Companion Report (Pete's Corpus)

- Stergion, P. (2026, April 1). AI Hallucination Metrics in 2026. [https://github.com/pete-builds/research-reports/blob/main/ai-hallucination-metrics-2026.md](https://github.com/pete-builds/research-reports/blob/main/ai-hallucination-metrics-2026.md)

## Update History

- 2026-04-27 17:30 ET. Initial publication. Fresh-mode research focused on scholarly and academic-library context, complementing the existing technical metrics report.
- 2026-04-27 17:55 ET. Verify-pass corrections: replaced two unsupported aggregator URLs with primary sources (OpenAI o3/o4-mini system card and TechCrunch for the o3 PersonQA and SimpleQA figures; Columbia Journalism Review Tow Center direct for the eight-search-engine citation accuracy figures). Added CBS News (AP) as a second independent source for *Mata v. Avianca*. Added single-source-confirmation note to the RefChecker finding.

## How This Report Was Generated

Compiled by the Research agent (Claude Opus 4.7, 1M context) on 2026-04-27. Searches conducted via SearXNG (`mcp__searxng__search_deep`) across multiple engines, with WebFetch verification of every cited source's URL. Every quoted percentage was extracted directly from the cited primary or secondary source rather than from search snippets. Two URLs in the source set are paywalled or block direct fetching (Springer *Internal and Emergency Medicine* and *Nature Scientific Reports*); the report flags these inline and tags affected confidence as Medium.
