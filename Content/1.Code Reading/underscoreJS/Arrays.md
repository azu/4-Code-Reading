@Title Array Functions of underscore.js
@description underscore.jsのArray関係
@keywords underscore.js
@Dates 2011年05月14日
@Body

## first

headにもエイリアスが貼られている。

> _.first(array, [n])

配列の一番目の要素を返す。nを指定しすると、先頭からn番目の要素までの配列を返す。

> _.first([5, 4, 3, 2, 1]);
// => 5

    // Get the first element of an array. Passing **n** will return the first N
    // values in the array. Aliased as `head`. The **guard** check allows it to work
    // with `_.map`.
    _.first = _.head = function(array, n, guard) {
        // nが無指定、guardも無指定 => 先頭　array[0]
        // nが指定 => slice
        // guardは_.mapと一緒に動くため
        return (n != null) && !guard ? slice.call(array, 0, n) : array[0];
    };

## rest

tailにもエイリアスされている。

> _.rest(array, [index])

先頭の要素を取り除いた配列を返す。indexを指定すると、index番目以降の要素を配列として返す。

> _.rest([5, 4, 3, 2, 1]);
//=> [4, 3, 2, 1]

    // Returns everything but the first entry of the array. Aliased as `tail`.
    // Especially useful on the arguments object. Passing an **index** will return
    // the rest of the values in the array from that index onward. The **guard**
    // check allows it to work with `_.map`.
    _.rest = _.tail = function(array, index, guard) {
        // indexがどっちであっても、結局はindex以降をsliceするだけ
        return slice.call(array, (index == null) || guard ? 1 : index);
    };

## last

> _.last(array)

配列の最後にある要素を返す。

> _.last([5, 4, 3, 2, 1]);
//=> 1

何も言うことないぐらいシンプル…

    // Get the last element of an array.
    _.last = function(array) {
        return array[array.length - 1];
    };

## compact

>_.compact(array)

falsyなfalse, null, 0, "", undefinedを取り除いた配列を返す。非破壊的。

> _.compact([0, 1, false, 2, '', 3]);
//=> [1, 2, 3]

疎な配列から密な配列を作るのに使えそう。

    // Trim out all falsy values from an array.
    _.compact = function(array) {
        // filterを使ってtrueなものだけを残す
        return _.filter(array, function(value) {
            // !! でvalueを真偽値に変換する
            return !!value;
        });
    };

## flatten

> _.flatten(array)

多次元配列を一次元化する。非破壊的。
配列の配列みたいのを平らにするメソッド

> _.flatten([1, [2], [3, [[[4]]]]]);
// => [1, 2, 3, 4];

reduceを使って、配列を順番に平らにしてくっつけたものを返してくれる。

    // Return a completely flattened version of an array.
    _.flatten = function(array) {
        return _.reduce(array, function(memo, value) {
            // memoはresultにあたるもの,初期値はreduceの第三引数[]になる。
            // 配列を見つけたら再帰
            if (_.isArray(value)){
                // 配列をconcatで分解してくっつける
                return memo.concat(_.flatten(value));
            }
            // 配列じゃない普通のものはpushして結果に追加する
            memo[memo.length] = value;
            return memo;
        }, []);// memo初期値
    };

同じ機能の簡単な書き方として、

> Array.concat.apply(0,[1, [2], [3, [[[4]]]]]);

みたいのがあるけど、こっちは破壊的になる。

## without

> _.without(array, [*values])

valuesを取り除いた配列を返す。===演算子で比較される。非破壊的。
> _.without([1, 2, 1, 0, 3, 1, 4], 0, 1);
// => [2, 3, 4]

内容的にはfilterとinclude(contain)の組み合わせ。

    // Return a version of the array that does not contain the specified value(s).
    _.without = function(array) {
        var values = slice.call(arguments, 1);// 第二引数以降をvaluesとして取得
        return _.filter(array, function(value) {
            // includeはvaluesにvalueを含んでるものがあればtrueを返す
            // includeの結果がtrueのものだけを取り除く
            return !_.include(values, value);
        });
    };

## uniq

> _.uniq(array, [isSorted])

配列から重複を取り除く。===演算子で比較される。配列がソートされている場合は、isSrotedをtrueにすると処理が早くなる。非破壊的。

> _.uniq([1, 2, 1, 3, 1, 4]);
// => [1, 2, 3, 4]

reduceで一個づつ見ていって、ソートの場合は処理を単純化している。

    // Produce a duplicate-free version of the array. If the array has already
    // been sorted, you have the option of using a faster algorithm.
    // Aliased as `unique`.
    _.uniq = _.unique = function(array, isSorted) {
        return _.reduce(array, function(memo, el, i) {
            if (0 == i || (isSorted === true
                    ? _.last(memo) != el // arrayがsortされているなら、最後の要素だけをみればいい
                    // memoにはpushで追加してるので、同じものが連続的にくる=重複
                    : !_.include(memo, el))){ // ソートされてないときはincludeで含んでいるかを調べる
                // memoにまだ無い要素だったらpushする
                memo[memo.length] = el;
            }
            return memo;
        }, []);// memoの初期値
    };

## intersect

> _.intersect(*arrays)

複数の配列の共通集合を返す。

>_.intersect([1, 2, 3], [101, 2, 1, 10], [2, 1]);
// => [1, 2]

結構複雑な感じがしたけど、どれか基準となる配列を中心に考えれば、単純かされる。_.indexOfを使ってるのはネイティブindexOfがあるからだと思う。

    // Produce an array that contains every item shared between all the
    // passed-in arrays.
    _.intersect = function(array) {
        // 引数はarrayに限定されるので、どれか一つのarrayを基準にfilterしていく
        var rest = slice.call(arguments, 1);
        // 先頭のarrayを基準にする、同じ値を複数持っていると処理が増えるので、
        // 先にuniq(array)しておく
        return _.filter(_.uniq(array), function(item) {
            // itemが他の配列の中に含まれているかを調べる
            // filterはtrueなものだけの配列を作る
            return _.every(rest, function(other) { // everyは全てがtrueならtrueを返す
                return _.indexOf(other, item) >= 0;// 配列なので_.indexOfで調べるほうが効率よい
            });
        });
    };

## zip

> _.zip(*arrays)

複数の配列の、同じ位置にある要素をマージする。

> _.zip(['moe', 'larry', 'curly'], [30, 40, 50], [true, false, false]);
// => [["moe", 30, true], ["larry", 40, false], ["curly", 50, false]]

pluckを上手く使って結構簡単に書かれてる

  // Zip together multiple lists into a single array -- elements that share
    // an index go together.
    _.zip = function() {
        var args = slice.call(arguments);
        var length = _.max(_.pluck(args, 'length'));// argsで最大の長さの配列に合わせる
        var results = new Array(length);
        for (var i = 0; i < length; i++){
            // pluckでそれぞれのi番目の要素を集めた配列をresultへいれる
            results[i] = _.pluck(args, "" + i);
        }
        return results;
    };


## indexOf

> _.indexOf(array, value, [isSorted])

配列にvalueが含まれていた場合、valueが一番最初に現れた位置を返す。含まれていなかった場合は-1を返す。配列がソートされている場合は、isSrotedをtrueにすると処理が早くなる。

>_.indexOf([1, 2, 3], 2);
//=> 1

IEなど配列のindexOfが無いがサポートされてないものでもindexOfが使えるようにする。
ソート済みならバイナリサーチなsortedIndexを使用する

    // If the browser doesn't supply us with indexOf (I'm looking at you, **MSIE**),
    // we need this function. Return the position of the first occurrence of an
    // item in an array, or -1 if the item is not included in the array.
    // Delegates to **ECMAScript 5**'s native `indexOf` if available.
    // If the array is large and already in sort order, pass `true`
    // for **isSorted** to use binary search.
    _.indexOf = function(array, item, isSorted) {
        if (array == null) return -1;
        var i, l;
        if (isSorted) {
            // sortされてるならバイナリサーチできる
            i = _.sortedIndex(array, item);
            return array[i] === item ? i : -1;
        }
        if (nativeIndexOf && array.indexOf === nativeIndexOf) return array.indexOf(item);
        // 最悪な場合は逐次探索していく
        for (i = 0,l = array.length; i < l; i++) if (array[i] === item) return i;
        return -1;
    };

## lastIndexOf

> _.lastIndexOf(array, value)

配列にvalueが含まれていた場合、valueが一番最後に現れた位置を返す。含まれていなかった場合は-1を返す。

> _.lastIndexOf([1, 2, 3, 1, 2, 3], 2);
// => 4

後ろから調べていくだけ

    // Delegates to **ECMAScript 5**'s native `lastIndexOf` if available.
    _.lastIndexOf = function(array, item) {
        if (array == null) return -1;
        if (nativeLastIndexOf && array.lastIndexOf === nativeLastIndexOf) return array.lastIndexOf(item);
        var i = array.length;
        // 後ろから逐次探索する
        while (i--) if (array[i] === item) return i;
        return -1;
    };

## range

>_.range([start], stop, [step])

startからstopまでstepずつ増加(または減少)する整数のリストを作る。排他的。
startのデフォルト値は0。stepのデフォルト値は1。

    _.range(10);
    => [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
    _.range(1, 11);
    => [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    _.range(0, 30, 5);
    => [0, 5, 10, 15, 20, 25]
    _.range(0, -10, -1);
    => [0, -1, -2, -3, -4, -5, -6, -7, -8, -9]
    _.range(0);
    => []

Pythonからもってきたもの。

    // Generate an integer Array containing an arithmetic progression. A port of
    // the native Python `range()` function. See
    // [the Python documentation](http://docs.python.org/library/functions.html#range).
    _.range = function(start, stop, step) {
        if (arguments.length <= 1) { // 引数が一個なら0-startまでのリストを作る
            stop = start || 0;
            start = 0;
        }
        step = arguments[2] || 1;
        // 何個になるから調べる
        var len = Math.max(Math.ceil((stop - start) / step), 0);
        var idx = 0;
        var range = new Array(len);

        while (idx < len) {
            range[idx++] = start;
            // step分足す
            start += step;
        }

        return range;
    };

これでArray編は終わり

次は[Function](Functions)へ