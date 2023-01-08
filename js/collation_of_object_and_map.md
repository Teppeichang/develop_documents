# オブジェクト・Mapのキーの列挙順は保証される or されない？

## 結論


- オブジェクト: 使用するメソッドによっては保証される**こともある**が、**基本的に保証されないものとして扱うべき**。
- `Map`: 挿入順が保証される。

ベストプラクティス：Mapを使おう

## 本記事の内容を調べるに至った経緯


実務で開発中の広告レポート作成サービス開発において、Facebook APIからGETした広告指標（オブジェクト）をDBにINSERTする際、「オブジェクトのプロパティ順と同じ順番」でDBにINSERTすることができない事象が発生した。

当初書いていたソースコード（※実際に書いているコードをかなり省略＆簡略化したものです）

```jsx
let apiData = { data: 
   [{ id: '1',
      first_name: 'Teppei',
      last_name: 'Torii',
      gender: 'Male',
      age: '29'}]
   };

let tableColList = Object.keys(apiData['data'][0]);

let tableColValue = '?'.repeat(tableColList.Length).replace(/(.)(?=.)/g, '?,');

function insertApiData(){
  let db = connectToDatabase();

  let dbStatement = db.prepareStatement('INSERT INTO table(' + tableColList.join(',') + ') values(' + tableColValue + ')');

  for(let i = 0; i < apiData['data'].length; i++){
    dbStatement.setString(1,apiData['data'][i][tableCol[0]]);
    dbStatement.setString(2,apiData['data'][i][tableCol[1]]);
    dbStatement.setString(3,apiData['data'][i][tableCol[2]]);
    dbStatement.setString(4,apiData['data'][i][tableCol[3]]);
    dbStatement.setString(5,apiData['data'][i][tableCol[4]]);
    databaseStatement.addBatch();
  }
  databaseStatement.close();
  database.close();
}
```

こうなるだろうと思っていた

```jsx
let apiData = { data: 
   [{ id: '1',
      first_name: 'Teppei',
      last_name: 'Torii',
      gender: 'Male',
      age: '29'}]
   };

let tableColList = Object.keys(apiData['data'][0]);
// ['id', 'first_name', 'last_name', 'gender', 'age']

let tableColValue = '?'.repeat(tableColList.Length).replace(/(.)(?=.)/g, '?,');

function insertApiData(){
  let db = connectToDatabase();

  let dbStatement = db.prepareStatement('INSERT INTO table(' + tableColList.join(',') + ') values(' + tableColValue + ')');
// 'INSERT INTO table('id, first_name, last_name, gender, age') values('?, ?, ?, ?, ?')'

  for(let i = 0; i < apiData['data'].length; i++){
    dbStatement.setString(1,apiData['data'][i].id); // 1
    dbStatement.setString(2,apiData['data'][i].first_name); // Teppei
    dbStatement.setString(3,apiData['data'][i].last_name); // Torii
    dbStatement.setString(4,apiData['data'][i].gender); // Male
    dbStatement.setString(5,apiData['data'][i].age); // 29
    databaseStatement.addBatch();
  }
  databaseStatement.close();
  database.close();
}
```

だが実際動かしてみると・・・

```jsx
let apiData = { data: 
   [{ id: '1',
      first_name: 'Teppei',
      last_name: 'Torii',
      gender: 'Male',
      age: '29'}]
   };

let tableColList = Object.keys(apiData['data'][0]);
// ['id', 'first_name', 'gender', 'age', 'last_name']

let tableColValue = '?'.repeat(tableColList.Length).replace(/(.)(?=.)/g, '?,');

function insertApiData(){
  let db = connectToDatabase();

  let dbStatement = db.prepareStatement('INSERT INTO table(' + tableColList.join(',') + ') values(' + tableColValue + ')');
// 'INSERT INTO table('id, first_name, gender, age, last_name') values('?, ?, ?, ?, ?')'

  for(let i = 0; i < apiData['data'].length; i++){
    dbStatement.setString(1,apiData['data'][i].id); // 1
    dbStatement.setString(2,apiData['data'][i].first_name); // Teppei
    dbStatement.setString(3,apiData['data'][i].last_name); // Male
    dbStatement.setString(4,apiData['data'][i].gender); // 29
    dbStatement.setString(5,apiData['data'][i].age); // Torii
    databaseStatement.addBatch();
  }
  databaseStatement.close();
  database.close();
}
```

当初想定していた順番とは違う結果になり、おまけにDBにINSERTされる値がlast_nameカラムにgenderの値が入るといった、カオス状態に。

# Mapの挙動について

---

```jsx
const map = new Map();

map.set("a", 1);
map.set("b", 2);
map.set("c", 3);
map.set("b", 4);

console.log([...map.entries()]); // => [["a", 1], ["b", 4], ["c", 3]]

```

キーの順番は挿入順になることが保証されている。

 同一のキーで上書いた場合、キーの順番に関しては変化せず、値だけが書き換わる挙動になる。 （この挙動は仕様で保証されている。）

さらに、コンストラクタからキーと値を一気に流し込んだ場合も、`set`を順番通りに複数回呼んだときと同じ挙動になる。
（同じくこの挙動は仕様で保証されている。）

```jsx
const map = new Map([["a", 1], ["b", 2], ["c", 3], ["b", 4]]);

console.log([...map.entries()]); // => [["a", 1], ["b", 4], ["c", 3]]

```

# オブジェクトのプロパティの列挙順が保証されるケースとされないケース

---

冒頭に記述の通り、保証されるメソッドとされないメソッドがある。

## 列挙順が保証されるメソッド

- `Object.assign`
- `Object.defineProperties`
- `Object.getOwnPropertyNames`
- `Object.getOwnPropertySymbols`
- `Reflect.ownKeys`

## 列挙順が保証されないメソッド

- `Object.keys`
- `for-in`（正確にはメソッドではない）
- `JSON.parse`
- `JSON.stringify`

保証される場合も、直感的とは言い難い順番で列挙される。
したがって、オブジェクトのプロパティの列挙順に依存するコードを書くべきではない。

## 列挙順が保証される場合の列挙順

---

次の順に列挙される。

1. 0~2^32 - 2^1 の整数として解釈可能な文字列であるプロパティを、整数として解釈した場合の昇順で列挙する
2. 0~2^32 - 2 の整数として解釈できない文字列であるプロパティを、挿入順に列挙する
3. シンボルであるプロパティを、挿入順に列挙する

## なぜこんなことになっているのか

---

ECMAScript 5以前は、オブジェクトのプロパティの列挙順は（少なくとも仕様レベルでは）全く保証されていなかった。

ここに動きが出たのがECMAScript 2015である。

ECMAScript 2015では、オブジェクトのプロパティについても列挙順を保証しようということになった。

しかし、ある特定の列挙順を仕様で決めてしまうと、互換性が崩れかねない問題がある。

たとえば、実装依存のある特定の列挙順を前提としているコードが壊れてしまわないだろうか？

ECMAScript、ひいてはWebの技術は、仕様の明快さよりも今あるものが動くことを大事にする哲学を持つ。
そういうことで、ECMAScript 5以前からあるメソッドは、あえて列挙順を保証しないままとすることになった。

オブジェクトのプロパティの列挙順に依存するコードを書くべきではない。
この目的では`Map`を使うべきである。`Map`のキーは挿入順で列挙される。

## 原因と対処法

---

列挙順が保証されないObject.keysを使用していたこと。

```jsx
// 諸悪の根源
let tableColList = Object.keys(apiData['data'][0]);
```

なので、列挙順が保証されるObject.getOwnPropertyNamesに置き換えた。

```jsx
let tableColList = Object.getOwnPropertyNames(apiData['data'][0]);
```

プラスで、APIの仕様変更によるパラメーター名称変更を考慮し、こちらのコードも以下のように修正。

```jsx
function insertApiData(){
  let db = connectToDatabase();

  let dbStatement = db.prepareStatement('INSERT INTO table(' + tableColList.join(',') + ') values(' + tableColValue + ')');

  for(let i = 0; i < apiData['data'].length; i++){
    dbStatement.setString(1,apiData['data'][i][tableColList[0]]); 
    dbStatement.setString(2,apiData['data'][i][tableColList[1]]); 
    dbStatement.setString(3,apiData['data'][i][tableColList[2]]); 
    dbStatement.setString(4,apiData['data'][i][tableColList[3]]); 
    dbStatement.setString(5,apiData['data'][i][tableColList[4]]);
    databaseStatement.addBatch();
  }
  databaseStatement.close();
  database.close();
}
```

## あとがき


一番理想的なのは Map を使って、仕様に従った「オブジェクトのプロパティ列挙順に依存しない」コードを書くことっていうのがすごく腑に落ちた。

転職以来、これまで素の言語でしか開発していないことになんとなく不安を感じていたが、フレームワークだと見落としそうなこういう点に気づけるのは、素の言語での開発ならではかなと。

リリース後のタスクになるが、技術的負債になることを防ぐために Map を使ったコードに改修するのと、今後同様のサービスを展開していくので、それらはちゃんと Map を使ってコードを書くことにする。

## 参考記事

[https://qiita.com/anqooqie/items/ab3fed5d269d278cdd2b](https://qiita.com/anqooqie/items/ab3fed5d269d278cdd2b)