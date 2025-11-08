# レッスン管理システム セットアップガイド

## 📋 前提条件

- Googleアカウント
- Google Cloud Platform (GCP) プロジェクト
- Googleスプレッドシート（3つ）

## 🚀 セットアップ手順

### ステップ1: Google Apps Script プロジェクトの作成

1. [Google Apps Script](https://script.google.com/) にアクセス
2. 「新しいプロジェクト」をクリック
3. プロジェクト名を「レッスン管理システム」に変更

### ステップ2: ファイルのアップロード

以下のファイルをGASエディタにコピーします（`.txt` 拡張子を削除）：

1. **Settings.gs** ← `Settings.gs.txt`
2. **Main.gs** ← `Main.gs.txt`
3. **SpreadsheetService.gs** ← `SpreadsheetService.gs.txt`
4. **GeminiService.gs** ← `GeminiService.gs.txt`
5. **SlideService.gs** ← `SlideService.gs.txt`
6. **DriveService.gs** ← `DriveService.gs.txt` (空でOK)
7. **index.html** ← `index.html.txt`
8. **Header.html** ← `Header.html.txt`
9. **Page-StrategyInput.html** ← `Page-StrategyInput.html.txt`
10. **Page-StrategyView.html** ← `Page-StrategyView.html.txt`
11. **Page-GoalManagement.html** ← `Page-GoalManagement.html.txt` (既存ならそのまま)

### ステップ3: appsscript.json の設定

1. GASエディタで「プロジェクトの設定」→「appsscript.json」マニフェストファイルをエディタで表示する」をON
2. `appsscript.json` を開いて、以下の内容に置き換え：

```json
{
  "timeZone": "Asia/Tokyo",
  "dependencies": {
    "enabledAdvancedServices": [
      {
        "userSymbol": "Slides",
        "version": "v1",
        "serviceId": "slides"
      }
    ]
  },
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",
  "webapp": {
    "executeAs": "USER_ACCESSING",
    "access": "ANYONE"
  },
  "oauthScopes": [
    "https://www.googleapis.com/auth/spreadsheets",
    "https://www.googleapis.com/auth/presentations",
    "https://www.googleapis.com/auth/script.container.ui",
    "https://www.googleapis.com/auth/drive",
    "https://www.googleapis.com/auth/script.external_request",
    "https://www.googleapis.com/auth/cloud-platform"
  ]
}
```

### ステップ4: GCPプロジェクトの設定

#### 4-1. GCPプロジェクトの作成（まだない場合）

1. [Google Cloud Console](https://console.cloud.google.com/) にアクセス
2. プロジェクト番号をメモ（例: `123456789012`）

#### 4-2. 必要なAPIを有効化

以下のAPIを有効化してください：

1. **Vertex AI API**
   - [Vertex AI API](https://console.cloud.google.com/apis/library/aiplatform.googleapis.com)

2. **Google Slides API**
   - [Google Slides API](https://console.cloud.google.com/apis/library/slides.googleapis.com)

3. **Google Drive API**
   - [Google Drive API](https://console.cloud.google.com/apis/library/drive.googleapis.com)

4. **Google Sheets API**
   - [Google Sheets API](https://console.cloud.google.com/apis/library/sheets.googleapis.com)

#### 4-3. GASプロジェクトとGCPプロジェクトを紐付け

1. GASエディタで「プロジェクトの設定」
2. 「Google Cloud Platform (GCP) プロジェクト」セクション
3. 「プロジェクトを変更」をクリック
4. GCPプロジェクト番号を入力

### ステップ5: スプレッドシートの準備

#### 5-1. 全生徒一覧スプレッドシート

1. 新しいスプレッドシートを作成
2. シート名を「**全生徒一覧**」に変更
3. ヘッダー行を以下のように設定：

| ID | 名前 | 連絡先 |
|----|------|--------|
| 001 | 山田太郎 | yamada@example.com |
| 002 | 佐藤花子 | sato@example.com |

4. スプレッドシートIDをコピー（URLの`/d/`と`/edit`の間の文字列）

#### 5-2. レッスンふりかえりスプレッドシート

1. 新しいスプレッドシートを作成
2. シート名を「**レッスンふりかえり**」に変更
3. ヘッダー行を以下のように設定：

| 日付 | 生徒ID | 作戦 | 種別 | 振り返り | 次回やること |
|------|--------|------|------|----------|--------------|
|      |        |      |      |          |              |

4. スプレッドシートIDをコピー

#### 5-3. 目標管理スプレッドシート

1. 新しいスプレッドシートを作成
2. シート名を「**目標一覧**」に変更
3. ヘッダー行を以下のように設定：

| 生徒ID | 目標タイプ | 内容 |
|--------|------------|------|
| 001    | 長期       | プログラミングの基礎を習得する |
| 001    | 中期       | Pythonで簡単なゲームを作る |
| 001    | 短期       | 変数と条件分岐を理解する |

4. スプレッドシートIDをコピー

### ステップ6: スクリプトプロパティの設定

1. GASエディタで「プロジェクトの設定」→「スクリプト プロパティ」
2. 以下のプロパティを追加：

| プロパティ | 値 | 説明 |
|------------|----|----|
| `GCP_PROJECT_ID` | `your-project-id` | GCPプロジェクトID |
| `SS_ID_STUDENT_LIST` | `1ABC...` | 全生徒一覧スプレッドシートID |
| `SS_ID_FEEDBACK` | `1DEF...` | レッスンふりかえりスプレッドシートID |
| `SS_ID_GOAL_MANAGEMENT` | `1GHI...` | 目標管理スプレッドシートID |
| `SLIDES_FOLDER_ID` | `1JKL...` | （オプション）スライド保存先フォルダID |

**注意**: GCPプロジェクトIDは、プロジェクト番号ではなく、プロジェクトIDです（例: `my-project-12345`）

### ステップ7: デプロイ

1. GASエディタで「デプロイ」→「新しいデプロイ」
2. 種類：「ウェブアプリ」を選択
3. 設定：
   - 説明: `初回デプロイ`
   - 次のユーザーとして実行: `自分`
   - アクセスできるユーザー: `全員`
4. 「デプロイ」をクリック
5. 承認画面で権限を許可
6. デプロイされたURLをコピー

### ステップ8: 動作確認

1. デプロイされたURLにアクセス
2. ナビゲーションバーが表示されることを確認
3. 「作戦入力」ページで生徒を選択してテスト入力
4. 「作戦表示」ページで保存した内容が表示されることを確認

## 🔧 トラブルシューティング

### エラー: "スクリプト プロパティ 'XXX' が設定されていません"

→ ステップ6のスクリプトプロパティが正しく設定されているか確認してください。

### エラー: "Gemini API呼び出しエラー"

→ GCPプロジェクトでVertex AI APIが有効化されているか確認してください。

### エラー: "スライド作成エラー"

→ Google Slides APIが有効化されているか、appsscript.jsonの設定が正しいか確認してください。

### スプレッドシートにデータが保存されない

→ スプレッドシートIDとシート名が正しいか確認してください。

## 📝 次のステップ

セットアップが完了したら：

1. 実際の生徒データを「全生徒一覧」に入力
2. 各生徒の目標を「目標一覧」に入力
3. レッスン当日に「作戦入力」で作戦を入力
4. レッスン後に「作戦表示」で振り返りを入力してスライド生成

## 💡 ヒント

- **ローカルストレージ**: 作戦入力ページでは、入力内容が自動保存されるため、リロードしても内容が保持されます
- **Toast通知**: スライド生成が完了するとToast通知が表示されます
- **エラーログ**: 問題が発生した場合は、GASエディタの「実行ログ」を確認してください

## 🔒 セキュリティに関する注意

- スクリプトプロパティには機密情報（スプレッドシートIDなど）を保存しています
- WebアプリのアクセスレベルはYOUR_NEEDSに応じて調整してください
- 本番環境では`executeAs: "USER_ACCESSING"`を推奨します
