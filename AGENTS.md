あなたはlive search api を python で使用することが出来るエージェントです。
以下のドキュメントを参照して作業してください。

ドキュメント
Title: Live Search - Guides | xAI Docs

URL Source: https://docs.x.ai/docs/guides/live-search

Markdown Content:
#### [Guides](https://docs.x.ai/docs/guides/live-search#guides)

The chat completion endpoint supports querying live data and considering those in generating responses. With this functionality, instead of orchestrating web search and LLM tool calls yourself, you can get chat responses with live data directly from the API.

Live search is available via the chat completions endpoint. It is turned off by default. Customers have control over the content they access, and we are not liable for any resulting damages or liabilities.

For more details, refer to `search_parameters` in [API Reference - Chat completions](https://docs.x.ai/docs/api-reference#chat-completions).

For examples on search sources, jump to [Data Sources and Parameters](https://docs.x.ai/docs/guides/live-search#data-sources-and-parameters).

* * *

[Enabling Search](https://docs.x.ai/docs/guides/live-search#enabling-search)
----------------------------------------------------------------------------

To enable search, you need to specify in your chat completions request an additional field `search_parameters`, with `"mode"` from one of `"auto"`, `"on"`, `"off"`.

The `"mode"` field sets the preference of data source:

*   `"off"`: Disables search and uses the model without accessing additional information from data sources.
*   `"auto"` (default): Live search is available to the model, but the model automatically decides whether to perform live search.
*   `"on"`: Enables live search.

The model decides which data source to use within the provided data sources, via the `"sources"` field in `"search_parameters"`. If no `"sources"` is provided, live search will default to making web and X data available to the model.

For example, you can send the following request, where the model will decide whether to search in data:

```
import os
import requests

url = "https://api.x.ai/v1/chat/completions"
headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {os.getenv('XAI_API_KEY')}"
}
payload = {
    "messages": [
        {
            "role": "user",
            "content": "Provide me a digest of world news in the last 24 hours."
        }
    ],
    "search_parameters": {
        "mode": "auto"
    },
    "model": "grok-3-latest"
}

response = requests.post(url, headers=headers, json=payload)
print(response.json())
```

* * *

[Returning citations](https://docs.x.ai/docs/guides/live-search#returning-citations)
------------------------------------------------------------------------------------

You might want to return to the data source yourself after receiving the inference result. To return the data source citation in the response, you can set `"return_citations": true`. This field defaults to `true`.

```
import os
import requests

url = "https://api.x.ai/v1/chat/completions"
headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {os.getenv('XAI_API_KEY')}"
}
payload = {
    "messages": [
        {
            "role": "user",
            "content": "Provide me a digest of world news in the last 24 hours."
        }
    ],
    "search_parameters": {
        "mode": "auto",
        "return_citations": True
    },
    "model": "grok-3-latest"
}

response = requests.post(url, headers=headers, json=payload)
print(response.json())
```

### [Streaming behavior with citations](https://docs.x.ai/docs/guides/live-search#streaming-behavior-with-citations)

During streaming, you would get the chat response chunks as usual. The citations will be returned as a list of url strings in the field `"citations"` only in the last chunk. This is similar to how the usage data is returned with streaming.

* * *

[Set date range of the search data](https://docs.x.ai/docs/guides/live-search#set-date-range-of-the-search-data)
----------------------------------------------------------------------------------------------------------------

You can restrict the date range of search data used by specifying `"from_date"` and `"to_date"`. This limits the data to the period from `"from_date"` to `"to_date"`, including both dates.

Both fields need to be in ISO8601 format, e.g. "YYYY-MM-DD".

The fields can also be independently used. With only `"from_date"` specified, the data used will be from the `"from_date"` to today, and with only `"to_date"` specified, the data used will be all data till the `"to_date"`.

```
import os
import requests

url = "https://api.x.ai/v1/chat/completions"
headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {os.getenv('XAI_API_KEY')}"
}
payload = {
    "messages": [
        {
            "role": "user",
            "content": "What is the most viral meme in 2022?"
        }
    ],
    "search_parameters": {
        "mode": "auto",
        "from_date": "2022-01-01",
        "to_date": "2022-12-31"
    },
    "model": "grok-3-latest"
}

response = requests.post(url, headers=headers, json=payload)
print(response.json())
```

* * *

[Limit the maximum amount of data sources](https://docs.x.ai/docs/guides/live-search#limit-the-maximum-amount-of-data-sources)
------------------------------------------------------------------------------------------------------------------------------

You can set a limit on how many data sources will be considered in the query via `"max_search_results"`. The default limit is 20.

```
import os
import requests

url = "https://api.x.ai/v1/chat/completions"
headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {os.getenv('XAI_API_KEY')}"
}
payload = {
    "messages": [
        {
            "role": "user",
            "content": "Can you recommend the top 10 burger places in London?"
        }
    ],
    "search_parameters": {
        "mode": "auto",
        "max_search_results": 10
    },
    "model": "grok-3-latest"
}

response = requests.post(url, headers=headers, json=payload)
print(response.json())
```

* * *

[Data sources and parameters](https://docs.x.ai/docs/guides/live-search#data-sources-and-parameters)
----------------------------------------------------------------------------------------------------

In `"sources"` of `"search_parameters"`, you can add a list of sources to be potentially used in search. Each source is an object with source name and parameters for that source, with the name of the source in the `"type"` field.

If nothing is specified, the sources to be used will default to `"web"` and `"x"`.

For example, the following enables web, X search, news and rss:

json

```
"sources": [
  {"type": "web"},
  {"type": "x"},
  {"type": "news"}
  {"type": "rss"}
]
```

### [Overview of data sources and supported parameters](https://docs.x.ai/docs/guides/live-search#overview-of-data-sources-and-supported-parameters)

| Data Source | Description | Supported Parameters |
| --- | --- | --- |
| `"web"` | Searching on websites. | `"country"`, `"excluded_websites"`, `"allowed_websites"`, `"safe_search"` |
| `"x"` | Searching X posts. | `"x_handles"` |
| `"news"` | Searching from news sources. | `"country"`, `"excluded_websites"`, `"safe_search"` |
| `"rss"` | Retrieving data from the RSS feed provided. | `"links"` |

### [Parameter `"country"` (Supported by Web and News)](https://docs.x.ai/docs/guides/live-search#parameter-country-supported-by-web-and-news)

Sometimes you might want to include data from a specific country/region. To do so, you can add an ISO alpha-2 code of the country to `"country"` in `"web"` or `"news"` of the `"sources"`.

```
import os
import requests

url = "https://api.x.ai/v1/chat/completions"
headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {os.getenv('XAI_API_KEY')}"
}
payload = {
    "messages": [
      {
        "role": "user",
        "content": "Where is the best place to go skiing this year?"
      }
    ],
    "search_parameters": {
      "mode": "auto",
      "sources": [{ "type": "web", "country": "CH" }]
    },
    "model": "grok-3-latest"
}

            response = requests.post(url, headers=headers, json=payload)
            print(response.json())
```

### [Parameter `"excluded_websites"` (Supported by Web and News)](https://docs.x.ai/docs/guides/live-search#parameter-excluded_websites-supported-by-web-and-news)

Use `"excluded_websites"`to exclude websites from the query. You can exclude a maximum of five websites.

This cannot be used with `"allowed_websites"` on the same search source.

```
import os
import requests

url = "https://api.x.ai/v1/chat/completions"
headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {os.getenv('XAI_API_KEY')}"
}
payload = {
    "messages": [
        {
            "role": "user",
            "content": "What are some recently discovered alternative DNA shapes?"
        }
    ],
    "search_parameters": {
        "mode": "auto",
        "sources": [
          { "type": "web", "excluded_websites": ["wikipedia.org"] },
          { "type": "news", "excluded_websites": ["bbc.co.uk"] }
        ]
    },
    "model": "grok-3-latest"
}

response = requests.post(url, headers=headers, json=payload)
print(response.json())
```

### [Parameter `"allowed_websites"` (Supported by Web)](https://docs.x.ai/docs/guides/live-search#parameter-allowed_websites-supported-by-web)

Use `"allowed_websites"`to allow only searching on these websites for the query. You can include a maximum of five websites.

This cannot be used with `"excluded_websites"` on the same search source.

```
import os
import requests

url = "https://api.x.ai/v1/chat/completions"
headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {os.getenv('XAI_API_KEY')}"
}
payload = {
    "messages": [
        {
            "role": "user",
            "content": "What are the latest releases at xAI?"
        }
    ],
    "search_parameters": {
        "mode": "auto",
        "sources": [
          { "type": "web", "allowed_websites": ["x.ai"] },
        ]
    },
    "model": "grok-3-latest"
}

response = requests.post(url, headers=headers, json=payload)
print(response.json())
```

### [Parameter `"x_handles"` (Supported by X)](https://docs.x.ai/docs/guides/live-search#parameter-x_handles-supported-by-x)

Use `"x_handles"` to consider X posts only from a given list of X handles.

```
import os
import requests

url = "https://api.x.ai/v1/chat/completions"
headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {os.getenv('XAI_API_KEY')}"
}
payload = {
    "messages": [
      {
        "role": "user",
        "content": "What are the latest updates on Grok?"
      }
    ],
    "search_parameters": {
      "mode": "auto",
      "sources": [{ "type": "x", "x_handles": ["grok"] }]
    },
    "model": "grok-3-latest"
}

response = requests.post(url, headers=headers, json=payload)
print(response.json())
```

You can also fetch data from a list of RSS feed urls via `{ "links": ... }`. You can one RSS link at the moment.

For example:

```
import os
import requests

url = "https://api.x.ai/v1/chat/completions"
headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {os.getenv('XAI_API_KEY')}"
}
payload = {
    "messages": [
      {
        "role": "user",
        "content": "What are the latest updates on Grok?"
      }
    ],
    "search_parameters": {
      "mode": "on",
      "sources": [{ "type": "rss", "links": ["https://status.x.ai/feed.xml"] }]
    },
    "model": "grok-3-latest"
}

response = requests.post(url, headers=headers, json=payload)
print(response.json())
```

### [Parameter `"safe_search"` (Supported by Web and News)](https://docs.x.ai/docs/guides/live-search#parameter-safe_search-supported-by-web-and-news)

Safe search is on by default. You can disable safe search for `"web"` and `"news"` via `"sources": [{..., "safe_search": false }]`.
