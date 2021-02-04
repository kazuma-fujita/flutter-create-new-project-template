# Android StudioのFlutterテンプレートをカスタマイズする方法

Android StudioのFlutterテンプレートをカスタマイズしてプロジェクト作成時のLint静的解析/Visual Debugging設定/画像設定を自動化する方法です。

Flutter テンプレートは Flutter SDK をインストールしたディレクトリ配下にあります。

- {flutter-sdk-path}/flutter/packages/flutter_tools/templates/app/

### 環境

- macOS Big Sur 11.1
- Android Studio 4.1.2
- Flutter 1.22.6
- Dart 2.10.5

## レイアウト構成を見ながら作業ができる Visual Debugging 設定

レイアウト構成を確認しながら作業をしたい場合は、`rendering.dart` package を import して `debugPaintSizeEnabled` を有効にします。

`debugPaintSizeEnabled` を有効にすると margin や padding、リストの向きなどが可視化され Visual Debugging することができます。

設定をプロジェクト新規作成時のテンプレートに反映するにはアプリの Run 時に必ず実行される `main` メソッドがある `main.dart.tmpl` に設定を追記します。

`main.dart.tmpl` の場所は以下になります。

- {flutter-sdk-path}/flutter/packages/flutter_tools/templates/app/lib/main.dart.tmpl

ファイルを開いてコメントの `add` のコードを追記します。

```dart
import 'package:flutter/material.dart';
import 'package:flutter/rendering.dart'; // add
{{#withDriverTest}}
import 'package:flutter_driver/driver_extension.dart';
{{/withDriverTest}}
{{#withPluginHook}}
import 'dart:async';

import 'package:flutter/services.dart';
import 'package:{{pluginProjectName}}/{{pluginProjectName}}.dart';
{{/withPluginHook}}

void main() {
{{#withDriverTest}}
  // Enable integration testing with the Flutter Driver extension.
  // See https://flutter.dev/testing/ for more info.
  enableFlutterDriverExtension();
{{/withDriverTest}}
  debugPaintSizeEnabled = true; // Add
  runApp(MyApp());
}
```

## Lint 静的解析設定

静的解析をする為の Lint 設定をするにはプロジェクトルート(`pubspec.yaml`がある階層)に `analysis_options.yaml` というファイルを作成して Lint ルールを記述します。

`analysis_options.yaml` のテンプレートとして以下の階層に `analysis_options.yaml.tmpl` を作成します。

- {flutter-sdk-path}/flutter/packages/flutter_tools/templates/app/analysis_options.yaml.tmpl

`analysis_options.yaml.tmpl` に以下 1 行を追記します。

```yml:analysis_options.yaml
include: package:pedantic_mono/analysis_options.yaml
```

筆者はまだ個別の細かい Lint ルールを把握していないので、ここでは推奨設定がまとまっている `pedantic_mono` package を利用しています。

個別設定する場合の Lint ルールは [Linter for Dart - Supported Lint Rules](https://dart-lang.github.io/linter/lints/) に記載されています。

次に以下に階層にある `pubspec.yaml` のテンプレート `pubspec.yaml.tmpl` を開きます。

- {flutter-sdk-path}/flutter/packages/flutter_tools/templates/app/pubspec.yaml.tmpl

`pubspec.yaml.tmpl` の `dev_dependencies` に `pedantic_mono: any` を追記します。

```yml:pubspec.yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  pedantic_mono: any # add
```

## ローカル画像を扱えるようにする設定

大なり小なりアプリ開発では画像を扱うことになります。

assets の設定をしてローカルの画像ファイルを扱えるようにしましょう。

まず Flutter テンプレートディレクトリに `assets.tmpl/images` ディレクトリを作成します。

```txt
mkdir -p {flutter-sdk-path}/flutter/packages/flutter_tools/templates/app/assets.tmpl/images
```

`assets.tmpl` というディレクトリはテンプレートからプロジェクト作成時に `assets` というディレクトリ名にリネームされます。

次になんでもいいので画像ファイルを `assets.tmpl/images` 以下に配置します。

```txt
cp sample_image.jpeg {flutter-sdk-path}/flutter/packages/flutter_tools/templates/app/assets.tmpl/images/sample_image.jpeg
```

最後に以下に階層にある `pubspec.yaml` のテンプレート `pubspec.yaml.tmpl` を編集します。

- {flutter-sdk-path}/flutter/packages/flutter_tools/templates/app/pubspec.yaml.tmpl

`pubspec.yaml.tmpl` に 以下を追記してください。

```yaml:pubspec.yaml
  assets:
    -  assets/images/
```

上記設定で　`assets/images` 配下の全てのローカル画像ファイルを扱うことができます。

プロジェクト作成後は以下コードでローカル画像を呼び出すことができます。

```dart
Image.asset('assets/images/sample_image.jpeg')
```

## template_manifest.json にプロジェクト作成時にコピーするファイルを追記する

以下の場所に `template_manifest.json` があるので、新規で追加したテンプレートファイルを追記していきます。

- {flutter-sdk-path}/flutter/packages/flutter_tools/templates/template_manifest.json

今回 `analysis_options.yaml.tmpl` を新規で作成しているのでファイルパスを追記します。

また `assets.tmpl` ディレクトリを追加しているのでそちらも追記します。

```
{
    "version": 1.0,
    "_comment": "A listing of all possible template output files.",
    "files": [
        "templates/app/analysis_options.yaml.tmpl",
        "templates/app/assets.tmpl/images/sample_image.jpeg",
```

`template_manifest.json` にテンプレートファイルを追記するとプロジェクト新規作成にファイルをコピーしてくれます。

以上でプロジェクト新規作成時の Flutter テンプレート作成を完了です。

## プロジェクトを新規作成する

それでは Android Studio を開いて、 `Create New Flutter Project` > `Flutter Application` から新規プロジェクトを作成しましょう。

プロジェクトに今回新規で追加したテンプレートの `analysis_options.yaml` `assets/images/sample_image.jpeg` が作成されています。

また、`main.dart` には Visual Debugging の設定が追記されています。

`pubspec.yaml` には今回追加した package `pedantic_mono` と `assets` 設定が追記されています。

## 終わりに

他にも有用な設定は Flutter テンプレートをカスタマイズすれば自由に設定の自動化が可能です。

筆者はまだ Flutter 歴が浅いのでもっと有用な設定が他にもあると思います。

もし他にもこんな便利な設定があるよ、という方はぜひ [Twitter](https://twitter.com/____ZUMA____) で DM していただくか [Contact](/contact) で連絡お願いします。

