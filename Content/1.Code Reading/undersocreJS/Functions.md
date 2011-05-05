@Title Functions of undersocore.js
@keywords undersocore.js
@Dates 2011年05月03日
@Body

ループでよく出てくるbreakerというオブジェクトは最初に定義されてる。

    // Establish the object that gets returned to break out of a loop iteration.
    var breaker = {};

列挙するときに終了判定に使われてるようだ。

## each(forEach)
forEach自体については[forEach - MDC Docs](https://developer.mozilla.org/ja/JavaScript/Reference/Global_Objects/Array/forEach "forEach - MDC Docs")から

> _.each(list, iterator, [context]);// contextはthisとなるオブジェクトの指定

eachは結構他の関数から多様されている。

      // The cornerstone, an `each` implementation, aka `forEach`.
      // Handles objects implementing `forEach`, arrays, and raw objects.
      // Delegates to **ECMAScript 5**'s native `forEach` if available.
      // さりげなくeachという関数も作ってる
    var each = _.each = _.forEach = function(obj, iterator, context) {
        if (obj == null) return;
        // ネイティブにあるならそれを使う
        if (nativeForEach && obj.forEach === nativeForEach) {
            obj.forEach(iterator, context);
        } else if (_.isNumber(obj.length)) {// lengthがあるかないかで
            for (var i = 0, l = obj.length; i < l; i++) {
                // callback=iteratorにはelement, index, arrayが渡される。
                // callを使ってcontextを決める
                // iteratorでbreakerを返されたときはそこで終了する
                if (iterator.call(context, obj[i], i, obj) === breaker) return;
            }
        } else {
            for (var key in obj) {
                if (hasOwnProperty.call(obj, key)) {
                    if (iterator.call(context, obj[key], key, obj) === breaker) return;
                }
            }
        }
    };


 undersocre.jsの方でnativeのES5メソッドはエイリアスがついてるので、それをみて実装するかを決めてる。

    // All **ECMAScript 5** native function implementations that we hope to use
    // are decla    red here.
    var
    nativeForEach      = ArrayProto.forEach,
    nativeMap          = ArrayProto.map,
    nativeReduce       = ArrayProto.reduce,
    nativeReduceRight  = ArrayProto.reduceRight,
    nativeFilter       = ArrayProto.filter,
    nativeEvery        = ArrayProto.every,
    nativeSome         = ArrayProto.some,
    nativeIndexOf      = ArrayProto.indexOf,
    nativeLastIndexOf  = ArrayProto.lastIndexOf,
    nativeIsArray      = Array.isArray,
    nativeKeys         = Object.keys,
    nativeBind         = FuncProto.bind;

## map

mapは中身的にもほぼeachと同じで、最後に結果を返すぐらいの違いしかない
result.pushの代わりにlengthを使って直接入れてる

    // Return the results of applying the iterator to each element.
    // Delegates to **ECMAScript 5**'s native `map` if available.
    _.map = function(obj, iterator, context) {
        var results = [];
        if (obj == null) return results;
        if (nativeMap && obj.map === nativeMap) return obj.map(iterator, context);
        each(obj, function(value, index, list) {
            // results.lengthは0,1,2..と変化 = pushの代わり
            results[results.length] = iterator.call(context, value, index, list);
        });
        return results;
    };

## reduce

> _.inject , _.foldl

という名前でも使えるようになってる。

    var sum = _.reduce([1, 2, 3], function(memo, num){ return memo + num; }, 0);
    => 6

第二引数に[イテレータ](http://ja.wikipedia.org/wiki/%E3%82%A4%E3%83%86%E3%83%AC%E3%83%BC%E3%82%BF "イテレータ")となる関数を指定する。
それぞれ渡されるものは以下のようになってるので、iterator.callもその順番

> iterator(previousValue, currentValue, index, array)


-[Reduce - MDC Docs](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Array/Reduce "Reduce - MDC Docs")


    // **Reduce** builds up a single result from a list of values, aka `inject`,
    // or `foldl`. Delegates to **ECMAScript 5**'s native `reduce` if available.
    _.reduce = _.foldl = _.inject = function(obj, iterator, memo, context) {
        // memoは初期値と指定するもの
        var initial = memo !== void 0;// undefinedの代わりにvoid 0
        if (obj == null) obj = [];
        if (nativeReduce && obj.reduce === nativeReduce) {
            if (context) iterator = _.bind(iterator, context);
            return initial ? obj.reduce(iterator, memo) : obj.reduce(iterator);
        }
        each(obj, function(value, index, list) {
            if (!initial && index === 0) {// 初期値がなくて、初回の時
                memo = value;// objの最初の要素がvalue
                initial = true;
            } else {
                // 引数は previousValue, currentValue, index, array
                memo = iterator.call(context, memo, value, index, list);
            }
        });
        if (!initial) throw new TypeError("Reduce of empty array with no initial value");
        return memo;
    };


中身ではeachを使い、初回とそれ以外で分けて処理している。(最初に分けておかないのは見やすく書くためなのかな?)
もし、reduceの引数にmemo(初期値)が指定されてないなら、先頭の要素をmemoに入れて、後はひたすらiteratorをcallしてその結果をmemoに入れていく。

## reduceRight

[reduceRight](https://developer.mozilla.org/ja/JavaScript/Reference/Global_Objects/Array/ReduceRight "reduceRight - MDC Docs")は反転させてreduceかけるだけ(うわ手抜き…)まああんまり使われてない印象もあるし、ネイティブが対応してればいいし、とりあえず動くって感じでやったのかな。

    // The right-associative version of reduce, also known as `foldr`.
    // Delegates to **ECMAScript 5**'s native `reduceRight` if available.
    _.reduceRight = _.foldr = function(obj, iterator, memo, context) {
        if (obj == null) obj = [];
        if (nativeReduceRight && obj.reduceRight === nativeReduceRight) {
            if (context) iterator = _.bind(iterator, context);
            return memo !== void 0 ? obj.reduceRight(iterator, memo) : obj.reduceRight(iterator);
        }
        // objを反転させてる
        var reversed = (_.isArray(obj) ? obj.slice() : _.toArray(obj)).reverse();
        return _.reduce(reversed, iterator, memo, context);//後はreduce
    };

## filter

これも基本的にitelatorをeachで回して処理するだけ。
undersocre.jsではpushではなくresults[results.length]を使うみたい。
[push vs. \[\].length · jsPerf](http://jsperf.com/push-vs-lengthf00 "push vs. [].length · jsPerf")を見るとどっちもどっちかな

    // Return all the elements that pass a truth test.
    // Delegates to **ECMAScript 5**'s native `filter` if available.
    // Aliased as `select`.
    _.filter = _.select = function(obj, iterator, context) {
        var results = [];
        if (obj == null) return results;
        if (nativeFilter && obj.filter === nativeFilter) return obj.filter(iterator, context);
        each(obj, function(value, index, list) {
            // iterator関数がtrulyなものを返せばresultに追加
            if (iterator.call(context, value, index, list)) results[results.length] = value;
        });
        return results;
    };

## reject

filterの逆。特に_.filterを使わないで判定に!をいれたぐらいの違い。
> var odds = _.reject([1, 2, 3, 4, 5, 6], function(num){ return num % 2 == 0; });// [1, 3, 5]


    // Return all the elements for which a truth test fails.
    _.reject = function(obj, iterator, context) {
        var results = [];
        if (obj == null) return results;
        each(obj, function(value, index, list) {
            if (!iterator.call(context, value, index, list)) results[results.length] = value;
        });
        return results;
    };

## every

_.allにエイリアスされている。

> _.all(list, iterator, [context])

[Array.prototype.every](http://es5.github.com/#x15.4.4.16 "Array.prototype.every")の仕様を見た感じ、ループ中にfalseになったらfalseを返すで、そうでない時はtrueを返すと言う感じになってるので、listが空ならそもそもループに入らないのでtrueになる。

> _.all([], function(){return});// true

実際、下のようなネイティブなものもtrueになる。

    [].every(function(element, index, array) {
      return false;
    });// true

コードを見るとreturn breakerってのが初めて出てきた。
eachでbreakするために最初の頃にvarで定義されてる。

    // Determine whether all of the elements match a truth test.
    // Delegates to **ECMAScript 5**'s native `every` if available.
    // Aliased as `all`.
    _.every = _.all = function(obj, iterator, context) {
        var result = true;
        // objがないならtrueを返す
        if (obj == null) return result;
        //if (nativeEvery && obj.every === nativeEvery) return obj.every(iterator, context);
        each(obj, function(value, index, list) {
            // 一個でもfalseな結果になったらresult=falseしてbreakする
            if (!(result = result && iterator.call(context, value, index, list))) return breaker;
        });
        return result;// true or false
    };

> result = result && iterator.call(context, value, index, list)

が、ただ単にresult = resultしてるように見えて、

> result = (result && iterator.call(context, value, index, list)))

が実際の意味だけど、なんでresult代入してるんだろ?と見えてしまってた…


## find

ドキュメントには載ってない

    // Return the first value which passes a truth test. Aliased as `detect`.
    _.find = _.detect = function(obj, iterator, context) {
        var result;
        any(obj, function(value, index, list) {
            if (iterator.call(context, value, index, list)) {
                result = value;
                return true;
            }
        });
        return result;
    };
