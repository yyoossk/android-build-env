# android-build-env

Android アプリ（APK）を GitHub Actions でビルドするための共通ビルド環境です。
他のリポジトリのワークフローから呼び出して使います。

JDK・Android SDK・（必要なら）NDK / CMake・Gradle を自動で用意し、
ビルドした APK を Artifact としてアップロードします。

## 使い方

ビルドしたいリポジトリに `.github/workflows/build.yml` を作り、次のように書きます。

```yaml
name: Build APK
on:
  push:
  workflow_dispatch:

jobs:
  apk:
    uses: yyoossk/android-build-env/.github/workflows/android-build.yml@main
```

NDK（C/C++ ネイティブコード）を使うプロジェクトの例:

```yaml
jobs:
  apk:
    uses: yyoossk/android-build-env/.github/workflows/android-build.yml@main
    with:
      ndk-version: "26.3.11579264"
      cmake-version: "3.22.1"
      gradle-task: assembleDebug
      artifact-name: my-app-debug
```

ビルド後、Actions の実行結果ページの **Artifacts** から APK をダウンロードできます。

## 入力（with:）

| 名前 | 既定値 | 内容 |
|---|---|---|
| `project-dir` | `.` | Gradle プロジェクトのディレクトリ |
| `gradle-task` | `assembleDebug` | 実行する Gradle タスク |
| `java-version` | `17` | JDK のバージョン |
| `ndk-version` | （なし） | NDK のバージョン。空なら入れない |
| `cmake-version` | （なし） | CMake のバージョン。空なら入れない |
| `gradle-version` | `8.7` | `gradlew` が無いときに使う Gradle |
| `submodules` | `recursive` | サブモジュールの取得方法 |
| `apk-path` | `**/build/outputs/apk/**/*.apk` | 回収する APK |
| `artifact-name` | `apk` | Artifact の名前 |

## リリース署名（任意）

呼び出し側リポジトリの Secrets に次を登録し、`secrets: inherit` を付けます。

- `KEYSTORE_BASE64`（`base64 -w0 release.keystore` の出力）
- `KEYSTORE_PASSWORD` / `KEY_ALIAS` / `KEY_PASSWORD`

```yaml
jobs:
  apk:
    uses: yyoossk/android-build-env/.github/workflows/android-build.yml@main
    with:
      gradle-task: assembleRelease
    secrets: inherit
```

ビルド時には環境変数 `SIGNING_STORE_FILE` / `SIGNING_STORE_PASSWORD` /
`SIGNING_KEY_ALIAS` / `SIGNING_KEY_PASSWORD` が設定されるので、
`build.gradle` の `signingConfigs` から `System.getenv(...)` で読み込んでください。

## 注意

- このリポジトリが **非公開** の場合、Settings → Actions → General →
  Access で「Accessible from repositories owned by the user」を有効にしてください。
  公開リポジトリなら設定は不要です。

## 動作確認

`sample/` に最小の Android アプリがあり、`main` に push するたびに
`.github/workflows/self-test.yml` がこの共通環境でビルドします。
Actions の **Self test** が緑なら、ビルド環境は正常に動いています。
