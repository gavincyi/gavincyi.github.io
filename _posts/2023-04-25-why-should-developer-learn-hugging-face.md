---
layout: post
title: Why Every (Quant) Developer Should Learn Hugging Face 🤗
subtitle: 
cover-img: https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/datasets/datasets_logo.png
tags: [softwaredevelopment, ai, ml, llm, huggingface, nlp]
comments: true
---

Why should every (Quant) developer learn Hugging Face? Not because every developer must learn ML / AI / NLP / LLM (though they should learn why AI may replace some of their jobs in the future). Not because we should learn Hugging Face to automate our jobs (though I believe it can bring in tons of amazing ideas). Just because Hugging Face has built an amazing platform for data scientists to contribute with their models, datasets and tasks. I think it is an interesting case study of what composes a successful platform.

## What is Hugging Face 🤗? 

Hugging Face takes hold a platform for the NLP community to share models and datasets, and even build machine learning demo web applications. Users can take any transformer models from the platform and use it to solve the specific tasks, just in a few lines!


```
>>> from transformers import pipeline
>>> classifier = pipeline('sentiment-analysis')
>>> classifier('We are very happy to show you the 🤗 Transformers library.')
[{'label': 'POSITIVE', 'score': 0.9998}]
```

For example, the above example first downloads a pre-trained model and its tokenizer, and then the text preprocessing and prediction are grouped and completed in a single pipeline.

For starters, text preprocessing and reusing the pretrained model could fall into a steep learning curve. If you look into the tensorflow [tutorial](https://www.tensorflow.org/tutorials/keras/text_classification) on text classification, it is still a long page to follow so as to fit a simple pre-trained model.

## Model hub

In the model hub, I can easily grab a text processing model by topics or languages, and use it in my local environment. For example, I can find a whisper Cantonese recognition [model](https://huggingface.co/alvanlii/whisper-small-cantonese) in the hub. 

Take another example. I searched for an ESG model in the hub and found a categorization model on ESG topics

![unknown](https://user-images.githubusercontent.com/10500805/234402585-cf55e701-8198-4fe4-86d3-cb8c8a12db56.png)


Then I pulled it into my local environment

```
from transformers import BertTokenizer, BertForSequenceClassification, pipeline

finbert = BertForSequenceClassification.from_pretrained('yiyanghkust/finbert-esg-9-categories',num_labels=9)
tokenizer = BertTokenizer.from_pretrained('yiyanghkust/finbert-esg-9-categories')
nlp = pipeline("text-classification", model=finbert, tokenizer=tokenizer)
```

retrieved the latest news titles from Microsoft (MSFT), NIVIDA (NVDA) and Ford Motor (F)

```
def get_news(ticker):
  import yfinance as yf
  ticker = yf.Ticker(ticker)
  news = ticker.news
  news_titles = [item['title'] for item in news]
  return news_titles

msft_news = get_news('msft')
nvda_news = get_news('nvda')
ford_news = get_news('f')
```

computed the ESG scores on each category

```
nlp(msft_news[0], return_all_scores=True)
# Output:
# [[{'label': 'Climate Change', 'score': 0.016919471323490143},
#   {'label': 'Natural Capital', 'score': 0.0022099234629422426},
#   {'label': 'Pollution & Waste', 'score': 0.006400629412382841},
#   {'label': 'Human Capital', 'score': 0.005831864662468433},
#   {'label': 'Product Liability', 'score': 0.013546546921133995},
#   {'label': 'Community Relations', 'score': 0.027623875066637993},
#   {'label': 'Corporate Governance', 'score': 0.013731945306062698},
#   {'label': 'Business Ethics & Values', 'score': 0.0053318822756409645},
#   {'label': 'Non-ESG', 'score': 0.9084038138389587}]]
```

and finally averaged out the ESG scores for each news title in the three companies

```
pd.concat({
   'MSFT': pd.DataFrame([{i['label']: i['score'] for i in nlp(n, return_all_scores=True)[0]} for n in msft_news]).mean(),
   'NVDA': pd.DataFrame([{i['label']: i['score'] for i in nlp(n, return_all_scores=True)[0]} for n in nvda_news]).mean(),
   'F': pd.DataFrame([{i['label']: i['score'] for i in nlp(n, return_all_scores=True)[0]} for n in ford_news]).mean(),
}, axis=1)
```

![unknown](https://user-images.githubusercontent.com/10500805/234402902-c784c458-27b7-4125-89b3-a38c6e474ce3.png)


Coincidently, the result aligns well with the running Refinitiv ESG [scores](https://www.refinitiv.com/en/sustainable-finance/esg-scores). For example, Ford has the highest climate change score while Microsoft has the highest ESG score overall.

|MSFT|NVDA|F|
|:---:|:---:|:---:|
|![Microsoft Corp ESG](https://user-images.githubusercontent.com/10500805/234403081-f13ba571-bd16-492a-b247-93b2f2267dc6.png)|![unknown](https://user-images.githubusercontent.com/10500805/234403159-c918db11-771d-4390-a8fd-c3a0aba7f662.png)|![unknown](https://user-images.githubusercontent.com/10500805/234403200-8f2b5c4c-8a25-44bb-be77-47a7ed4b713b.png)|

It sounds like an open source pre-trained model may be capable of producing a similar result as the provider's millions-dollar-subscription-required data. I believe so but I haven't run analysis yet on the entire universe.

##### *If you want to attract users, your product should be solving a millions dollar problem. At least it should sound like it is.*

## Pre-trained model not an end

Tokenizers and pre-trained models can be further trained and reiterated as new ones. It means models on the hub could be represented as "ancestor trees". 

Users can avoid paying hundreds of dollars in cloud platforms to train models from scratch, but re-train the existing models to examine the model improvement, and in turn contribute back into the model hub.

```
from transformers import AutoModelForSequenceClassification

model = AutoModelForSequenceClassification.from_pretrained("bert-base-cased", num_labels=5)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=small_train_dataset,
    eval_dataset=small_eval_dataset,
    compute_metrics=compute_metrics,
)
trainer.train()
```

Pre-trained and open source tokenizers and models is not a new concept. Tensorflow has provided APIs to retrieve academic level trained models. But Hugging Face now provides a loop for every data scientist, even amateurs, to contribute back into the platform. We have Github for open source projects but it doesn't provide a bespoken framework for data scientists to socialise their trained models. In the next section, we will go through how Hugging Face creates an amazing framework for sharing models.

##### *If you want your platform to grow exponentially, allow users to contribute back.*

## Boundary is good

Hugging Face provides a standard framework to share models in the hub. The flow is generally as follows

1. Import an existing model with the corresponding tokenizer
2. Fine tune your model by training with other datasets
3. Push the trained model back to the hub

Each model is managed in users' git repository.

For example, a PyTorch model is trained and then the method `push_to_hub` updates the model in the model repository.

```
model.push_to_hub("test_model")
```

The model repository shows the PyTorch model file `pytorch_model.bin` and model arguments `config.json` are updated. Therefore, data scientists can focus on model evaluation and training data, and fully rely on the standardized framework to manage models.

Most data scientists nailed in one of the ML engines, e.g. PyTorch /  Tensorflow, and the transformers library paves the way for model conversion between supported engines. For example, you can always load a PyTorch model from remote or local, and then convert them with the desired engine.

```
from transformers import TFDistilBertForSequenceClassification, FlaxDistilBertForSequenceClassification

tf_model = TFDistilBertForSequenceClassification.from_pretrained("gavincyi/test_model", from_pt=True)
jax_model = FlaxDistilBertForSequenceClassification.from_pretrained("gavincyi/test_model", from_pt=True)
```

Then, push back to the hub right after.

```
tf_model.push_to_hub("gavincyi/test_model")
jax_model.push_to_hub("gavincyi/test_model")
```

The above code snippet updates the model repository with TF model file `tf_model.h5` and JAX model file `flax_model.msgpack`.

Last but not least, the model card format is well defined. For example, licence and datasets can be selected from the autocomplete list. 

![unknown](https://user-images.githubusercontent.com/10500805/234403332-53e00256-fb17-4cf0-9245-d42d66d0a3de.png)

`README.md` is provided with a template to fill in the detailed descriptions and helps the interactions between authors and users in respect of the sufficient documentation.

![unknown](https://user-images.githubusercontent.com/10500805/234403384-3a04aa74-4afd-4fe6-a6d5-c8efeed1397f.png)


##### *Do not worry about setting up boundaries for user experience. Sometimes we need to help them define it well*

## Collaboration to save lives

In March, Hugging Face blog published a [post](https://huggingface.co/blog/using-ml-for-disasters) to describe how ML collaboration can touch the ground to solve immediate problems after earthquakes near South Eastern Turkey severely hit the cities. Hugging Face became a stage for itself to test how rapidly they can build applications on the real world requests.

Primarily, survivors posted screenshots of texts with their addresses and what they needed (including rescue) on social media, and Hugging Face team built an app to recognize text with open source OCR tools, tokenize texts with [bert-base-turkish-cased](https://huggingface.co/dbmdz/bert-base-turkish-cased) and call a customized model to extract addresses.

Primarily, survivors posted screenshots of texts with their addresses and what they needed (including rescue) on social media, and Hugging Face team built an app to recognize text with open source OCR tools, tokenize texts with [bert-base-turkish-cased](https://huggingface.co/dbmdz/bert-base-turkish-cased) and call a customized model to extract addresses.

![unknown](https://user-images.githubusercontent.com/10500805/234403460-77b558cd-3b54-4bd4-92a8-afc21ac13a43.png)

They created a volunteer contribution organisation ["deprem-ml"](https://huggingface.co/deprem-ml) and posted a [leaderboard](https://huggingface.co/spaces/deprem-ml/intent-leaderboard) to find out the best turkish BERT text classification. Later, they extended their model so that it could be evaluated and crowd-sourced through a labelling interface.

![unknown](https://user-images.githubusercontent.com/10500805/234403609-aa44dba4-03c6-4dc0-a410-933dc1301fb4.png)


## Open AI framework

It just came across when I was reading Peter Cotton's book [Microprediction](https://mitpress.mit.edu/9780262047326/microprediction/). It introduces an idea that, with a brilliant open AI framework, real life problems can be crumbled down into tons of small problems which can be resolved by crowdsourced AI models (called "oracle" in the book). Though Peter just outlined an ambitious philosophical but yet practical idea to challenge the AI revolution, it echoes well with the growth of Hugging Face Hub for the ML community in providing "autonomous micromanagers". 

## Conclusion

I truly believe the platform has a huge potential to grow, and its operation model should be referenced in a text book. Similar to Github, Hugging Face makes ML models more accessible and flatten the learning curve for the public. NLP and text processing models should no longer be luxury toys of enterprises but, in respect to open source libraries, can be crowdsourced and contributed at an exponential rate. Well, to some extent, Hugging Face should be labelled as an ESG company, right?

### Reference

[Using Machine Learning to Aid Survivors and Race through Time](https://huggingface.co/blog/using-ml-for-disasters)

[Fine-tune a pretrained model](https://huggingface.co/docs/transformers/training)

[Share a model](https://huggingface.co/docs/transformers/model_sharing)

[Colab Notebook - Hugging Face Share Model](https://colab.research.google.com/drive/1sGbPW-fdStydez2K3l5vWDuDPNUvXxgP?usp=sharing)
