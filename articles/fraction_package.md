---
title: "【Flutter】fractionパッケージを使って分数と帯分数を簡単に扱う"
emoji: "💔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter]
published: false
---
# はじめに
fractionパッケージを使うことで、分数と帯分数を簡単に扱うことができます。  

例えば、整数や小数を分数や帯分数に変換したり、分数同士で計算したり、少数+分数の計算したりといったことができます。

# fractionパッケージの使い方
fractionパッケージでは、Fraction型は分数とMixedFraction型が用意されています。  

:::message
Fraction型は分数、MixedFractin型は帯分数を表しています。
:::

## 変換
### fromString()
fromString()を使うことで、String型から変換することができます。  

:::message alert
MixedFractionへ変換する際は'a b/c'という形式でないとエラーになります。
:::

```dart
// Fraction
Fraction.fromString('1/2'); // 1/2
Fraction.fromString('-1/3'); // -1/3
Fraction.fromString('-2'); // -2
Fraction.fromString('2/-5'); // Error
Fraction.fromString('/3'); // Error

// MixedFraction
MixedFraction.fromString('1 1/2'); // 1 1/2
MixedFraction.fromString('-1 1/3'); // -1 1/3
MixedFraction.fromString('1 2/-5'); // Error
MixedFraction.fromString('-2'); // Error
MixedFraction.fromString('1/3'); // Error
```

### fromDouble()
fromDouble()を使うことで、Double型から変換することができます。

:::message alert  
1や3.0のように、整数をMixedFractionに変換する場合、「a 0/1」という形式になります。
:::

```dart
// Fraction
Fraction.fromDouble(1.5); // 3/2
Fraction.fromDouble(-1.5); // -3/2
Fraction.fromDouble(1.0); // 1
Fraction.fromDouble(3); // 3

// MixedFraction
MixedFraction.fromDouble(1.5); // 1 1/2
MixedFraction.fromDouble(-1.5); // -1 2/5
MixedFraction.fromDouble(1.0); // 1 0/1
MixedFraction.fromDouble(3); // 3 0/1
```

### toFraction(), toMixedFraction()
String型やDouble型に対して、toFraction(), toMixedFraction()を使うことで、Fraction, MixedFractionへ変換することができます。

```dart
// Fraction
'2/5'.toFraction(); // 2/5
'1'.toFraction(); // 2/5
1.5.toFraction(); // 3/2
1.toFraction(); // 1
3.0.toFraction(); // 3

// MixedFraction
'1 2/5'.toMixedFraction(); // 1 2/5
1.5.toMixedFraction(); // 1 1/2
1.toMixedFraction(); // 1
3.0.toMixedFraction(); // 3
```

また、Fraction型, MixedFraction型に対してtoFraction(), toMixedFraction()を使うこともできます。

```dart
// Fraction.toMixedFraction()
'2/5'.toFraction().toMixedFraction(); // 2/5
'1'.toFraction().toMixedFraction(); // 1 0/1
1.5.toFraction().toMixedFraction(); // 1 1/2
1.toFraction().toMixedFraction(); // 1 0/1
3.0.toFraction().toMixedFraction(); // 3 0/1

// MixedFraction.toFraction()
'1 2/5'.toMixedFraction().toFraction(); // 7/5
1.5.toMixedFraction().toFraction(); // 3/2
1.toMixedFraction().toFraction(); // 1
3.0.toMixedFraction().toFraction(); // 3
```

### Rational.tryParse()
Rational.tryParse()を使うことで、Fraction型かMixedFraction型のどちらかを解析し変換することができます。  

もし、Fraction型にもMixedFraction型にも変換できない際にはnullを返してくれます。

:::message 
Rational.tryParse()の引数はString型である必要があります。
:::

```dart
Rational.tryParse('2/5'); // 2/5 (Fraction)
Rational.tryParse('3/2'); // 3/2 (Fraction)
Rational.tryParse('2'); // 2 (Fraction)
Rational.tryParse('1 1/2'); // 1 1/2 (MixedFraction)
Rational.tryParse(''); // null
Rational.tryParse('壱'); // null
```

### reduce()
reduce()を使うことで、Fraction, MixedFractionを約分することができます。

```dart
'6/10'.toFraction().reduce(); // 3/5
'1 4/10'.toMixedFraction().reduce(); // 1 2/5
```

## 計算
Fraction同士もしくは、MixedFraction同士であれば四則演算を行うことができます。  

:::message alert  
FractionとMixedFractionでの四則演算はできないため、toFraction(), toMixedFraction()を使うなどして、両方の型を揃える必要があります。
:::

以下は足し算の例です。

```dart
final a = '1/4'.toFraction();
final b = '2/4'.toFraction();
final c = '1 2/4'.toMixedFraction();

final sumA = a + b; // 12/16
final reducedSumA = sum.reduce(); // 3/4

final sumB = a.toMixedFraction() + c; // 1 12/16
final reducedSumB = sumB.reduce(); // 1 3/4
```

:::message  
上記のように、「1/4 + 1/4」 を行うと「12/16」という結果になりますが、  
計算結果に対して.reduce()関数を使うことで約分できます。
:::

# 参考
https://pub.dev/packages/fraction
https://matsumarudesu.com/flutter-dart-fraction/
https://github.com/albertodev01/fraction