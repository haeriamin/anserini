# Anserini Regressions: MS MARCO Passage Ranking

**Model**: [Cohere embed-english-v3.0](https://docs.cohere.com/reference/embed) with HNSW quantized indexes (using pre-encoded queries)

This page describes regression experiments, integrated into Anserini's regression testing framework, using the [Cohere embed-english-v3.0](https://docs.cohere.com/reference/embed) model on the [TREC 2019 Deep Learning Track passage ranking task](https://trec.nist.gov/data/deep2019.html).

In these experiments, we are using pre-encoded queries (i.e., cached results of query encoding).

The exact configurations for these regressions are stored in [this YAML file](../../src/main/resources/regression/dl19-passage-cohere-embed-english-v3-hnsw-int8.yaml).
Note that this page is automatically generated from [this template](../../src/main/resources/docgen/templates/dl19-passage-cohere-embed-english-v3-hnsw-int8.template) as part of Anserini's regression pipeline, so do not modify this page directly; modify the template instead and then run `bin/build.sh` to rebuild the documentation.

From one of our Waterloo servers (e.g., `orca`), the following command will perform the complete regression, end to end:

```bash
python src/main/python/run_regression.py --index --verify --search --regression dl19-passage-cohere-embed-english-v3-hnsw-int8
```

We make available a version of the MS MARCO Passage Corpus that has already been encoded with Cohere embed-english-v3.0.

From any machine, the following command will download the corpus and perform the complete regression, end to end:

```bash
python src/main/python/run_regression.py --download --index --verify --search --regression dl19-passage-cohere-embed-english-v3-hnsw-int8
```

The `run_regression.py` script automates the following steps, but if you want to perform each step manually, simply copy/paste from the commands below and you'll obtain the same regression results.

## Corpus Download

Download the corpus and unpack into `collections/`:

```bash
wget https://rgw.cs.uwaterloo.ca/pyserini/data/msmarco-passage-cohere-embed-english-v3.tar -P collections/
tar xvf collections/msmarco-passage-cohere-embed-english-v3.tar -C collections/
```

To confirm, `msmarco-passage-cohere-embed-english-v3.tar` is 38 GB and has MD5 checksum `6b7d9795806891b227378f6c290464a9`.
With the corpus downloaded, the following command will perform the remaining steps below:

```bash
python src/main/python/run_regression.py --index --verify --search --regression dl19-passage-cohere-embed-english-v3-hnsw-int8 \
  --corpus-path collections/msmarco-passage-cohere-embed-english-v3
```

## Indexing

Sample indexing command, building HNSW indexes:

```bash
target/appassembler/bin/IndexHnswDenseVectors \
  -collection JsonDenseVectorCollection \
  -input /path/to/msmarco-passage-cohere-embed-english-v3 \
  -generator HnswDenseVectorDocumentGenerator \
  -index indexes/lucene-hnsw.msmarco-passage-cohere-embed-english-v3-int8/ \
  -threads 16 -M 16 -efC 100 -noMerge -quantize.int8 \
  >& logs/log.msmarco-passage-cohere-embed-english-v3 &
```

The path `/path/to/msmarco-passage-cohere-embed-english-v3/` should point to the corpus downloaded above.
Upon completion, we should have an index with 8,841,823 documents.

Note that here we are explicitly using Lucene's `NoMergePolicy` merge policy, which suppresses any merging of index segments.
This is because merging index segments is a costly operation and not worthwhile given our query set.

## Retrieval

Topics and qrels are stored [here](https://github.com/castorini/anserini-tools/tree/master/topics-and-qrels), which is linked to the Anserini repo as a submodule.
The regression experiments here evaluate on the 43 topics for which NIST has provided judgments as part of the TREC 2019 Deep Learning Track.
The original data can be found [here](https://trec.nist.gov/data/deep2019.html).

After indexing has completed, you should be able to perform retrieval as follows using HNSW indexes:

```bash
target/appassembler/bin/SearchHnswDenseVectors \
  -index indexes/lucene-hnsw.msmarco-passage-cohere-embed-english-v3-int8/ \
  -topics tools/topics-and-qrels/topics.dl19-passage.cohere-embed-english-v3.jsonl.gz \
  -topicReader JsonIntVector \
  -output runs/run.msmarco-passage-cohere-embed-english-v3.cohere-embed-english-v3.topics.dl19-passage.cohere-embed-english-v3.jsonl.txt \
  -generator VectorQueryGenerator -topicField vector -threads 16 -hits 1000 -efSearch 1000 &
```

Evaluation can be performed using `trec_eval`:

```bash
target/appassembler/bin/trec_eval -m map -c -l 2 tools/topics-and-qrels/qrels.dl19-passage.txt runs/run.msmarco-passage-cohere-embed-english-v3.cohere-embed-english-v3.topics.dl19-passage.cohere-embed-english-v3.jsonl.txt
target/appassembler/bin/trec_eval -m ndcg_cut.10 -c tools/topics-and-qrels/qrels.dl19-passage.txt runs/run.msmarco-passage-cohere-embed-english-v3.cohere-embed-english-v3.topics.dl19-passage.cohere-embed-english-v3.jsonl.txt
target/appassembler/bin/trec_eval -m recall.100 -c -l 2 tools/topics-and-qrels/qrels.dl19-passage.txt runs/run.msmarco-passage-cohere-embed-english-v3.cohere-embed-english-v3.topics.dl19-passage.cohere-embed-english-v3.jsonl.txt
target/appassembler/bin/trec_eval -m recall.1000 -c -l 2 tools/topics-and-qrels/qrels.dl19-passage.txt runs/run.msmarco-passage-cohere-embed-english-v3.cohere-embed-english-v3.topics.dl19-passage.cohere-embed-english-v3.jsonl.txt
```

## Effectiveness

With the above commands, you should be able to reproduce the following results:

| **AP@1000**                                                                                                  | **cohere-embed-english-v3**|
|:-------------------------------------------------------------------------------------------------------------|-----------|
| [DL19 (Passage)](https://trec.nist.gov/data/deep2020.html)                                                   | 0.487     |
| **nDCG@10**                                                                                                  | **cohere-embed-english-v3**|
| [DL19 (Passage)](https://trec.nist.gov/data/deep2020.html)                                                   | 0.690     |
| **R@100**                                                                                                    | **cohere-embed-english-v3**|
| [DL19 (Passage)](https://trec.nist.gov/data/deep2020.html)                                                   | 0.647     |
| **R@1000**                                                                                                   | **cohere-embed-english-v3**|
| [DL19 (Passage)](https://trec.nist.gov/data/deep2020.html)                                                   | 0.850     |

Note that due to the non-deterministic nature of HNSW indexing, results may differ slightly between each experimental run.
Nevertheless, scores are generally within 0.005 of the reference values recorded in [our YAML configuration file](../../src/main/resources/regression/dl19-passage-cohere-embed-english-v3-hnsw-int8.yaml).

## Reproduction Log[*](../../docs/reproducibility.md)

To add to this reproduction log, modify [this template](../../src/main/resources/docgen/templates/dl19-passage-cohere-embed-english-v3-hnsw-int8.template) and run `bin/build.sh` to rebuild the documentation.
