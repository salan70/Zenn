---
title: "FutureProviderを使ってAPIからデータを取得する"
emoji: "🎋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter]
published: true
---

# はじめに
Riverpodの**FutureProvider**を使って**API**からデータを取得する方法について書いていきます。

Flutterを触ってから1年半以上経ちますが、恥ずかしながらFutureProviderをまともに使ったことと、APIからデータを取得したことがなかったため、勉強しながら実装したサンプルアプリのコードをもとに説明していきます。

## 実装したサンプルアプリ
![](/images/use_futureprovider_for_api/2022-07-13-16-47-53.png =250x)

## 全体のコード
https://github.com/salan70/api_practice

# 実装
## 0. 準備・前提
### 使用したAPI
https://reqres.in/

### 使用したパッケージ
- [dio](https://pub.dev/packages/dio)
- [flutter_riverpod](https://pub.dev/packages/flutter_riverpod)

### pubspec.yaml
```yaml
version: 1.0.0+1

environment:
  sdk: ">=2.17.5 <3.0.0"

dependencies:
  flutter:
    sdk: flutter

  cupertino_icons: ^1.0.2
  flutter_riverpod: ^1.0.4
  dio: ^4.0.6
  pedantic_mono: ^1.19.2

  freezed_annotation:

dev_dependencies:
  flutter_test:
    sdk: flutter

  flutter_lints: ^2.0.0

  build_runner:
  freezed:

  json_serializable:

flutter:

  uses-material-design: true
```

## 1. APIからデータを取得
以下のような **「fetchParsonDataList()」** という関数を作成しました。

```dart:person_ripository.dart
  Future<List<Person>> fetchParsonDataList() async {
    List<dynamic> responseDataList;
    final personDataList = <Person>[];

    // 利用するAPIのURL
    const url = 'https://reqres.in/api/users?page=2';
    final response = await Dio().get<Map<dynamic, dynamic>>(url);

    if (response.statusCode == 200) {
      // null check
      if (response.data != null) {
        // responseのdataを一旦、dynamic型のListとして格納
        responseDataList = response.data!['data'] as List<dynamic>;

        for (final responseData in responseDataList) {
          // Listの要素を1つ1つPerson(自作の型)に変換し、返すListに追加していく
          personDataList
              .add(Person.fromJson(responseData as Map<String, dynamic>));
        }
      } else {
        throw Exception('Data is not exist');
      }
    } else {
      throw Exception('Failed to load sentence');
    }

    return personDataList;
  }
```
API（LIST USERSを使用）の構成は以下のようになっており、
今回は **"data"** の値を取得しています。

```json
{
    "page": 2,
    "per_page": 6,
    "total": 12,
    "total_pages": 2,
    "data": [
        {
            "id": 7,
            "email": "michael.lawson@reqres.in",
            "first_name": "Michael",
            "last_name": "Lawson",
            "avatar": "https://reqres.in/img/faces/7-image.jpg"
        },
        {
            "id": 8,
            "email": "lindsay.ferguson@reqres.in",
            "first_name": "Lindsay",
            "last_name": "Ferguson",
            "avatar": "https://reqres.in/img/faces/8-image.jpg"
        },
        .
        .
        .
    ],
    "support": {
        "url": "https://reqres.in/#support-heading",
        "text": "To keep ReqRes free, contributions towards server costs are appreciated!"
    }
}
```

なお、格納する際に使用したPersonは以下のようになっています。
※freezedとjson_serializableを使用しました。

```dart:person_model.dart
@freezed
class Person with _$Person {
  const factory Person({
    required int id,
    required String email,
    required String first_name,
    required String last_name,
    required String avatar,
  }) = _Person;

  factory Person.fromJson(Map<String, dynamic> json) => _$PersonFromJson(json);
}
```

前述した **「fetchParsonDataList()」** 関数内では、APIから取得した値をPersonに変換するために、json_serializableを使用して生成された以下の**fromJson**を使っています。


```dart:person_model.g.dart
_$_Person _$$_PersonFromJson(Map<String, dynamic> json) => _$_Person(
      id: json['id'] as int,
      email: json['email'] as String,
      first_name: json['first_name'] as String,
      last_name: json['last_name'] as String,
      avatar: json['avatar'] as String,
    );
```

## 2. FutureProviderを作成
最初に作成した **「fetchParsonDataList()」** で返ってくるListを返すFutureProviderを作成しました。

```dart: provider.dart
final personDataListFutureProvider =
    FutureProvider.autoDispose<List<Person>>((ref) async {
  final personRepository = PersonRepository();
  final personDataList = await personRepository.fetchParsonDataList();
  return personDataList;
});
```

## 3. Viewを作成
先程作成した **personDataListFutureProvider** を宣言し、
 **.when()** を使用してデータを表示するようにしました。

```dart: person_list_page.dart
class PersonListPage extends ConsumerWidget {
  const PersonListPage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final personDataList = ref.watch(personDataListFutureProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Person'),
      ),
      body: ColoredBox(
        color: Colors.black12,
        child: personDataList.when(
          data: (personDataList) {
            return ListView.builder(
              itemCount: personDataList.length,
              itemBuilder: (BuildContext context, int index) {
                final personData = personDataList[index];
                return Card(
                  child: ListTile(
                    leading: CircleAvatar(
                      backgroundImage: NetworkImage(
                        personData.avatar,
                      ),
                    ),
                    title: Text(
                      '${personData.first_name} ${personData.last_name}',
                    ),
                    subtitle: Text(
                      personData.email,
                    ),
                    trailing: Text('id: ${personData.id}'),
                  ),
                );
              },
            );
          },
          error: (error, stack) => Text('Error: $error'),
          loading: () => const CircularProgressIndicator(),
        ),
      ),
    );
  }
}
```

# おまけ
ここまでで、FutureProviderを使ってAPIのデータを取得することは完了しました。

が、これだけでは物足りないと思い、first_nameの昇順ソートしてから表示するようにしました。
（もともとはidの昇順で表示されていました）
※冒頭に載せたサンプルアプリのスクショではfirst_nameの昇順でソートされています。

ここでは、おまけとしてそのコードを記載しますのでよろしければご参考ください！

## おまけの実装
まずは、引数として取得したPerson型のListをfirst_nameの昇順でソートして返す関数を作成しました。

```dart:person_list_view_model.dart
class PersonListViewModel {
  List<Person> sortFirstName(List<Person> personDataList) {
    final result = personDataList
      ..sort((a, b) => a.first_name.compareTo(b.first_name));

    return result;
  }
}
```

そして、ViewのWidget内で上記の関数を呼び出し、表示するListを変更しました。
※もともとは、 **personDataListFutureProvider** で返ってくるListをそのまま表示していましたが、このListを **sortFirstName()** 関数に引数として渡し、返ってくるListを表示するようにしました。

```diff dart:person_list_page.dart
class PersonListPage extends ConsumerWidget {
  const PersonListPage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final personDataList = ref.watch(parsonDataListFutureProvider);
+    final personListViewModel = PersonListViewModel();

    return Scaffold(
      appBar: AppBar(
        title: const Text('Person'),
      ),
      body: ColoredBox(
        color: Colors.black12,
        child: personDataList.when(
          data: (personDataList) {
+            final sortedPersonDataList =
+                personListViewModel.sortFirstName(personDataList);

            return ListView.builder(
+              itemCount: sortedPersonDataList.length,
-              itemCount: personDataList.length,
              itemBuilder: (BuildContext context, int index) {
+                final personData = sortedPersonDataList[index];
-                final personData = personDataList[index];
                return Card(
                  child: ListTile(
                    leading: CircleAvatar(
                      backgroundImage: NetworkImage(
                        personData.avatar,
                      ),
                    ),
                    title: Text(
                      '${personData.first_name} ${personData.last_name}',
                    ),
                    subtitle: Text(
                      personData.email,
                    ),
                    trailing: Text('id: ${personData.id}'),
                  ),
                );
              },
            );
          },
          error: (error, stack) => Text('Error: $error'),
          loading: () => const CircularProgressIndicator(),
        ),
      ),
    );
  }
}
```

# 参考
今回勉強するにあたり参考にさせていただいた記事を紹介いたします。

https://qiita.com/yasutaka_ono/items/6d2a0d3b0856598f9788
https://zenn.dev/jojojo/articles/2e85c8a885e85e


# おわりに
今回に限った話ではありませんが、情報を発信してくださっている方々には本当に感謝しております。
ありがとうございます！

また、もしこの記事で至らない部分がありましたらご指摘いただけるととても嬉しいです。

ご覧いただきありがとうございました！
