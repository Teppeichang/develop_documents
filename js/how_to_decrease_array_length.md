# JavaScriptで配列の長さを切り詰める方法

## 言葉の定義

配列の長さを切り詰める = 配列の要素の削除で、lengthが減るもの。

## 配列の長さを切り詰める方法


- length を使う
- splice() を使う

この２通り。(lengthを使った方法は、記事作成時に初めて知った)

配列の要素の削除するメソッドとして、delete()もあるが・・・

これは結論から言うと**lengthは減らない**ので、配列の長さを切り詰める方法には該当しない。

## length


配列オブジェクトの `length` プロパティは、通常配列の長さを調べるために参照するが、数値を代入することで配列のサイズを変更するのにも使用できる。

 `length` プロパティに、現在の配列サイズよりも小さい値を代入すると、配列のサイズが切り詰められる。

```jsx
var a = [1, 2, 3, 4, 5];
a.length = 3;
console.log(a.length);  //=> 3
console.log(a);         //=> [ 1, 2, 3 ]
```

## splice


配列の `splice()` メソッドは、指定したインデックスから指定した数だけの要素を抽出した配列を返す。

```jsx
var a = [1, 2, 3, 4, 5, 6, 7];
console.log(a.splice(2, 3));  // [ 3, 4, 5 ]
```

`splice()` は同時に、元の配列から抽出した要素を削除する。

 つまり、配列の中間要素を削除する形で配列を切り詰めることができる。

```jsx
var a = [1, 2, 3, 4, 5, 6, 7];
a.splice(2, 3); // [2]番目から3つ要素を削除
console.log(a);  //=> [ 1, 2, 6, 7 ]
```

## deleteではダメなのか


`delete` を使って配列の要素を削除しても、配列のサイズを切り詰めることはできない。

その位置の値が `undefined` になるだけで、配列のサイズは変わらない。

```jsx
var a = [1, 2, 3];
delete a[2];
console.log(a.length);  //=> 3
console.log(a);         //=> [ 1, 2, ] 正確には[1, 2, undefined]
```

## 参照記事


[https://maku77.github.io/js/array/cut.html](https://maku77.github.io/js/array/cut.html)