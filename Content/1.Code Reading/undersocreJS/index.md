@Title Underscore.js
@description 便利関数の詰め合わせライブラリUnderscore.jsのコードリーディング
@keywords Underscore.js
@Dates 2011年05月03日
@Body
[Underscore.js] 1.1.6のソースコードを見ていく

Table of Contentsによるとそれぞれジャンル分けされているので、それに沿って読む。

* [Collections](undersocreJS/Collections)
* [Arrays](undersocreJS/Arrays)
* [Functions](undersocreJS/Functions)
* [Objects](undersocreJS/Objects)
* [Utility](undersocreJS/Utility)
* [Chaining](undersocreJS/Chaining)

最初の初期化とか、取り決めらへん

##グローバル変数
Underscore.jsは_というグローバル変数のみを持っていて、_以下に便利な関数を
詰め込んだ感じのライブラリです。

    (function() {
      // Establish the root object, `window` in the browser, or `global` on the server.
      // rootになるオブジェクト、ブラウザだとwindowだけどunderscore.jsはブラウザ以外でも使えるのでthisでグローバルをとる
      var root = this;

      // Save the previous value of the `_` variable.
      // グローバル._を使う
      var previousUnderscore = root._;

      // Establish the object that gets returned to break out of a loop iteration.
      var breaker = {};

      /* 中身 */
    })();

全体を即時実行関数にして、thisでグローバルをとってそこに追加していくという感じになっている。

ライブラリのグローバル変数を調べるには[Global is the new private](http://mankz.com/code/GlobalCheck.htm "Global is the new private")が便利です。

    // Create a safe reference to the Underscore object for use below.
    var _ = function(obj) { return new wrapper(obj); };

_自体を実行したときもラップしたのが返ってくる

> _(obj);// new wrapper(obj);

## CommonJS

CommonJS的なルールに沿ってmodule.exports = _;をしておいてる。
Node.jsとかその辺でも使えるようにするための配慮かな

      // Export the Underscore object for **CommonJS**, with backwards-compatibility
      // for the old `require()` API. If we're not in CommonJS, add `_` to the
      // global object.
      if (typeof module !== 'undefined' && module.exports) {
        module.exports = _;
        _._ = _;
      } else {
        root._ = _;
      }


次は [Functions](undersocreJS/Functions) へ

[Underscore.js]: http://documentcloud.github.com/underscore/