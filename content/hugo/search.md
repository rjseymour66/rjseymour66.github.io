+++
title = 'Search'
date = '2025-07-13T09:32:07-04:00'
weight = 80
draft = false
+++


Sites with database-driven searches have built-in search capabilities. The site takes the search terms and builds a query that is sent to the database that contains the content. The db returns the matching results, and the site displays it to the user.

Hugo doesn't use a database, so you need a solution that can index your site and search it:
- Algolia indexes your content and lets you search it
- ElasticSearch indexes your content
- You can generate your own search index and use client-side JS to perform the search

To generate your own search engine, you need Hugo to generate a list of your site content in JSON format.

### Creating the document collection

Lunr search requires these files:
- `theme/docsite/layouts/search.json`: Search index. Create this with Hugo templates.
- `content/search.md`: Search page. This has no content, only the output formats. The `search.json` file outputs the JSON to this page.
- `theme/docsite/layouts/search.html`: The search interface.
- `theme/docsite/static/js/search.js`: The JS search logic.



Your content list needs the page title and some text to search.


#### search.json

Create a search.json layout:

```json
{
    "results": [
        {{- range $index, $page := .Site.RegularPages }}
        {{- if $index -}} , {{- end }}
        {
            "href": {{ .Permalink | jsonify }},
            "title": {{ .Title | jsonify }},
            "body": {{ .Content | plainify | jsonify }}
        }
        {{- end }}
    ]
}
```

#### search.md

Create search content page. It won't have content--you need it to specify the output formats:

```toml
+++
...
outputs = ["HTML", "JSON"]
layout = 'search'
+++
```
This is enough to generate the search page at `/search/index.json`.

#### search.html

Next, create the search page where the user enters the search terms:

```go
{{ define "main" }}

    <h2>{{ .Title }}</h2>

    <input type="search" id="searchField">
    <button id="searchButton">Search</button>
    <input type="checkbox" id="allwords">
    <label for="allwords">Require all words</label>
    
    <div id="output">
        <p>Waiting for search input</p>
    </div>

    // lunr search library CDN
    <script src="//unpkg.com/lunr@2.3.6/lunr.js"></script>
    <script src="{{ "js/search.js" | relURL }}"></script>

{{ end }}
```

#### search.js

Fetch the JSON file and populate the search index:

```js
'use strict';

window.SearchApp = {
    searchField: document.querySelector("#searchField"),
    searchButton: document.querySelector("#searchButton"),
    allwords: document.querySelector('#allwords'),
    output: document.querySelector("#output"),
    searchData: {},
    searchIndex: {}
};

fetch('/search/index.json')
    .then(response => {
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        return response.json();
    })
    .then(data => {
        SearchApp.searchData = data;
        SearchApp.searchIndex = lunr(function () {
            this.pipeline.remove(lunr.stemmer);
            this.searchPipeline.remove(lunr.stemmer);
            this.ref('href');
            this.field('title');
            this.field('body');
            data.results.forEach(e => {
                this.add(e);
            });
        });
    })
    .catch(error => {
        console.error('Error loading search index:', error);
    });
```

Next, build the search function and add an event listener to the search button on the search page:

```js
const search = () => {
    let searchText = SearchApp.searchField.value;
    if (searchText === '') return;          // return if empty

    searchText = searchText
        .split(' ')
        .map(word => `${word}*`)
        .join(' ');

    if (SearchApp.allwords.checked) {
        searchText = searchText
            .split(' ')
            .map(word => `+${word}`)
            .join(' ');
    }
    
    let resultList = SearchApp.searchIndex.search(searchText);

    let list = [];
    let results = resultList.map(entry => {
        SearchApp.searchData.results.filter(d => {
            if (entry.ref == d.href) {
                list.push(d);
            }
        });
    });
    display(list);
};

const display = (list) => {
    SearchApp.output.innerText = '';
    if (list.length > 0) {
        const ul = document.createElement('ul');
        list.forEach(el => {
            const li = document.createElement('li');
            const a = document.createElement('a');
            a.href = el.href;
            a.text = el.title;
            li.appendChild(a);
            ul.appendChild(li);
        });
        SearchApp.output.appendChild(ul);
    } else {
        SearchApp.output.innerHTML = "Nothing found";
    }
};

SearchApp.searchButton.addEventListener('click', search);
```
