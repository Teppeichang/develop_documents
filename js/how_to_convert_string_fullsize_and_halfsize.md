# 英数を全角/半角に変換する方法

## 英数字を半角にする場合

全角英数の文字コードから65248個前が半角英数の文字コードとなっている為、
文字コードから65248引いて変換。

```jsx
//10進数の場合
str.replace(/[Ａ-Ｚａ-ｚ０-９]/g, function(s) {
    return String.fromCharCode(s.charCodeAt(0) - 65248);
});

//16進数の場合
str.replace(/[Ａ-Ｚａ-ｚ０-９]/g, function(s) {
    return String.fromCharCode(s.charCodeAt(0) - 0xFEE0);
});

```

## 英数字を全角にする場合

全角にする場合は
文字コードから65248足して変換。

```jsx
//10進数の場合
str.replace(/[A-Za-z0-9]/g, function(s) {
    return String.fromCharCode(s.charCodeAt(0) + 65248);
});

//16進数の場合
str.replace(/[A-Za-z0-9]/g, function(s) {
    return String.fromCharCode(s.charCodeAt(0) + 0xFEE0);
});

```

## 備考

英字だけ、数字だけの場合は、replace内部[A-Za-z0-9]から必要のない部分を取り除く