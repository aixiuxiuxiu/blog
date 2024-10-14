---
layout: distill
title: How to Encode Long Text Using Large Language Models? 
#description: an example of a distill-style blog post and main elements
tags: LLM Evaluation
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
  - name: Naive Chuncking
  - name: Late Chuncking 

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

Most of current large language models (LLMs) have limited context length. For instance, BERT-based models typically have a context length of 512 tokens—if a sequence exceeds 512 tokens, only part of it is encoded. In contrast, standard GPT-3 models handle around 2,048 tokens, while GPT-4 offers two variants: one with 8,192 tokens and another with an extended window of 32,768 tokens (32K tokens). However, many tasks involving LLMs require handling documents that far exceed these limits. For example, building a retrieval-augmented generation (RAG) system requires integrating internal knowledge bases, which often involves encoding multi-page documents. Likewise, chat applications may need to include background context (e.g., previous conversations) spanning several pages. Additionally, tasks like text classification may involve encoding texts containing thousands of tokens.

In addressing the limits arising from context length, two main research directions have emerged. The first is to develop models with longer context lengths, as illustrated by the table showing the evolution of context length across different models. However, this is challenging because most LLMs, such as GPT and BERT, are based on the transformer architecture, which uses a self-attention mechanism. This mechanism compares each token in the input sequence with every other token, leading to quadratic complexity in both memory usage and computational cost.

| Model      | Context length | Number of English pages* |
|------------|----------------|--------------------------|
| GPT 3.5    | 4,096          | 6                        |
| GPT 4      | 8,192          | 12                       |
| GPT 4-32k  | 32,768         | 49                       |
| Llama 1    | 2,048          | 3                        |
| Llama 2    | 4,096          | 6                        |

*Context Length Comparison (*Assuming 500 words per page.) 

The second approach involves improving encoding techniques. Encoding all the information from a multi-page document into a single embedding vector is difficult, if not impossible. Although we have a model with a long context window, trying to encode everything from multiple pages into one vector may result in the loss of important information. Alternatively, chunking long texts into smaller segments while maintaining dependencies between them offers a more viable approach.


This blog will focus on the second direction, explaining the available techniques to encode long context.

## Naive Chuncking


The naive encoding approach (as seen on the left side of the image below) involves splitting the text a priori using sentences, paragraphs, or maximum length limits. Afterward, an embedding model is applied to each resulting chunk. To generate a single embedding for each chunk, many models use mean pooling on token-level embeddings, producing a single embedding vector. You can see an example in the [OpenAI CookBook](https://cookbook.openai.com/examples/embedding_long_inputs).

In some cases, it may be useful to split chunks at paragraph or sentence boundaries to better preserve the meaning of the text. This method has been implemented in Langchain using the function `RecursiveCharacterTextSplitter` (see this [blog](https://dev.to/eteimz/understanding-langchains-recursivecharactertextsplitter-2846) for more detailed explanations of this function.)

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

As a result, each chunk falls within the length limits of the LLM. Then, the chunks are encoded into embeddings, and combined into one using mean pooling (by averaging the embeddings of all the chunks). 

Naive chunking allows encoding the entire sequence without cutting until the maximum context window is reached. However, because it encodes each chunk independently, it breaks the dependencies between chunks. This means that each chunk is treated as an independent element, without considering the context before or after it.

## Late Chuncking 

Late chunking, introduced by Jina AI <d-cite key='gunther2024late'></d-cite>, addresses the contextual issue. It first applies the transformer layer of the embedding model to the entire text, or as much of it as possible. This generates a sequence of vector representations for each token, encompassing textual information from the entire text. Subsequently, mean pooling is applied to each chunk of this sequence of token vectors, yielding embeddings that consider the context of the entire text. Unlike the naive encoding approach, which generates independent and identically distributed (i.i.d.) chunk embeddings, late chunking creates chunk embeddings that are "conditioned on" the previous ones, thereby encoding more contextual information for each chunk.

<div class="row mt-3" style="background-color: black;">
    <div class="col-sm mt-3 mt-md-0">
        <figure style="width: 90%; margin: 0 auto;">
            {% include figure.liquid loading="eager" path="assets/img/latechuncking.png" class="img-fluid rounded z-depth-1" %}
            <figcaption class="text-white text-center mt-2">
                An illustration of the naive chunking strategy (left) and the late chunking strategy (right), from Jina AI 
                <a href="https://jina.ai/news/late-chunking-in-long-context-embedding-models/" class="text-white">blog</a>
            </figcaption>
        </figure>
    </div>
</div>

## ColBERT’s Late Interaction <d-cite key="santhanam2021colbertv2"></d-cite>

## Long context in evaluation

 <d-cite key="dubois2024length"></d-cite>
