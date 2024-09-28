---
layout: post
title: Optimizing PDF Extraction for Building a Robust RAG
date: 2024-09-11 00:32:13
description: 
tags: RAG Evaluation
categories: sample-posts
tabs: true
---

Recently, Retrieval-Augmented Generation (RAG) has emerged as a prominent approach that leverages large language models for building advanced applications. However, in practical industrial settings, the primary bottleneck affecting RAG performance—especially in document retrieval—often lies not in the capabilities of the embedding model, but in the data ingestion pipeline. The process of building a RAG system begins with indexing documents, many of which are in PDF format. This typically involves using PDF parsers to extract text from the document’s pages, a crucial step that significantly impacts the overall retrieval accuracy.

Extracting text from PDFs are challenging in many aspects (can also read [this](https://pypdf.readthedocs.io/en/stable/user/extract-text.html)):
* **Complex and variable structures**: PDFs are designed for visual presentation, not structured text extraction, leading to fragmented or misaligned text.
* **Layout complexity**: PDFs often contain multi-column formats, tables, and embedded images, making it difficult to maintain a logical reading order.
* **Inconsistent text encoding**: Different fonts, character encodings, or special symbols can lead to extraction errors, such as missing or garbled text.
* **Discontinuous text chunks**: Text might be broken into fragments that need to be reassembled into meaningful sequences.


<!-- Recently, the new model [ColPali](https://arxiv.org/html/2407.01449v2) has attracte a lot of attention, for its use of a vision-language model to extract information for retrieval purposes. This approach demonstrates that leveraging recent Vision Language Models can produce high-quality, contextualized embeddings directly from images of document pages. -->


There are various  PDF extraction tools available on the market. Some companies provide paid solutions with advanced features, while several open-source Python packages, such as PyPDF and PDFPlumber, are also available. In this blog, I will present an experiment comparing the impact of different PDF extraction methods on retrieval performance. The results show that high-quality PDF extraction can significantly improve retrieval. (The code is available on [Github](https://github.com/aixiuxiuxiu/pdf-extraction-blog/tree/main))


## PDF extraction


The following example is a PDF excerpt from the [Swiss Civil Code](https://www.fedlex.admin.ch/eli/cc/24/233_245_233/en). The PDF page presents a high level of complexity, with text that is discontinuously arranged and interspersed with numerous footnotes.


<div class="row justify-content-center mt-3">
    <div class="col-sm mt-3 mt-md-0" style="width: 90%;">
        {% include figure.liquid loading="eager" path="assets/img/pdf_example.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    
</div>


Here I use [`pdfplumber`](https://github.com/jsvine/pdfplumber), a library built on top of [`pdfminer.six`](https://github.com/goulu/pdfminer), that offers a wide range of customizable features for extracting text from PDFs. This package allows the extraction of pages and text while preserving the original layout. Additionally, it can parse various character properties, such as page number, text, and coordinates. For instance, the `.crop()` method can be used to crop a page into a specific bounding box: `.crop((x0, top, x1, bottom), relative=False, strict=True)`.


I built two data loaders. The first one, called `PDFLoader`, extracts sequences directly using `pdfplumber` without additional postprocessing. In this approach, the raw extraction from the PDF example above appears unstructured and unnatural, as it reads lines horizontally, even when they are split into two blocks. 

{::nomarkdown}
{% assign jupyter_path = 'assets/jupyter/pdfplumber_old.ipynb' | relative_url %}
{% capture notebook_exists %}{% file_exists assets/jupyter/pdfplumber_old.ipynb %}{% endcapture %}
{% if notebook_exists == 'true' %}
  {% jupyter_notebook jupyter_path %}
{% else %}
  <p>Sorry, the notebook you are looking for does not exist.</p>
{% endif %}
{:/nomarkdown}
<div class="caption">
Illustraction of PDFLoader result
</div>


The other is called `Custom PDFLoader`, which uses `pdfplumber`  to extract text from different boxes and combine them together. Here, I extract text from two distinct boxes: one for the left side and one for the right side of the page. I use the `x0` of the word **Art** as the `x0` for the right box and the same value as the `x1` for the left box. This makes the extracted sequences more natural.


{::nomarkdown}
{% assign jupyter_path = 'assets/jupyter/pdfplumber.ipynb' | relative_url %}
{% capture notebook_exists %}{% file_exists assets/jupyter/pdfplumber_old.ipynb %}{% endcapture %}
{% if notebook_exists == 'true' %}
  {% jupyter_notebook jupyter_path %}
{% else %}
  <p>Sorry, the notebook you are looking for does not exist.</p>
{% endif %}
{:/nomarkdown}
<div class="caption">
Illustraction of Custom PDFLoader result
</div>


## Evaluation

This section evaluates retrieval quality by comparing the two methods of PDF extraction: `Custom PDFLoader` and `PDFLoader`, using the `RetrieverEvaluator` module provided in `LLAMAIndex`.

<div class="row justify-content-center mt-3">
    <div class="col-sm-9 mt-3 mt-md-0" style="width: 65%;">
        {% include figure.liquid loading="eager" path="assets/img/pdf-flow.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>


* First, I split the extracted text from the PDF file [Swiss Civil Code](https://www.fedlex.admin.ch/eli/cc/24/233_245_233/en) (about 350 pages), into small chunks of 1024 characters, with a chunk overlap of 200.
* Second, I created the evaluation dataset using the `generate_question_context_pairs` function. This function can automatically generate a set of (question, context) pairs using LLMs. I generated two questions from each context chunk with GPT-4-mini, resulting in a total number of 450 queries.
* I built a retriever using the built-in `VectorStoreIndex` function in `LLAMAIndex`, and performed retrieval using the top-k similarity method.
* Finally, I ran the `RetrieverEvaluator` on the evaluation dataset we generated, using the provided evaluation metrics.
  * hit-rate: the correct answer is present in the top k retrieved results
  * MRR: the reciprocal of the rank at which the first relevant result appears.
  * Precision:  the fraction of relevant documents retrieved out of the total number of documents retrieved.
  * Recall:  the fraction of relevant documents that were retrieved out of the total number of relevant documents available.
  * AP: the average of precision values at the ranks where relevant documents are retrieved.
  * NDCG: the quality of a ranking based on the positions of relevant documents

The results demonstrate a clear improvement in retrieval performance of approximately 5% when using the `custom PDFLoader`.

<div class="row justify-content-center mt-3">
    <div class="col-sm mt-3 mt-md-0" style="width: 90%;">
        {% include figure.liquid loading="eager" path="assets/img/result.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    



This experiment highlights that optimizing PDF extraction is crucial, and can be just as important as improving embedding models. Please note that this article focuses on PDFs containing only text. If your PDF contains more complex elements, such as images or tables, alternative methods may need to be considered. Recently, there has been a spike in interest in the new model [ColPali](https://arxiv.org/html/2407.01449v2) for its use of a vision-language model to extract information for retrieval purposes. This approach demonstrates that leveraging modern Vision-Language Models can generate high-quality, contextualized embeddings directly from images of document pages.




