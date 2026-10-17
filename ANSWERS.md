# Midterm 1: answers

GitHub username: SpanerSan
Session code: MAPLE-66
Fingerprint: 1b98ce4d

> Fill in the three lines above from `my_data/DATA.md`. Do not rename headings or table columns: a script reads the numbers.
> Write only in the empty cells and lines. Percentages as 0.78 or 78 %, money as plain numbers.

## AI-use declaration

I used ChatGPT (GPT-6) extensively to interpret the requirements, reproduce the data-generation logic, perform and verify calculations, compare the retrieval and document-processing options, and prepare the initial full draft of ANSWERS.md, including tables, explanations, decisions, and the operations memo. I used the instructor's Google Colab notebook with my own GitHub username and session code to generate my individual data and confirm the fingerprint. I did not use another student's answers.

Python was used through the AI assistant to compute recall, MRR, Wilson confidence intervals, model and pipeline costs, and answers to the five numerical questions. The submitted figures come from the synthetic individual dataset and the formulas in the assignment README. I understand that I am responsible for the submission and must be able to explain these results during the oral check.

## Part 1: Retrieval and cost decision

### 1a. Metrics (all 60 questions)

| config | recall@1 | recall@3 | recall@5 | MRR@10 |
|---|---|---|---|---|
| A | 0.4000 | 0.5333 | 0.7500 | 0.5107 |
| B | 0.3833 | 0.5333 | 0.8000 | 0.5062 |

### 1b. Recall and 95 % Wilson interval of each candidate (recall at its own k)

| candidate | recall@k | lower | upper |
|---|---|---|---|
| A, k=3 | 0.5333 | 0.4089 | 0.6537 |
| A, k=5 | 0.7500 | 0.6277 | 0.8422 |
| B, k=3 | 0.5333 | 0.4089 | 0.6537 |
| B, k=5 | 0.8000 | 0.6822 | 0.8817 |

### 1c. Annual model cost in USD

| candidate | annual cost |
|---|---|
| A, k=3 | 20180.48 |
| A, k=5 | 26398.06 |
| B, k=3 | 13533.48 |
| B, k=5 | 15319.73 |

### 1d. Decision (at most 150 words)

Chosen candidate: B, k=5

I recommend B with k=5: its annual cost is $15,319.73, below the $23,300 budget, and it retrieves the correct chunk for 48/60 questions (80.0%, Wilson 95% CI 68.2–88.2%). Its confidence interval does not overlap those of affordable A,k=3 or B,k=3 (both 53.3%, CI 40.9–65.4%), supporting a meaningful advantage under the course rule. A,k=5 reaches 75.0% (CI 62.8–84.2%), overlapping B,k=5, so these two are not shown to differ; however, A,k=5 costs $26,398.06 and exceeds the budget. For the chosen B,k=5, per-language recall is EN 16/20 = 80%, RU 15/20 = 75%, and KK 17/20 = 85%. Russian-language users are served worst in this test. The 20 questions per language give limited precision, so I would not treat that ranking as definitive.

### 1e. Next measurement (at most 60 words)

Collect a larger, independently labelled retrieval test set stratified by EN, RU and KK, particularly Russian-language queries. Measure recall@5 and Wilson intervals for B,k=5 and A,k=5, alongside production latency. This could reveal a real language-specific deficit or justify revisiting the configuration if the traffic mix or budget changes.

## Part 2: A document pipeline you can defend

### 2a. Numbers

| pipeline | overall accuracy | weakest cell (language, condition) | accuracy there | lower | upper | annual cost |
|---|---|---|---|---|---|---|
| H | 0.8833 | KK, degraded | 0.8000 | 0.6269 | 0.9050 | 7051.80 |
| L | 0.8333 | KK, degraded | 0.6333 | 0.4551 | 0.7813 | 5400.00 |

Personal-data pages per year sent abroad under H: 408800

### 2b. Memo (at most 300 words)

**To: Head of Operations — Invoice-extraction pipeline decision**

I recommend **L, with mandatory human review of its weak cases**, rather than immediate use of H for personal-data scans.

Across 180 tested invoices per pipeline, H is more accurate overall (159/180 = 88.3%) than L (150/180 = 83.3%), but both perform worst on **Kazakh, degraded** scans. H gets 24/30 right (80.0%; Wilson 95% CI 62.7–90.5%); L gets 19/30 (63.3%; CI 45.5–78.1%). Overall accuracy conceals this disadvantage. The two weak-cell intervals overlap, so the sample does not conclusively establish a difference there.

**Where does the scan go; who stores it; for how long?** H sends each scan to a data centre abroad and is processed by the external vendor. Its storage practices, subcontractors, deletion period, and any copies in logs/backups are unknown and require independent verification and legal/compliance review. With 3,500 daily pages and 32% containing names and ИИН, H would transfer **408,800 personal-data pages/year** abroad. A vendor's assertion is not a compliance review under Kazakhstan's personal-data framework (Law No. 94-V). L processes scans on the bank's Kazakhstan server, under the bank's control. Actual retention is not specified: the bank must approve, document and enforce it.

L costs **$5,400/year**, against H's **$7,051.80/year**. For L, block external egress and audit firewall logs; encrypt scans and restrict role-based access; set and test a 30-day raw-image deletion policy; require two-person verification of every degraded KK extraction; and re-test each language/condition cell monthly against an agreed minimum accuracy threshold before automation. The residual risk is L's high extraction-error rate, plus operational security and review costs not included in the quoted server model.

## Part 3: Explain your own numbers

### Q1

Number: 1.28 (character ratio = 318/305 = 1.04; byte ratio = 639/305 = 2.10; token ratio = 153/57 = 2.68; 2.68/2.10 ≈ 1.28).
Why (at most 40 words): Kazakh uses more UTF-8 bytes, but the token ratio exceeds even the byte ratio by 28%. Thus the cost premium is not solely character encoding; Model M's tokenization also uses more tokens relative to bytes.

### Q2

Number: 35.1% ((768 × 263/256 − 512)/(768 × 263/256) × 100 = 277/789 × 100).
Why (at most 40 words): Only English is truncated: 789 e5 tokens exceed 512. Russian reaches 384 and Kazakh 228. The English tail is invisible to the embedding, so relevant text there cannot influence retrieval.

### Q3

Number: 54.0% (softmax at T=1: top probability ≈ 0.49066; top three cumulative probability ≈ 0.90919; after top-p, 0.49066/0.90919 ≈ 0.53966).
Why (at most 40 words): Top-p removes lower-ranked tokens outside the smallest set reaching 90% cumulative probability and then renormalises. Lower temperature instead sharpens the probability distribution while leaving every token with a nonzero chance.

### Q4

Number: +24 (Y = 73 − 2×12 = 49; X = 75 − 2×25 = 25; Y − X = 24).
Why (at most 40 words): The public leaderboard prefers X because 75 correct exceeds 73. With no penalty for mistakes, such scoring rewards risky guesses. Penalising wrong answers makes Y's cautious abstentions worthwhile.

### Q5

Number: 0.60 ((84/7)/(1,400/70) = 12/20 = 0.60).
Why (at most 40 words): The model has 12 training tokens per parameter versus Chinchilla's 20, so it is under-trained by this ratio. Chinchilla showed that models such as GPT-3 allocated too much compute to parameters and too little to training data.
