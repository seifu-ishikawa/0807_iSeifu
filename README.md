# 🎓 生徒名簿管理システム

<div align="center">

![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?logo=javascript&logoColor=black)

プログラミングスクール向けの生徒名簿管理Webアプリケーション

[デモを見る](https://your-username.github.io/student-roster-system/) | [ドキュメント](./name_list.html) | [問題を報告](https://github.com/your-username/student-roster-system/issues)

</div>

## 📋 目次

- [概要](#概要)
- [主な機能](#主な機能)
- [デモ](#デモ)
- [インストール](#インストール)
- [使い方](#使い方)
- [技術スタック](#技術スタック)
- [プロジェクト構成](#プロジェクト構成)
- [開発](#開発)
- [貢献](#貢献)
- [ライセンス](#ライセンス)

## 🌟 概要

生徒名簿管理システムは、プログラミングスクールの生徒情報を効率的に管理するためのWebアプリケーションです。最大40名のクラス運営に最適化されており、直感的なインターフェースで生徒の基本情報管理、CSV入出力、高度な検索機能などを提供します。

### なぜこのシステムが必要か？

- 📊 **データの一元管理** - 生徒情報を一箇所で管理
- ⚡ **リアルタイム更新** - 変更が即座に反映
- 🔍 **高度な検索機能** - 必要な情報をすぐに見つける
- 📥 **CSV対応** - ExcelやGoogleスプレッドシートとの連携

## ✨ 主な機能

### 基本機能
- ✅ **生徒情報管理** - 氏名、フリガナ、生年月日、性別、得意分野の管理
- ✅ **新規登録** - 簡単な入力フォームで生徒を追加
- ✅ **編集・削除** - ワンクリックで情報の更新・削除
- ✅ **40名制限** - クラスサイズに最適化

### 検索・フィルタリング
- 🔍 **リアルタイム検索** - 入力と同時に結果を表示
- 📝 **複数フィールド検索** - 氏名、フリガナ、得意分野で検索
- 🎯 **部分一致検索** - 柔軟な検索オプション

### データ入出力
- 📥 **CSVダウンロード** - 現在のデータをCSV形式でエクスポート
- 📤 **CSVアップロード** - CSVファイルからデータをインポート
- 💾 **自動バックアップ** - 定期的なデータ保存を推奨

### ユーザビリティ
- 🎨 **レスポンシブデザイン** - PC、タブレット、スマートフォン対応
- ⌨️ **キーボードショートカット** - 効率的な操作
- 🌈 **美しいUI** - グラデーション配色の modern デザイン

## 🖼️ スクリーンショット

<div align="center">

### メインインターフェース
```
┌─────────────────────────────────────────────────────────┐
│  🎓 生徒名簿管理システム                                   │
│  プログラミングスクール 40名クラス                          │
├─────────────────────────────────────────────────────────┤
│  🔍 [検索ボックス] [➕新規追加] [📥CSV↓] [📤CSV↑]        │
├─────────────────────────────────────────────────────────┤
│  氏名    │ フリガナ    │ 生年月日  │ 性別 │ 得意分野     │
│  山田太郎 │ ヤマダタロウ │ 2005-04-15│ 男  │ Webフロント  │
│  佐藤花子 │ サトウハナコ │ 2005-06-22│ 女  │ データサイエンス│
└─────────────────────────────────────────────────────────┘
```

</div>

## 🚀 インストール

### オプション 1: 直接ダウンロード

1. このリポジトリをクローンまたはダウンロード
```bash
git clone https://github.com/your-username/student-roster-system.git
cd student-roster-system
```

2. `index.html` をブラウザで開く
```bash
# macOS
open index.html

# Windows
start index.html

# Linux
xdg-open index.html
```

### オプション 2: GitHub Pages でホスティング

1. リポジトリをフォーク
2. Settings → Pages → Source を `main` ブランチに設定
3. `https://[your-username].github.io/student-roster-system/` でアクセス

### オプション 3: ローカルサーバーで実行

```bash
# Python 3
python -m http.server 8000

# Python 2
python -m SimpleHTTPServer 8000

# Node.js (http-server をインストール済みの場合)
npx http-server
```

## 📖 使い方

### 基本操作

#### 1. 新規生徒の追加
```
1. 「➕ 新規生徒追加」ボタンをクリック
2. 必要情報を入力
   - 氏名（必須）
   - フリガナ（必須）
   - 生年月日（必須）
   - 性別
   - 得意分野
3. 「保存」をクリック
```

#### 2. 生徒情報の編集
```
1. 編集したい生徒の行の「✏️」ボタンをクリック
2. 情報を修正
3. 「更新」をクリック
```

#### 3. 生徒情報の削除
```
1. 削除したい生徒の行の「🗑️」ボタンをクリック
2. 確認ダイアログで「削除」を選択
```

### キーボードショートカット

| ショートカット | 動作 |
|--------------|------|
| `Ctrl + N` | 新規生徒追加 |
| `Ctrl + S` | CSVダウンロード |
| `Ctrl + O` | CSVアップロード |
| `Ctrl + F` | 検索フォーカス |
| `Esc` | ダイアログを閉じる |

### CSV形式

CSVファイルは以下の形式に従ってください：

```csv
氏名,フリガナ,生年月日,性別,得意分野
山田太郎,ヤマダタロウ,2005-04-15,男,Webフロントエンド
佐藤花子,サトウハナコ,2005-06-22,女,データサイエンス
```

## 🛠️ 技術スタック

- **フロントエンド**
  - HTML5
  - CSS3 (Flexbox, Grid, Animations)
  - Vanilla JavaScript (ES6+)

- **デザイン**
  - レスポンシブデザイン
  - グラデーション配色
  - マテリアルデザイン原則

- **機能**
  - LocalStorage API (データ永続化)
  - FileReader API (CSV読み込み)
  - Blob API (CSVダウンロード)

## 📁 プロジェクト構成

```
student-roster-system/
│
├── index.html           # メインアプリケーション
├── name_list.html       # 取扱説明書
├── README.md           # このファイル
│
├── css/                # スタイルシート（組み込みの場合）
├── js/                 # JavaScriptファイル（組み込みの場合）
└── docs/               # 追加ドキュメント
```

## 🔧 開発

### 前提条件

- モダンブラウザ (Chrome, Firefox, Safari, Edge)
- テキストエディタ (VS Code推奨)
- Git

### 開発環境のセットアップ

```bash
# リポジトリをクローン
git clone https://github.com/your-username/student-roster-system.git

# ディレクトリに移動
cd student-roster-system

# VS Codeで開く
code .
```

### カスタマイズ

#### 色の変更
`index.html` 内の CSS 変数を編集：
```css
:root {
  --primary-color: #667eea;
  --secondary-color: #764ba2;
}
```

#### 最大生徒数の変更
JavaScript内の定数を編集：
```javascript
const MAX_STUDENTS = 40; // この値を変更
```

## 🤝 貢献

プルリクエストを歓迎します！大きな変更の場合は、まずissueを開いて変更内容を議論してください。

1. プロジェクトをフォーク
2. フィーチャーブランチを作成 (`git checkout -b feature/AmazingFeature`)
3. 変更をコミット (`git commit -m 'Add some AmazingFeature'`)
4. ブランチにプッシュ (`git push origin feature/AmazingFeature`)
5. プルリクエストを開く

### 貢献ガイドライン

- コードスタイルを統一
- 適切なコメントを追加
- テストを実行
- ドキュメントを更新

## 📝 ライセンス

このプロジェクトは MIT ライセンスの下で公開されています。詳細は [LICENSE](LICENSE) ファイルを参照してください。

## 👥 作者

- **あなたの名前** - *初期開発* - [GitHub](https://github.com/your-username)

## 🙏 謝辞

- プログラミングスクールの先生方と生徒の皆様
- オープンソースコミュニティ
- すべての貢献者

## 📞 サポート

問題が発生した場合は、以下の方法でサポートを受けられます：

- [GitHub Issues](https://github.com/your-username/student-roster-system/issues) で問題を報告
- [Discussions](https://github.com/your-username/student-roster-system/discussions) で質問
- メール: support@example.com

## 🔮 今後の機能

- [ ] 生徒の出席管理機能
- [ ] 成績管理機能
- [ ] 保護者連絡先管理
- [ ] 複数クラス対応
- [ ] データベース連携
- [ ] API対応
- [ ] 多言語対応

## 📊 プロジェクトステータス

プロジェクトは現在 **アクティブ開発中** です。定期的なアップデートと改善を行っています。

---

<div align="center">

**[⬆ トップに戻る](#-生徒名簿管理システム)**

Made with ❤️ by [Your Name](https://github.com/your-username)

</div>