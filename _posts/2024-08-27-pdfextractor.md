---
layout: post
title: Find the Right PDF Extractor for Building RAG
date: 2024-08-27 00:32:13
description: 
tags: RAG data
categories: sample-posts
tabs: true
---

Recently, Retrieval-Augmented Generation (RAG) has emerged as a prominent approach that leverages large language models for building applications. However, in practical industrial settings, the primary bottleneck for the performance of RAG, particularly in terms of document retrieval, often lies not in the embedding model’s capabilities, but in the prior data ingestion pipeline. Building a RAG system begins with indexing documents, which are often in PDF format. This process typically starts with the use of PDF parsers or Optical Character Recognition (OCR) systems to extract text from the document's pages.


<!-- Recently, the new model [ColPali](https://arxiv.org/html/2407.01449v2) has attracte a lot of attention, for its use of a vision-language model to extract information for retrieval purposes. This approach demonstrates that leveraging recent Vision Language Models can produce high-quality, contextualized embeddings directly from images of document pages. -->

When a (machine-generated) PDF contains mostly text, there are various PDF extraction tools available on the market. Some companies offer paid solutions with advanced capabilities, while several open-source Python packages, such as `PDFPlumber` and `PyPDF`, are also very useful. In this discussion, I will compare two different free PDF extraction Python packages, highlighting their advantages and disadvantages.


## pdfplumber vs pypdf 

[`pdfplumber`](https://github.com/jsvine/pdfplumber) is built on [`pdfminer.six`](https://github.com/goulu/pdfminer), enabling many customizable functions. This package can extract pages and text while preserving the layout. Additionally, it can identify the coordinates of words, allowing for the extraction of text within specific areas.

On the other hand, [`pypdf`](https://pypi.org/project/pypdf/) also allows for text extraction while maintaining the layout. You can use `visitor` functions to control which parts of a page you want to process and extract. However, it does not support extracting the coordinates of words.

The following example compares the extraction results of pdfplumber and pypdf using a PDF excerpt from the [Swiss Civil Code](https://www.fedlex.admin.ch/eli/cc/24/233_245_233/en). The PDF page presents a high level of complexity, with text that is discontinuously arranged and interspersed with numerous footnotes.


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/pdf_example.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    
</div>



`pdfplumber`  can parse various properties of characters, such as page number, text, coordinates, etc. You can use the `.crop()` method to crop a page into a bounding box, `.crop((x0, top, x1, bottom), relative=False, strict=True)`. Here is how I extract text from two boxes: one for the left side and the other for the right side. The x and y coordinate values can be determined using the x0, y0, x1, and y1 values of certain characters. For example, I use the x0 of the word ‘Art’ as the x0 for the right box (and x1 for the left box).

{::nomarkdown}
{% assign jupyter_path = 'assets/jupyter/pdfplumber.ipynb' | relative_url %}
{% capture notebook_exists %}{% file_exists assets/jupyter/blog.ipynb %}{% endcapture %}
{% if notebook_exists == 'true' %}
  {% jupyter_notebook jupyter_path %}
{% else %}
  <p>Sorry, the notebook you are looking for does not exist.</p>
{% endif %}
{:/nomarkdown}





## Evaluation

Here, I will use the `RetrieverEvaluator` module provided within `LLAMAIndex` to evaluate the retrieval quality by comparing two different methods of PDF extraction.

* First, I used the `generate_question_context_pairs` function to auto-generate a set of (question, context) pairs over the entire PDF file [Swiss Civil Code](https://www.fedlex.admin.ch/eli/cc/24/233_245_233/en), which contains about 350 pages. I generated 2 questions from each context chunk, resulting in a total number of questions.
* Then, I ran the RetrieverEvaluator on the evaluation dataset we generated, using the evaluation metrics provided.

  * hit-rate: the correct answer is present in the top k retrieved results
  * MRR: the reciprocal of the rank at which the first relevant result appears.
  * Precision:  the fraction of relevant documents retrieved out of the total number of documents retrieved.
  * Recall:  the fraction of relevant documents that were retrieved out of the total number of relevant documents available.
  * AP: the average of precision values at the ranks where relevant documents are retrieved.
  * NDCG: the quality of a ranking based on the positions of relevant documents



<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/result.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    
</div>




{% tabs something-else %}

{% tab something-else text %}



{% endtab %}

{% tab something-else quote %}

> A quote

{% endtab %}

{% tab something-else list %}


{% endtabs %}
