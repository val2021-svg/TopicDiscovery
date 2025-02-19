# Topic Modeling

Link to the original repository: https://github.com/vrjkmr/arxiv-topic

Befor running it you may have to do:
python3 -m spacy download en_core_web_sm python3 -m nltk.downloader stopwords

### For now:

For the moment we've separated the catalog in courses with the CreateCourseFolder.ipynb.

With the files saved we can read them and pass them trough our model, getting a coherence of: 0.3803.

After reading, we perform the following steps: 
 [1/9] Removing headings...
 [2/9] Removing gradings, resources and course support...
 [3/9] Removing LaTex equations...
 [4/9] Removing newlines and extra spaces...
 [5/9] Tokenizing documents...
 [6/9] Removing stopwords...
 [7/9] Identifying n-gram phrases...
 [8/9] Lemmatizing...
 [9/9] Removing common words...

To do that, we had to make some changes to the files: Dataset preparation.ipynb, dataset.py, preprocess.py and utils.py.

## Original Readme:

This repository contains the code for a [Latent Dirichlet Allocation (LDA)](https://www.jmlr.org/papers/volume3/blei03a/blei03a.pdf) topic model built and trained on the abstracts of ~160,000 ML-related research papers from the [ArXiv.org dataset](https://www.kaggle.com/Cornell-University/arxiv) on Kaggle.

This model can be used to detect topics in any given arXiv paper related to machine learning. To illustrate, shown below is an example of the model's ability to predict topics present in the seminal paper ["Attention Is All You Need"](https://arxiv.org/abs/2001.01595) by Vaswani et al. (2017).

```
Paper
-----
"Attention Is All You Need" by Vaswani et al. (2017)

Abstract
--------
  The dominant sequence transduction models are based on complex recurrent or
convolutional neural networks in an encoder-decoder configuration. The best
performing models also connect the encoder and decoder through an attention
mechanism. We propose a new simple network architecture, the Transformer, based
solely on attention mechanisms, dispensing with recurrence and convolutions
entirely. Experiments on two machine translation tasks show these models to be
superior in quality while being more parallelizable and requiring significantly
less time to train. Our model achieves 28.4 BLEU on the WMT 2014
English-to-German translation task, improving over the existing best results,
including ensembles by over 2 BLEU. On the WMT 2014 English-to-French
translation task, our model establishes a new single-model state-of-the-art
BLEU score of 41.8 after training for 3.5 days on eight GPUs, a small fraction
of the training costs of the best models from the literature. We show that the
Transformer generalizes well to other tasks by applying it successfully to
English constituency parsing both with large and limited training data.

Predicted topics
----------------
[('Deep learning', 0.7827023),
 ('Natural language processing', 0.18202062),
 ('ML-related terms?', 0.022977384)]
```

### Motivation

You know how when reading research papers, the first thing we read is the abstract? The abstract helps us (as humans) get a general sense of what different topics are explored in any given paper. But what if we can train an unsupervised model to automatically "categorize" papers for us?

In this project, my ultimate goal was to build an clustering model that can:

1. Identify salient trends and sub-topics in machine learning research today, and
2. Automatically predict the topic(s) explored in any given paper simply by looking at its abstract.

### Project structure

This project is organized as follows.

```
.
├── dataset.py                          # script containing the dataset class
├── model.py                            # script containing the topic model class
├── preprocess.py                       # script containing the text preprocessor class
├── utils.py                            # script containing helper functions
└── notebooks/
    ├── Dataset preparation.ipynb       # notebook to build the arXiv abstracts dataset
    ├── Inference.ipynb                 # notebook illustrating how to predict topics of papers
    └── Training.ipynb                  # notebook to train and tune LDA models
└── README.md
```

### Results: Topics

The final model achieved a `c_v` coherence score of 50.2%. While this score is quite low (an ideal coherence score tends to be around 60-75%), the model was able to detect some interesting topic clusters, a few of which are listed below. Note that while the topic terms were generated by the LDA model, the topic titles themselves are subjective, since they were added by me after looking at the term distribution for each of the topics.

1. **Natural language processing:** "text", "knowledge", "language", "information", "semantic"
2. **Probability and inference:** "model", "distribution", "inference", "bayesian", "parameter"
3. **Computer vision:** "image", "object", "segmentation", "detection", "video"
4. **Recommendation systems:** "user", "group", "item", "preference", "product"
5. **Sequences and time-series:** "time", "dynamic", "series", "sequence", "temporal"
6. **Reinforcement learning:** "agent", "policy", "environment", "game", "action"

### Model inference

To predict which topics might be related to any paper on arXiv, simply build a `TopicModel` object, scrape the abstract section, and pass in the raw text into the model's `predict()` method. The output is an ordered list of tuples, with each tuple holding the topic name and the likelihood of the topic's presence in the paper.

```python
from model import TopicModel
from utils import scrape_arxiv_abstract

lda_filepath = "./models/model_001"
dataset_filepath = "./data/dataset.obj"
topic_model = TopicModel(lda_filepath, dataset_filepath)

# Optional: Set topic names
topic_model.set_topic_names([...])

# Paper: "Personalized Re-ranking for Recommendation" by Pei et al. (2019)
paper_url = "https://arxiv.org/abs/1904.06813"
abstract = scrape_arxiv_abstract(paper_url)
predictions = topic_model.predict(abstract)
print(predictions)

'''
Output
------
[('Recommendation systems', 0.32558212),
 ('Deep learning', 0.17530766),
 ('Paper-related terms?', 0.16065647)]
'''
```

### Acknowledgements

- Radim Řehůřek's [tips](https://radimrehurek.com/gensim/auto_examples/tutorials/run_lda.html) on building Gensim LDA models
- Cornell University's [arXiv.org dataset](https://www.kaggle.com/Cornell-University/arxiv) hosted on Kaggle
