extends: post.liquid

title: Adding search to your cobalt site - Part One
date: 20 Jun 2017 08:06:11 +0100

path: 2017/adding-search-to-your-cobalt-site-part-one
---
This will be a two part post, where I detail the steps it took to enable
search on my Cobalt site.

In this first post I will detail how to integrate lunr using a manually created 
index file. If you already know how to do this you can skip to [second post](http://booyaa.wtf/2017/adding-search-to-your-cobalt-site-part-two), where I 
create the index file using a liquid template.

This is more of a note to myself, I will explain the details at a later date.

## search.liquid

The search template looks like this.

### frontmatter

```
extends: default.liquid

title: search
path:  search/
---
```

### liquid template

```html
<input id="search" type="text" size="25" placeholder="search for stuff here..." 
       autofocus><br />

<ul id="results">
</ul>

<script type="text/javascript" 
        src="https://code.jquery.com/jquery-2.1.3.min.js"></script>
<script type="text/javascript" src="https://unpkg.com/lunr/lunr.js"></script>
<script type="text/javascript">
var lunrIndex,
    $results,
    pagesIndex;

// This is pretty much a copy of 
// https://gist.github.com/sebz/efddfc8fdcb6b480f567
// Initialize lunrjs using our generated index file
function initLunr() {
    // First retrieve the index file
    $.getJSON("/js/lunr_index.json")
        .done(function(index) {
            pagesIndex = index;

            // Set up lunrjs by declaring the fields we use
            // Also provide their boost level for the ranking
            lunrIndex = lunr(function() {
                this.field("title", {
                    boost: 10
                });

                this.field("content");

                // ref is the result item identifier (I chose the page URL)
                this.ref("href");
                
                // Feed lunr with each file and let lunr actually index them
                pagesIndex.forEach(function(page) {
                    this.add(page)
                }, this);
            });
            
        })
        .fail(function(jqxhr, textStatus, error) {
            var err = textStatus + ", " + error;
            console.error("Error getting cobalt index file:", err);
        });
}

// Nothing crazy here, just hook up a listener on the input field
function initUI() {
    $results = $("#results");
    $("#search").keyup(function() {
        $results.empty();

        // Only trigger a search when 2 chars. at least have been provided
        var query = $(this).val();
        if (query.length < 2) {
            return;
        }

        var results = search(query);

        renderResults(results);
    });
}

/**
    * Trigger a search in lunr and transform the result
    *
    * @param  {String} query
    * @return {Array}  results
    */
function search(query) {
    // Find the item in our index corresponding to the lunr one to have more 
    // info
    // Lunr result: 
    //  {ref: "/section/page1", score: 0.2725657778206127}
    // Our result:
    //  {title:"Page1", href:"/section/page1", ...}
    return lunrIndex.search(query).map(function(result) {
            return pagesIndex.filter(function(page) {
                return page.href === result.ref;
            })[0];
        });
}

/**
    * Display the 10 first results
    *
    * @param  {Array} results to display
    */
function renderResults(results) {
    if (!results.length) {
        return;
    }

    // Only show the ten first results
    results.slice(0, 10).forEach(function(result) {
        var $result = $("<li style=\"list-style:none;\">"); // FUUUUUU!
        $result.append($("<a>", {
            href: result.href,
            text: "» " + result.title
        }));
        $results.append($result);
    });
}

// Let's get started
initLunr();

$(document).ready(function() {
    initUI();
});
</script>

```