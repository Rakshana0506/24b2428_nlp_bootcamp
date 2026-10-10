# NLP Selection Bootcamp report

Rakshana Sundaram (24B2428)

All analysis is on the validation claims.

## Q1. Two claims where BM25 beats t2_general, two where t2_general beats BM25, and why

BM25 beats bge large on claims 673 and 34 and loses to it on 463 and 541. Claims 673 and 34 share rare exact terms with their gold abstracts ('LDL' and 'causal', 'folate' and 'homocysteine'). These get high idf weight, so BM25 ranks the gold abstract 1st while the dense model ranks it 5th and 2nd. In 463 and 541 the abstract says the same thing in different words ('genomic aberrations' vs 'mutational landscapes', 'energy balance' vs 'hypoglycemia'). BM25 only matches generic terms and ranks it 13th and 7th, but the embeddings still put it 1st and 3rd.

## Q2. Which Task 3 component gave the largest validation gain, and its latency cost

The hybrid gave the largest gain. Z score interpolation (0.7 general, 0.15 BM25, 0.15 scientific) reached 0.935 val nDCG@10, slightly above the best single retriever (bge large, 0.933). It adds almost no latency since it only combines scores that are already computed. Reranking with the MedCPT cross encoder did not help. The best setting (top 3) gave 0.926, and reranking deeper made it worse (0.912 at top 100). It also added about 25 ms per claim on top of the 46 ms dense query.

## Q3. Did hard negatives help? Overfitting and how it was detected

Yes. Zero shot bge large had 0.933 val nDCG@10. Fine tuning with hard negatives raised it to 0.946, while random negatives dropped it to 0.915. Random negatives were too easy to separate (first epoch loss 0.04 vs 0.20 for hard), so the model learnt little useful from them. Both runs overfit. Train loss kept falling and train nDCG@10 reached 0.99 (hard) and 0.96 (random) by epoch 4, but val nDCG@10 peaked at epoch 3 for hard and epoch 1 for random. I evaluated on val after every epoch and kept the best checkpoint.

## Q4. Fraction of validation verification errors that are retrieval_miss

8 of 45 validation errors are retrieval_miss, which is 0.18. I tagged a claim as retrieval_miss when a gold abstract was not in the top 3 candidates given to the verifier. The rest are entity_mismatch 20, other 13, needs_multiple_docs 3 and numerical 1. So most errors come after retrieval, where the gold abstract was available but the verifier got the label wrong or also labelled an unrelated abstract. 9 of the entity_mismatch errors are claims with no evidence that still got a verdict. Val F1 is 0.784 (precision 0.778, recall 0.790).

## Q5. The single change to try next, and why

I would add an evidence sentence selection step before the verifier. Each sentence of a candidate abstract would be scored against the claim, and only the top one to three sentences would be passed on, trained using the annotated evidence sentences. Most of my errors happen after retrieval, and the simpler fixes I tried did not beat 0.784 on val (RoBERTa large 0.753, rebalanced training data 0.73 to 0.75, 3 seed ensemble 0.779). Right now the verifier reads the whole abstract, so the one sentence that decides the label gets diluted.
