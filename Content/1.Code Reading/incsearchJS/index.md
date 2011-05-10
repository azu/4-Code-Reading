@Title incsearch.js
@description incsearch.jsのコードリーディング
@keywords incremental,search
@Dates 2011年05月06日
@Body

インクリメンタル検索を行える[incsearch.js]を読んだ記録

[incsearch.js code reading - jsdo.it](http://jsdo.it/azu_re/3Q2c/fullscreen "incsearch.js code reading - jsdo.it - Share JavaScript, HTML5 and CSS")

Doccoを使ってソースコードとコメントを振り分けて表示してる。

## 内容

多く分けて

- IncSearch.ViewBase
- IncSearch.ViewList
- IncSearch.ViewTable
- IncSearch.ViewBookmark

となってて、IncSearch.ViewBaseにほとんどの部分が書かれてて、
最後にIncSearch.ViewBaseを継承してIncSearch.ViewListなどを作るって感じになってた。

[incsearch.js]: http://www.enjoyxstudy.com/javascript/incsearch/ "incsearch.js"