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


      // The cornerstone, an `each` implementation, aka `forEach`.
      // Handles objects implementing `forEach`, arrays, and raw objects.
      // Delegates to **ECMAScript 5**'s native `forEach` if available.
    var each = _.each = _.forEach = function(obj, iterator, context) {
        if (obj == null) return;
        // ネイティブにあるならそれを使う
        if (nativeForEach && obj.forEach === nativeForEach) {
            obj.forEach(iterator, context);
        } else if (_.isNumber(obj.length)) {// lengthがあるかないかで
            for (var i = 0, l = obj.length; i < l; i++) {
                // callback=iteratorにはelement, index, arrayが渡される。
                // callを使ってcontextを決める
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

## reduce
