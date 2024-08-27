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

When a PDF (not a scanned one) contains only text (without figures, tables, etc.), the most straightforward way to extract the information is by using a Python package. There are various PDF extraction tools available on the market. Some useful open-source Python packages, like PDFPlumber and PyPDF, are very handy, while some companies offer paid solutions with more advanced capabilities. In this discussion, I will compare different free PDF extraction tools, highlighting their advantages and disadvantages.



## First tabs

To add tabs, use the following syntax:

{% raw %}

```liquid
{% tabs group-name %}

{% tab group-name tab-name-1 %}

Content 1

{% endtab %}

{% tab group-name tab-name-2 %}

Content 2

{% endtab %}

{% endtabs %}
```

{% endraw %}

With this you can generate visualizations like:

{% tabs log %}

{% tab log php %}

```php
var_dump('hello');
```

{% endtab %}

{% tab log js %}

```javascript
console.log("hello");
```

{% endtab %}

{% tab log ruby %}

```javascript
pputs 'hello'
```

{% endtab %}

{% endtabs %}

## Another example

{% tabs data-struct %}

{% tab data-struct yaml %}

```yaml
hello:
  - "whatsup"
  - "hi"
```

{% endtab %}

{% tab data-struct json %}

```json
{
  "hello": ["whatsup", "hi"]
}
```

{% endtab %}

{% endtabs %}

## Tabs for something else

{% tabs something-else %}

{% tab something-else text %}

Regular text

{% endtab %}

{% tab something-else quote %}

> A quote

{% endtab %}

{% tab something-else list %}

Hipster list

- brunch
- fixie
- raybans
- messenger bag

{% endtab %}

{% endtabs %}
