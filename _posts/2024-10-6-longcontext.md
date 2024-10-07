---
layout: distill
title: How LLMs handle long context? 
#description: an example of a distill-style blog post and main elements
tags: LLM evaluation
#giscus_comments: true
date: 2024-10-07
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
  - name: Why long context is so hard?
    # if a section has subsections, you can add them as follows:
    # subsections:
    #   - name: Example Child Subsection 1
    #   - name: Example Child Subsection 2
  - name: Long Context in training
  - name: Encode Long Context


# Below is an example of injecting additional post-specific styles.
# If you use this post as a template, delete this _styles block.
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

## Why long context is so hard?

Handling long contexts is one of the main challenges for large language models (LLMs). Current LLMs are limited by their context length; for example, BERT-based models typically have a context window of 512 tokens, while standard GPT-3 models can handle around 2048 tokens. GPT-4 offers two versions: one with a context window of 8,192 tokens and another with an extended window of 32,768 tokens (32K tokens).

Most LLMs, such as GPT and BERT, are built on the Transformer architecture, which relies on a self-attention mechanism. This mechanism compares each token in the input sequence to every other token, resulting in quadratic complexity in terms of both memory usage and computational cost.

In addition to computational constraints, processing long contexts also increases complexity. Long sentences often contain intricate language structures that are challenging even for humans to interpret, a phenomenon particularly evident in domains such as legal and financial texts.

## Long context in training

While most closed-source models provide limited information on this topic, the LLama report <d-cite key="dubey2024llama"></d-cite> elaborates on it, explaining that handling long contexts typically occurs during the later stages of pretraining:

> In the final stages of pre-training, we train on long sequences to support context windows of up to 128K tokens.
We do not train on long sequences earlier because the compute in self-attention layers grows quadratically in
the sequence length. We increase the supported context length in increments, pre-training until the model has
successfully adapted to the increased context length. We assess successful adaptation by measuring whether (1)
model performance on short-context evaluations has recovered completely and (2) the model perfectly solves
“needle in a haystack” tasks up to that length. In Llama 3 405B pre-training, we increased context length
gradually in six stages, starting from the original 8K context window and ending in the final 128K context
window. This long-context pre-training stage was performed using approximately 800B training tokens.

## Long context in encoding

Recently, many progress has made to encode the long context efficiently: 
- Navive chuncking: The naive encoding approach (as seen on the left side of the image below) involves using sentences, paragraphs, or maximum length limits to split the text a priori. Afterward, an embedding model is repetitively applied to these resulting chunks. To generate a single embedding for each chunk, many embedding models use mean pooling on these token-level embeddings to output a single embedding vector. See an example from [OpenAI CookBook](https://cookbook.openai.com/examples/embedding_long_inputs)
- Late chuncking Jina AI <d-cite key="gunther2024late"></d-cite>: first applies the transformer layer of the embedding model to the entire text or as much of it as possible. This generates a sequence of vector representations for each token that encompasses textual information from the entire text. Subsequently, mean pooling is applied to each chunk of this sequence of token vectors, yielding embeddings for each chunk that consider the entire text's context. Unlike the naive encoding approach, which generates independent and identically distributed (i.i.d.) chunk embeddings, late chunking creates a set of chunk embeddings where each one is "conditioned on" the previous ones, thereby encoding more contextual information for each chunk.

<div class="row mt-3" style="background-color: black;">
    <div class="col-sm mt-3 mt-md-0">
        <figure style="width: 90%; margin: 0 auto;">
            {% include figure.liquid loading="eager" path="assets/img/latechuncking.jpg" class="img-fluid rounded z-depth-1" %}
            <figcaption class="text-white text-center mt-2">
                An illustration of the naive chunking strategy (left) and the late chunking strategy (right), from Jina AI 
                <a href="https://jina.ai/news/late-chunking-in-long-context-embedding-models/" class="text-white">blog</a>
            </figcaption>
        </figure>
    </div>
</div>

- ColBERT’s Late Interaction <d-cite key="santhanam2021colbertv2"></d-cite>

## Long context in evaluation

 <d-cite key="dubois2024length"></d-cite>
