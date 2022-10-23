---
title: "FlutterでWebスクレイピングしてみた"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, Dart]
published: false
---
# はじめに
WebスクレイピングといえばPythonのイメージがあったのですが、  
ふと、FlutterでもWebスクレイピングできるんじゃないかと思い、色々調べて試してみました。

単純ではありますが、次のようなWebスクレイピングをFlutterで実装してみたので、その方法をご紹介します。

- Wikipediaから、現時点で広島東洋カープに所属している捕手の名前と出身地を取得する

:::message alert
本記記事では、Webスクレイピングの技法自体については深く触れません。
:::

# 完成イメージ
| アプリ起動時 | 画面右下のテキストボタン押下時 | 
| ---- | ---- | 
|![](/images/flutter_scraping/before_scraping.png) |![](/images/flutter_scraping/after_scraping.png) |!

# 使用したパッケージ
universal_htmlというパッケージを使用しました。
https://pub.dev/packages/universal_html

# 実装方法
## 実装内容を整理
まずは実装内容を細分化してみます。

今回実装するのは、次のようなWebスクレイピングでした。
- Wikipediaから、現時点で広島東洋カープに所属している捕手の名前と出身地を取得する

上記を次のような手順で実装していこうと思います。

1. Wikipediaの[広島東洋カープの選手一覧](https://ja.wikipedia.org/wiki/%E5%BA%83%E5%B3%B6%E6%9D%B1%E6%B4%8B%E3%82%AB%E3%83%BC%E3%83%97%E3%81%AE%E9%81%B8%E6%89%8B%E4%B8%80%E8%A6%A7)にアクセスし、現時点で広島東洋カープに所属している捕手の名前と、選手ごとのページのURLを取得する。
※選手ごとのサイトの例：[會澤翼](https://ja.wikipedia.org/wiki/%E6%9C%83%E6%BE%A4%E7%BF%BC)

2. 選手ごとのサイトにアクセスし、出身地（都道府県のみ）を取得する
3. 1と2で取得した選手名と出身地をListViewで表示する

以降は、上記手順ごとの実装方法を説明します。

## 1. Wikipediaの広島東洋カープの選手一覧にアクセスし、現時点で広島東洋カープに所属している捕手の名前と、選手ごとのサイトのURLを取得する。

はじめに、WindowControllerのopenHttp関数を使ってhtmlを取得します。
```dart
final url =
'https://ja.wikipedia.org/wiki/%E5%BA%83%E5%B3%B6%E6%9D%B1%E6%B4%8B%E3%82%AB%E3%83%BC%E3%83%97%E3%81%AE%E9%81%B8%E6%89%8B%E4%B8%80%E8%A6%A7';
final controller = WindowController();
await controller.openHttp(uri: Uri.parse(url));
```

次に、querySelectorAll関数を使って、取得したい要素を指定します。
querySelectorAll関数は、CSSセレクターを使用して要素を絞り込むことができます。
```dart
const selector =
    '#mw-content-text > div.mw-parser-output > table:nth-child(17) > tbody > tr';
final elementList = controller.window!.document.querySelectorAll(selector);
```

:::message
CSSセレクターについては、下記のサイトがとても参考になります。
他にも、多くの記事がありますので是非調べてみてください。
https://qiita.com/Azunyan1111/items/b161b998790b1db2ff7a
:::

次に、for文を使ってelementListの要素を1つずつ見ていきます。
各要素をさらにquerySelector関数で絞り、選手名と選手ごとのページのURLを取得します。
```dart
for (final element in elementList) {
    final catcher = element.querySelector('> td > a');
    if (catcher != null) {
    final name = catcher.title!;
    final url = catcher.attributes['href']!;
    }
}
```

## 2. 選手ごとのページにアクセスし、出身地（都道府県のみ）を取得する
手順1と同じように、まずはWindowControllerのopenHttp関数を使ってhtmlを取得します。
```dart
final controller = WindowController();
await controller.openHttp(uri: Uri.parse('https://ja.wikipedia.org$url'));
```

次に、こちらも手順1と同様にquerySelectorAll関数やquerySelector関数を使って、取得したい要素を指定します。
```dart
const selector =
    '#mw-content-text > div.mw-parser-output > table.infobox > tbody > tr:nth-child(5) > td';
final elementList = controller.window!.document.querySelectorAll(selector);

for (final element in elementList) {
    // 出身地（都道府県）をreturn
    return element.querySelector(' > a')!.title!;
}
```

ここまで実装したコードの全文は下記です。  

手順1で実装したものをfetchCatcherという関数にしました。  
また、手順2で実装したものをfetchHometownという関数にしました。  

fetchCatcher関数の中で、選手ごとにfetchHometown関数を実行するようになっています。
```dart
// Catcher型を定義
class Catcher {
  final String name;
  final String hometown;

  Catcher(this.name, this.hometown);
}

// スクレイピングを実装
class Scraping {

  // 手順1
  Future<List<Catcher>> fetchCatcher(String url) async {
    final controller = WindowController();
    await controller.openHttp(uri: Uri.parse(url));

    const selector =
        '#mw-content-text > div.mw-parser-output > table:nth-child(17) > tbody > tr';
    final elementList = controller.window!.document.querySelectorAll(selector);

    final List<Catcher> catcherList = [];

    for (final element in elementList) {
      final catcher = element.querySelector('> td > a');
      if (catcher != null) {
        final name = catcher.title!;

        final url = catcher.attributes['href']!;
        final hometown = await fetchHometown(url);

        catcherList.add(Catcher(name, hometown));
        await Future.delayed(const Duration(seconds: 5));
      }
    }

    return catcherList;
  }

  // 手順2
  Future<String> fetchHometown(String url) async {
    final controller = WindowController();
    await controller.openHttp(uri: Uri.parse('https://ja.wikipedia.org$url'));

    const selector =
        '#mw-content-text > div.mw-parser-output > table.infobox > tbody > tr:nth-child(5) > td';
    final elementList = controller.window!.document.querySelectorAll(selector);

    for (final element in elementList) {
      return element.querySelector(' > a')!.title!;
    }

    return '';
  }
}
```

:::message alert
上記関数は、あくまで広島の捕手の選手ページ(Wikipedia)に対応したものです。
他の球団の選手などのページのurlを引数として渡すと、出身地が帰ってこない場合もありますのでご了承ください。
:::

## 1と2で取得した選手名と出身地をListViewで表示する
手順1と2で選手名と出身地を取得することができますので、あとは取得したデータをListViewで表示します。  

今回はStatefulWidgetを使って、画面内のテキストボタンを押すことで手順1と2のスクレイピングを実行し、データを取得してListViewで表示するようにしました。

```dart
class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key, required this.title}) : super(key: key);

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  final _scraping = Scraping();
  final _url =
      'https://ja.wikipedia.org/wiki/%E5%BA%83%E5%B3%B6%E6%9D%B1%E6%B4%8B%E3%82%AB%E3%83%BC%E3%83%97%E3%81%AE%E9%81%B8%E6%89%8B%E4%B8%80%E8%A6%A7';

  List<Catcher> _catcherList = [];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: ListView.builder(
        itemCount: _catcherList.length,
        itemBuilder: (BuildContext context, int index) {
          return Container(
            padding: const EdgeInsets.all(8),
            alignment: Alignment.centerLeft,
            height: 64,
            decoration: const BoxDecoration(
              border: Border(
                  bottom: BorderSide(
                color: Colors.black12,
              )),
            ),
            child: Column(
              children: [
                Text(_catcherList[index].name),
                Text(_catcherList[index].hometown),
              ],
            ),
          );
        },
      ),
      floatingActionButton: TextButton(
        onPressed: () async {
          final tmpCatcherList = await _scraping.fetchCatcher(_url);
          setState(() {
            _catcherList = tmpCatcherList;
          });
        },
        child: const Text(
          '広島のキャッチャーをスクレイピング',
          style: TextStyle(
            color: Colors.red,
          ),
        ),
      ), 
    );
  }
}
```

# 参考
https://pub.dev/packages/universal_html
