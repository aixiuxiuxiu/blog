---
layout: distill
title: How to Encode Long Text Using Large Language Models? 
description: This blog explores methods for encoding long contexts using large language models, focusing on techniques for Retrieval-Augmented Generation (RAG) and document classification.
tags: RAG Classification
#giscus_comments: true
date: 2024-10-18
featured: false
anthor: Aixiu An

#authors:
#  - name: Aixiu An
    #url: "https://en.wikipedia.org/wiki/Albert_Einstein"
    #affiliations:
    #  name: IAS, Princeton

bibliography: 2024-10-6-longcontext.bib

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).
toc:
  - name: Why Long Context Is So Hard?
    # if a section has subsections, you can add them as follows:
  - name: Encoding Long Contexts in RAG
    subsections:
      - name: Naive chunking
      - name: Late chunking 
  - name: Encoding Long Contexts in Document Classification

    subsections:
      - name: Document Truncation
      - name: Longformer
      - name: Hierarchical Encoding
      - name: Cognize LongTeXts


_styles: >
  .fake-img {
    background: #bbb;
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 12px;
  }
  .fake-img p {
    font-family: monospace;
    color: white;
    text-align: left;
    margin: 12px 0;
    text-align: center;
    font-size: 16px;
  }
---

## Why Long Context Is So Hard?

Over the past few years, large language models (LLMs) have made remarkable progress in extending context length limits. For example, BERT-based models typically support up to 512 tokens, while standard GPT-3 models can handle 2,048 tokens. GPT-4 offers two configurations: one with 8,192 tokens and another with an extended window of 32,768 tokens (32K tokens). Recently, Gemmi announced a 2 million-token context window for Gemini 1.5 Pro. However, using models with larger context windows does not necessarily lead to better performance. I hope this blog helps unravel some of the myths about long contexts and explains some current solutions to address this challenge.

The limitations of the context window stem from the inherent properties of the transformer architecture itself. Most current LLMs, such as GPT and LLama, rely on the transformer architecture and its self-attention mechanism. This mechanism compares each token in the input sequence with every other token, resulting in a quadratic increase in both memory usage and computational cost as the context length grows. To address the limitations imposed by context length, two main research directions have emerged. The first focuses on training models with larger context windows, aiming to extend these limits. Some research has explored fine-tuning LLMs with longer context inputs (<d-cite key='dubey2024llama'></d-cite>, <d-cite key='tworkowski2024focused'></d-cite>), while others have used position extrapolation or interpolation, building on relative rotary positional embeddings (<d-cite key='su2024roformer'></d-cite>) to extend input lengths beyond the model’s original training limits (<d-cite key='press2021train'></d-cite>, <d-cite key='chen2023extending'></d-cite>).

The second approach focuses on improving encoding techniques. Encoding all the information from a multi-page document into a single embedding vector is difficult, if not impossible. Even with a model that has a long context window capable of handling large inputs, encoding everything from multiple pages into one vector may result in the loss of important information. Particularly for retrieval tasks, if you need to extract specific information from a sentence, a large embedding for multi-page text might not effectively capture it. In such cases, chunking long texts into smaller segments while maintaining the dependencies between them offers a more effective approach.



This blog will focus on the second direction, discussing available techniques for encoding long contexts in two different tasks: one in building Retrieval-Augmented Generation (RAG) and the other in document classification.

## Encoding Long Contexts in RAG

Building a RAG system requires integrating internal knowledge bases, which often involves encoding hundreds of pages of documents. Efficiently encoding these documents and facilitating retrieval and generation afterward remain core challenges. Here, I will explain two encoding techniques: naive chunking and late chunking.

### Naive chunking

The naive encoding approach (shown on the left side of the image below) involves splitting the text into chunks beforehand and applying an embedding model to each chunk. A common method for generating a single embedding from each chunk is to use mean pooling on the token-level embeddings, where the embeddings of all tokens are averaged.


 

To split the text, we can use a fixed length (see an implementation in the [OpenAI CookBook](https://cookbook.openai.com/examples/embedding_long_inputs)), though in some cases, splitting at paragraph or sentence boundaries may better preserve the text's meaning. This approach has been implemented in Langchain via the function `RecursiveCharacterTextSplitter` (see this [blog](https://dev.to/eteimz/understanding-langchains-recursivecharactertextsplitter-2846) for more detailed explanations of this function.)



{::nomarkdown}
{% assign jupyter_path = 'assets/jupyter/splitter.ipynb' | relative_url %}
{% capture notebook_exists %}{% file_exists assets/jupyter/splitter.ipynb %}{% endcapture %}
{% if notebook_exists == 'true' %}
  {% jupyter_notebook jupyter_path %}
{% else %}
  <p>Sorry, the notebook you are looking for does not exist.</p>
{% endif %}
{:/nomarkdown}
<div class="caption">
Example of chunking with sentence boundaries
</div>


As a result,  naive chunking allows encoding the entire sequence without cutting until the maximum context window is reached. However, because it encodes each chunk independently, it breaks the dependencies between chunks. This means that each chunk is treated as an independent element, without considering the context before or after it.

### Late chunking 

Late chunking, introduced by Jina AI <d-cite key='gunther2024late'></d-cite>, addresses the contextual issue. Their technique  involves first encoding the entire document with a long context embedding model to produce embeddings for all tokens in the text (or at least as long as possible). Once the token-level embeddings are generated using the long-context embedding model, the text is divided into chunks, and mean pooling is applied to the tokens in each chunk to create chunk-level embeddings.  This ensures that each chunk embedding is informed by the context of the whole document, not just the local text of the chunk.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0 d-flex justify-content-center">
        <figure style="width: 70%; margin: 0 auto; background-color: black; padding: 10px; text-align: center;">
            {% include figure.liquid loading="eager" path="assets/img/latechuncking.png" class="img-fluid rounded z-depth-1" %}
            <figcaption class="text-white text-center mt-2">
                An illustration of the naive chunking strategy (left) and the late chunking strategy (right), from Jina AI 
                <a href="https://jina.ai/news/late-chunking-in-long-context-embedding-models/" class="text-white">blog</a>
            </figcaption>
        </figure>
    </div>
</div>

It is important to note that effective late chunking relies on embedding models with long-context capabilities. In their example, they use [jina-embeddings-v2-base-en](https://jina.ai/news/jina-ai-launches-worlds-first-open-source-8k-text-embedding-rivaling-openai/), which can handle up to 8,192 tokens—roughly the equivalent of ten standard pages of text. Recently, other embeddings have become available, such as [voyage-3](https://blog.voyageai.com/2024/09/18/voyage-3/), which supports up to 32,000 tokens.

To evaluate the effectiveness of late chunking, they tested several retrieval tasks from the [BeIR  benchmark](https://github.com/beir-cellar/beir). In all cases, late chunking outperformed the naive approach, particularly for longer documents, where the performance gap between the two methods increased with document length. This demonstrates that late chunking becomes more effective as document length grows.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <figure style="width: 60%; margin: 0 auto;">
            {% include figure.liquid loading="eager" path="assets/img/jina_length.png" class="img-fluid rounded z-depth-1" %}
            <figcaption class="text-black text-center mt-2" style="color: black; width: 100%;">
                Late chunking's improvement over naive chunking in retrieval tasks is correlated with the avg. document length, from Jina AI 
                <a href="https://jina.ai/news/late-chunking-in-long-context-embedding-models/">blog</a>
            </figcaption>
        </figure>
    </div>
</div>


## Encoding Long Contexts in Document Classification

Unlike RAG, which relies on retrieving relevant pieces of information, classification tasks typically require fine-tuning a LLM for the downstream task. In such tasks, a [CLS] token is added at the beginning of the input sequence, and its final embedding, representing the entire sequence, is used for classification by adding a classifier layer on top. Fine-tuning is then used to adjust the weights of a pre-trained transformer model (like BERT) on task-specific labeled data, enabling it to make accurate predictions for that particular task.

Here, I would like to illustrate using the EURLEX-57K dataset <d-cite key='chalkidis2019large'></d-cite>, a multi-label classification dataset based on EU legal documents. EURLEX-57K includes 57,000 legislative documents from EUR-LEX, with an average length of 727 words. 


| Input(s) | Output(s) / Label(s) |
|-----------------------|----------|
| **Text**: <small> Commission Regulation (EC) No 1156/2001 of 13 June 2001 fixing the export refunds on white sugar and raw sugar exported in its unaltered state. <br> THE COMMISSION OF THE EUROPEAN COMMUNITIES Having regard to the Treaty establishing the European Community, Having regard to Council Regulation (EC) No 2038/1999 of 13 September 1999 on the common organisation of the markets in the sugar sector(1), as amended by Commission Regulation (EC) No 1527/2000(2), and in particular point (a) of the second subparagraph of Article 18(5) thereof, <br> Whereas: (1) Article 18 of Regulation (EC) No 2038/1999 provides that the difference between quotations or prices on the world market for the products listed in Article 1(1)(a) of that Regulation and prices for those products within the Community may be covered by an export refund. (2) Regulation (EC) No 2038/1999 provides that when refunds on white and raw sugar, undenatured and exported in its unaltered state, are being fixed account must be taken of the situation on the Community and world markets in sugar and in particular of the price and cost factors [...] </small> | 28 (Trade Policy) <br> 93 (Beverages and Sugar) <br> 94 (Foodstuff) |



Several approaches are available to bypass BERT’s maximum text length limit

- **Document Truncation**: The simplest approach involves fine-tuning BERT by truncating long documents to the first 512 tokens (this can be done by setting `truncation=True` in the tokenizer function).

<small><span style="background-color: lightgreen"> Commission Regulation (EC) No 1156/2001 of 13 June 2001 fixing the export refunds on white sugar and raw sugar exported in its unaltered state. <br> THE COMMISSION OF THE EUROPEAN COMMUNITIES Having regard to the Treaty establishing the European Community, Having regard to Council Regulation (EC) No 2038/1999 of 13 September 1999 on the common organisation of the markets in the sugar sector(1), as amended by Commission Regulation (EC) No 1527/2000(2), and in  </span> particular point (a) of the second subparagraph of Article 18(5) thereof, <br> Whereas: (1) Article 18 of Regulation (EC) No 2038/1999 provides that the difference between quotations or prices on the world market for the products listed in Article 1(1)(a) of that Regulation and prices for those products within the Community may be covered by an export refund. (2) Regulation (EC) No 2038/1999 provides that when refunds on white and raw sugar, undenatured and exported in its unaltered state, are being fixed account must be taken of the situation on the Community and world markets in sugar and in particular of the price and cost factors [...]</small>


- **Longformer** <d-cite key='beltagy2020longformer'></d-cite> : It is designed to process longer input sequences using an efficient self-attention mechanism that scales linearly with the input length. Unlike BERT, which can handle up to 512 tokens, Longformer can process up to 4,096 tokens.
- **Hierarchical Encoding** <d-cite key='pappagari2019hierarchical'></d-cite>: It divides long documents into smaller chunks of 200 tokens and uses a Transformer layer over BERT-based chunk representations (I implemented it under the Github folder).
- **Cognize LongTeXts** <d-cite key='ding2020cogltx'></d-cite>: In this model,  two BERT (or RoBERTa) models are jointly trained to select key sentences from long documents for various tasks including text classification. The underlying idea that a few key sentences are sufficient for a given task has been explored for question answering.

However, in this recent paper <d-cite key='park2022efficient'></d-cite>, they evaluate different models and show that more complex models often fail to outperform simple baselines and yield inconsistent performance across datasets.

As mentioned above, the context limitation arises from the transformer architecture. Some research has focused on developing new architectures, such as the State Space Model (SSM). However, there is still limited understanding of how these models can improve tasks like RAG or classification.

#### Acknowledgment