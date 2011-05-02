@Title DropPages Tips
@description DropPages関係のTips
@Dates 2011年05月03日
@Body
[DropPages]で運営しているので、[DropPages]関係のTipsなどを書いていく場所

##Markdown
DropPagesはMarkdown形式でContentを書いていける。
そのため、デフォルトテーマだとtxtなどの拡張子になってるけど、mdやmarkdownなどの拡張子でも問題なく使える。

元々WebStormの[Markdown](http://plugins.intellij.net/plugin/?webide&amp;id=5970 "Markdown")プラグインがあって、
これでコード読みながらMarkdownでいろいろ書けたら捗りそうだなと思ったところでDropPagesができてたので、WebStormを使って
書いていく。

JetBrains製IDEにはスニペット的なFile Templeteがあるので、以下のように最初にDropPagesに使えそうな変数をあらかじめ作って書くようにする。

    @Title ${Title}
    #if (${description} != "")@description ${description}
    #end
    #if (${keywords} != "")@keywords ${keywords}
    #end
    @Dates ${YEAR}年${MONTH}月${DAY}日
    @Body

併せてテンプレートの方にも{{変数}}という感じで参照できて、

>  {{変数|プレースホルダ}}

という感じで、変数がなかったときの初期値もかけるので意外とテンプレートも工夫できる。

このサイトのテンプレートとかを含めて[azu/4-Code-Reading - GitHub]に置いてある。


[DropPages]: http://droppages.com/
[azu/4-Code-Reading - GitHub]: https://github.com/azu/4-Code-Reading "azu/4-Code-Reading - GitHub"