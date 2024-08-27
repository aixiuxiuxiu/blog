---
layout: post
title: Find the Rigt PDF Extractor to Enhance Your RAG Workflow
date: 2024-08-27 00:32:13
description: 
tags: RAG data
categories: sample-posts
tabs: true
---

Over the past few years, pretrained language models have driven significant advancements across many tasks. However, in practical industrial applications, the primary performance bottleneck for efficient document retrieval often lies not in the embedding model’s capabilities but in the preceding data ingestion pipeline. Indexing a standard PDF document involves several steps, but always starting with the use of PDF parsers or Optical Character Recognition (OCR) systems to extract text from the document's pages.



Recently, the new model [ColPali](https://arxiv.org/html/2407.01449v2) has attracte a lot of attention, for its use of a vision-language model to extract information for retrieval purposes. This approach demonstrates that leveraging recent Vision Language Models can produce high-quality, contextualized embeddings directly from images of document pages.

In the market, a variety of PDF extraction tools are available. Free options like PDFPlumber and PyPDF are popular, while some companies offer paid solutions for more advanced PDF extraction capabilities. Here, I will compare different free PDF extractor tools, highlighting their advantages and disadvantages.



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
