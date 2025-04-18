---
title: Add semantic ranking
titleSuffix: Azure AI Search
description: Set a semantic query type to attach the deep learning models of semantic ranker.

manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.custom:
  - ignite-2023
ms.topic: how-to
ms.date: 09/24/2024
---

# Add semantic ranking to queries in Azure AI Search

You can apply semantic ranking to text queries, hybrid queries, and vector queries if your search documents contain string fields and the [vector query has a text representation](vector-search-how-to-query.md#query-with-integrated-vectorization) in the search document.

This article explains how to invoke the semantic ranker on queries. 

## Prerequisites

+ A search service, Basic tier or higher, with [semantic ranker enabled](semantic-how-to-enable-disable.md).

+ An existing search index with a [semantic configuration](semantic-how-to-configure.md) and rich text content.

+ Review [semantic ranking](semantic-search-overview.md) if you need an introduction to the feature.

> [!NOTE]
> Captions and answers are extracted verbatim from text in the search document. The semantic subsystem uses machine reading comprehension to recognize content having the characteristics of a caption or answer, but doesn't compose new sentences or phrases. For this reason, content that includes explanations or definitions work best for semantic ranking. If you want chat-style interaction with generated responses, see [Retrieval Augmented Generation (RAG)](retrieval-augmented-generation-overview.md).

## Choose a client

You can use any of the following tools and SDKs to build a query that uses semantic ranking:

+ [Azure portal](https://portal.azure.com), using the index designer to add a semantic configuration.
+ [Visual Studio Code](https://code.visualstudio.com/download) with a [REST client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client)
+ [Azure SDK for .NET](https://www.nuget.org/packages/Azure.Search.Documents)
+ [Azure SDK for Python](https://pypi.org/project/azure-search-documents)
+ [Azure SDK for Java](https://central.sonatype.com/artifact/com.azure/azure-search-documents)
+ [Azure SDK for JavaScript](https://www.npmjs.com/package/@azure/search-documents)

## Avoid features that bypass relevance scoring

A few query capabilities bypass relevance scoring, which makes them incompatible with semantic ranking. If your query logic includes the following features, you can't semantically rank your results:

+ A query with `search=*` or an empty search string, such as pure filter-only query, won't work because there's nothing to measure semantic relevance against and so the search scores are zero. The query must provide terms or phrases that can be evaluated during processing.

+ Sorting (orderBy clauses) on specific fields overrides search scores and a semantic score. Given that the semantic score is supposed to provide the ranking, adding an orderby clause results in an HTTP 400 error if you apply semantic ranking over ordered results.

## Set up the query

In this step, add parameters to the query request. To be successful, your query should be full text search (using the `search` parameter to pass in a string) or a vector query, and the index should contain text fields with rich semantic content and a semantic configuration.

### [**Azure portal**](#tab/portal-query)

[Search explorer](search-explorer.md) includes options for semantic ranking. 

1. Sign in to the [Azure portal](https://portal.azure.com).

1. Open a search index and select **Search explorer**.

1. Select **Query options**. If you already defined a semantic configuration, it's selected by default. If you don't have one, [create a semantic configuration](semantic-how-to-configure.md) for your index.

    :::image type="content" source="./media/semantic-search-overview/search-explorer-semantic-query-options-v2.png" alt-text="Screenshot showing query options in Search explorer." border="true":::

1. Enter a query, such as "historic hotel with good food", and select **Search**.

1. Alternatively, select **JSON view** and paste definitions into the query editor:

   :::image type="content" source="./media/semantic-search-overview/semantic-portal-json-query.png" alt-text="Screenshot showing JSON query syntax in the Azure portal." border="true":::

   Here's some JSON text that you can paste into the view:

   ```json
    {
        "queryType": "semantic",
        "search": "historic hotel with good food",
        "semanticConfiguration": "my-semantic-config",
        "answers": "extractive|count-3",
        "captions": "extractive|highlight-true",
        "highlightPreTag": "<strong>",
        "highlightPostTag": "</strong>",
        "select": "HotelId,HotelName,Description,Category",
        "count": true
    }
   ```

### [**REST API**](#tab/rest-query)

Use [Search Documents](/rest/api/searchservice/documents/search-post) to formulate the request.

A response includes an `@search.rerankerScore` automatically. If you want captions or answers in the response, add captions and answers to the request.

The following example in this section uses the [hotels-sample-index](search-get-started-portal.md) to demonstrate semantic ranking with semantic answers and captions.

1. Paste the following request into a web client as a template. Replace the service name and index name with valid values.

    ```http
    POST https://[service name].search.windows.net/indexes/hotels-sample-index/docs/search?api-version=2024-07-01      
    {
        "queryType": "semantic",
        "search": "newer hotel near the water with a great restaurant",
        "semanticConfiguration": "my-semantic-config",
        "answers": "extractive|count-3",
        "captions": "extractive|highlight-true",
        "highlightPreTag": "<strong>",
        "highlightPostTag": "</strong>",
        "select": "HotelId,HotelName,Description,Category",
        "count": true
    }
    ```

1. Set "queryType" to "semantic".

1. Set "search" to a full text search query. Your search string can support either the [simple syntax](query-simple-syntax.md) or [full Lucene syntax](query-lucene-syntax.md). Semantic ranking is an extension of full text search, so while "search" isn't required, you won't get an expected outcome if it's an empty search (`"search": "*"`).

1. Set "semanticConfiguration" to a [predefined semantic configuration](semantic-how-to-configure.md) that's embedded in your index.

1. Set "answers" to specify whether [semantic answers](semantic-answers.md) are included in the result. Currently, the only valid value for this parameter is `extractive`. Answers can be configured to return a maximum of 10. The default is one. This example shows a count of three answers: `extractive|count-3`.

   Answers aren't guaranteed on every request. To get an answer, the query must look like a question and the content must include text that looks like an answer.

1. Set "captions" to specify whether semantic captions are included in the result. Currently, the only valid value for this parameter is `extractive`. Captions can be configured to return results with or without highlights. The default is for highlights to be returned. This example returns captions without highlights: `extractive|highlight-false`.

   The basis for captions and answers are the fields referenced in the "semanticConfiguration". These fields are under a combined limit in the range of 2,000 tokens or approximately 20,000 characters. If you anticipate a token count exceeding this limit, consider a [data chunking step](vector-search-how-to-chunk-documents.md) using the [Text split skill](cognitive-search-skill-textsplit.md). This approach introduces a dependency on an [AI enrichment pipeline](cognitive-search-concept-intro.md) and [indexers](search-indexer-overview.md).

1. Set "highlightPreTag" and "highlightPostTag" if you want to override the default highlight formatting that's applied to captions.

   Captions apply highlight formatting over key passages in the document that summarize the response. The default is `<em>`. If you want to specify the type of formatting (for example, yellow background), you can set the highlightPreTag and highlightPostTag.

1. Set ["select"](search-query-odata-select.md) to specify which fields are returned in the response, and "count" to return the number of matches in the index. These parameters improve the quality of the request and readability of the response.

1. Send the request to execute the query and return results.

### [**.NET SDK**](#tab/dotnet-query)

Use QueryType or SemanticQuery to invoke semantic ranker on a semantic query. The [following example](https://github.com/Azure/azure-sdk-for-net/blob/main/sdk/search/Azure.Search.Documents/samples/Sample08_SemanticSearch.md) is from the Azure SDK team.

```csharp
SearchResults<Hotel> response = await searchClient.SearchAsync<Hotel>(
    "Is there any hotel located on the main commercial artery of the city in the heart of New York?",
    new SearchOptions
    {
        SemanticSearch = new()
        {
            SemanticConfigurationName = "my-semantic-config",
            QueryCaption = new(QueryCaptionType.Extractive),
            QueryAnswer = new(QueryAnswerType.Extractive)
        },
        QueryLanguage = QueryLanguage.EnUs,
        QueryType = SearchQueryType.Semantic
    });

int count = 0;
Console.WriteLine($"Semantic Search Results:");

Console.WriteLine($"\nQuery Answer:");
foreach (QueryAnswerResult result in response.SemanticSearch.Answers)
{
    Console.WriteLine($"Answer Highlights: {result.Highlights}");
    Console.WriteLine($"Answer Text: {result.Text}");
}

await foreach (SearchResult<Hotel> result in response.GetResultsAsync())
{
    count++;
    Hotel doc = result.Document;
    Console.WriteLine($"{doc.HotelId}: {doc.HotelName}");

    if (result.SemanticSearch.Captions != null)
    {
        var caption = result.SemanticSearch.Captions.FirstOrDefault();
        if (caption.Highlights != null && caption.Highlights != "")
        {
            Console.WriteLine($"Caption Highlights: {caption.Highlights}");
        }
        else
        {
            Console.WriteLine($"Caption Text: {caption.Text}");
        }
    }
}
Console.WriteLine($"Total number of search results:{count}");
```

---

## Evaluate the response

Only the top 50 matches from the initial results can be semantically ranked. As with all queries, a response is composed of all fields marked as retrievable, or just those fields listed in the `select` parameter. A response includes the original relevance score, and might also include a count, or batched results, depending on how you formulated the request.

In semantic ranking, the response has more elements: a new [semantically ranked relevance score](semantic-search-overview.md#how-ranking-is-scored), an optional caption in plain text and with highlights, and an optional [answer](semantic-answers.md). If your results don't include these extra elements, then your query might be misconfigured. As a first step towards troubleshooting the problem, check the semantic configuration to ensure it's specified in both the index definition and query.

In a client app, you can structure the search page to include a caption as the description of the match, rather than the entire contents of a specific field. This approach is useful when individual fields are too dense for the search results page.

The response for the above example query returns the following match as the top pick. Captions are returned because the "captions" property is set, with plain text and highlighted versions. Answers are omitted from the example because one couldn't be determined for this particular query and corpus.

```json
"@odata.count": 35,
"@search.answers": [],
"value": [
    {
        "@search.score": 1.8810667,
        "@search.rerankerScore": 1.1446577133610845,
        "@search.captions": [
            {
                "text": "Oceanside Resort. Luxury. New Luxury Hotel. Be the first to stay. Bay views from every room, location near the pier, rooftop pool, waterfront dining & more.",
                "highlights": "<strong>Oceanside Resort.</strong> Luxury. New Luxury Hotel. Be the first to stay.<strong> Bay</strong> views from every room, location near the pier, rooftop pool, waterfront dining & more."
            }
        ],
        "HotelName": "Oceanside Resort",
        "Description": "New Luxury Hotel. Be the first to stay. Bay views from every room, location near the pier, rooftop pool, waterfront dining & more.",
        "Category": "Luxury"
    },
  ...
]
```

## Expected workloads

For semantic ranking, you should expect a search service to support up to 10 concurrent queries per replica. 

The service throttles semantic ranking requests if volumes are too high. An error message that includes these phrases indicate the service is at capacity for semantic ranking:

```json
Error in search query: Operation returned an invalid status 'Partial Content'`
@search.semanticPartialResponseReason`
CapacityOverloaded
```

If you anticipate consistent throughput requirements near, at, or higher than this level, please file a support ticket so that we can provision for your workload.

## Next steps

Semantic ranking can be used in hybrid queries that combine keyword search and vector search into a single request and a unified response.

> [!div class="nextstepaction"]
> [Hybrid query with semantic ranker](hybrid-search-how-to-query.md#semantic-hybrid-search)