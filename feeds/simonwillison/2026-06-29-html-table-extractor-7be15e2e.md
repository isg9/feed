---
title: HTML table extractor
url: https://simonwillison.net/2026/Jun/29/html-table-extractor/#atom-everything
published: "2026-06-29T23:38:21Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jun/29/html-table-extractor/#atom-everything
---

# HTML table extractor

**Tool:** [HTML table extractor](https://tools.simonwillison.net/html-table-extractor)

Yet another in my growing collection of paste-conversion tools. This one accepts pasted rich text from browsers (with embedded HTML tables) and converts every detected table into HTML, Markdown, CSV, TSV, or JSON.

Try it out by selecting everything on the Wikipedia [List of cities and towns in the San Francisco Bay Area](https://en.wikipedia.org/wiki/List_of_cities_and_towns_in_the_San_Francisco_Bay_Area) page and pasting it directly into the tool:

![Screenshot of a web interface for converting table data between formats. A row of tabs labeled HTML, Markdown, CSV, TSV, and JSON sits below the bottom edge of a styled data table, with the TSV tab currently selected. The TSV tab displays the table's contents as tab-separated plain text in a monospaced font inside a bordered panel, with a "Copy" button in the upper right of that panel.](https://static.simonwillison.net/static/2026/html-table.jpg)

On a similar note, I recently [rebuilt](https://github.com/simonw/tools/commit/f278e977751dbc1948baedfc2f26b6de870f60e6) my [Rich text to markdown](https://tools.simonwillison.net/rich-text-to-markdown) tool to add support for tables and generally improve the UI.

**Update**: It turns out Wikipedia has an open CORS API for retrieving the full rendered HTML content of any page - [demo here](https://tools.simonwillison.net/cors-fetch#url=https%3A%2F%2Fen.wikipedia.org%2Fw%2Fapi.php%3Faction%3Dparse%26page%3DList_of_cities_and_towns_in_the_San_Francisco_Bay_Area%26prop%3Dtext%26format%3Djson%26origin%3D%2A) \- so I [had Codex](https://gist.github.com/simonw/f226fe96f464ec7d81d6996cb466436d) add the ability to search Wikipedia for a page and then automatically import and display any tables from that page.

Tags: [html](https://simonwillison.net/tags/html), [tools](https://simonwillison.net/tags/tools), [wikipedia](https://simonwillison.net/tags/wikipedia), [cors](https://simonwillison.net/tags/cors)
