# YOURLS Docker 日本語版

YOURLSとMySQL 8.0をDocker Composeで簡単に構築できる環境です。日本語化ファイル（YOURLS-ja_JP）を組み込んでいます。

## 概要

このプロジェクトは、[YOURLS](https://yourls.org/)（Your Own URL Shortener）をDockerコンテナで簡単に実行できるようにしたものです。以下の特徴があります：

- **Docker Compose**: YOURLSとMySQL 8.0を1つの設定ファイルで管理
- **日本語対応**: YOURLS-ja_JPによる完全な日本語化
- **簡単セットアップ**: 環境変数による柔軟な設定
- **データ永続化**: ボリュームマウントによるデータ保護
- **SSL対応**: Nginx + Let's Encryptによる本番HTTPS環境

## ライセンス

このプロジェクトはMITライセンスの下で公開されています。

- **YOURLS**: MIT License
- **YOURLS-ja_JP**: MIT License
- **このプロジェクト**: MIT License（詳細は[LICENSE.txt](LICENSE.txt)を参照）

## 必要要件

- Docker
- Docker Compose
- Git

## インストール

### 1. リポジトリのクローン

```bash
git clone https://github.com/potofo/yourls
cd yourls
```

### 2. 環境変数の設定

`.env.example`をコピーして`.env`ファイルを作成し、必要に応じて設定を変更します。

```bash
cp .env.example .env
```

`.env`ファイルの主要な設定項目：

```bash
# ドメイン名（SSL使用時に設定）
YOURLS_DOMAIN=example.com

# YOURLSのサイトURL（外部アクセス用のURL）
YOURLS_SITE=https://example.com

# YOURLS管理者のユーザー名とパスワード
YOURLS_USER=admin
YOURLS_PASS=admin

# 言語設定（ja_JP: 日本語）
YOURLS_LANG=ja_JP

# タイムゾーンオフセット（日本標準時は9）
YOURLS_HOURS_OFFSET=9

# 外部アクセス用のポート
YOURLS_EXTERNAL_PORT=80
YOURLS_EXTERNAL_SSLPORT=443

# MySQL設定
MYSQL_ROOT_PASSWORD=example
MYSQL_DATABASE=yourls
MYSQL_USER=yourls
MYSQL_PASSWORD=yourls
```

**セキュリティのため、本番環境では必ずパスワードを変更してください。**

### 3. SSL証明書の取得と初回起動

> **前提条件**: 公開可能なドメイン名を取得済みで、ポート80・443がサーバーに到達できること。

#### 3-1. HTTP のみの nginx 設定を準備（ACME チャレンジ用）

`YOUR_DOMAIN` を実際のドメイン名に置き換えて実行します。

```bash
sed 's/YOUR_DOMAIN/example.com/g' nginx/nginx.init.conf > nginx/nginx.conf
```

#### 3-2. nginx を起動

```bash
docker compose up -d nginx
```

#### 3-3. Let's Encrypt 証明書を取得

`example.com` と `your@email.com` を実際の値に置き換えて実行します。

```bash
docker compose run --rm certbot certonly \
  --webroot -w /var/www/certbot \
  -d example.com \
  --email your@email.com \
  --agree-tos \
  --no-eff-email
```

#### 3-4. HTTPS 設定に切り替えて全サービスを起動

```bash
sed 's/YOUR_DOMAIN/example.com/g' nginx/nginx.prod.conf > nginx/nginx.conf
docker compose up -d
```

### 4. YOURLSのセットアップ

ブラウザで `https://example.com/admin/` にアクセスし、初期セットアップを完了します。

1. 「Install YOURLS」ボタンをクリック
2. データベーステーブルが自動的に作成されます
3. `.env`で設定したユーザー名とパスワードでログイン

## 使用方法

### コンテナの起動

```bash
docker compose up -d
```

### コンテナの停止

```bash
docker compose down
```

### コンテナのログ確認

```bash
# 全コンテナのログ
docker compose logs -f

# YOURLSコンテナのみ
docker compose logs -f yourls

# MySQLコンテナのみ
docker compose logs -f db
```

### コンテナの再起動

```bash
docker compose restart
```

## データベースの初期化

データベースを完全に削除して初期状態に戻す場合は、以下の手順を実行します。

### ⚠️ 警告

この操作を実行すると、**全ての短縮URLとデータが完全に削除**されます。実行前に必ずバックアップを取ってください。

### 初期化手順

1. **コンテナの停止とボリュームの削除**

```bash
docker compose down --volumes
```

2. **MySQLデータディレクトリの削除**

```bash
# Windowsの場合（PowerShell）
Remove-Item -Recurse -Force .\volumes\mysql

# macOS/Linuxの場合
rm -rf ./volumes/mysql
```

3. **コンテナの再起動**

```bash
docker compose up -d
```

4. **YOURLSの再セットアップ**

ブラウザで `http://localhost/admin/` にアクセスし、再度初期セットアップを実行します。

## プロジェクト構成

```
.
├── config/
│   ├── config.php              # YOURLS設定ファイル
│   ├── config-sample.php       # 設定ファイルのサンプル
│   ├── languages/
│   │   └── ja_JP.mo           # 日本語翻訳ファイル
│   ├── plugins/               # プラグインディレクトリ
│   └── pages/                 # カスタムページ
├── nginx/
│   ├── nginx.conf             # Nginx設定ファイル（実行中に使用）
│   ├── nginx.init.conf        # 初回証明書取得用設定（HTTP のみ）
│   ├── nginx.prod.conf        # 本番用HTTPS設定
│   └── webroot/              # ACME チャレンジ用（自動生成）
├── volumes/
│   ├── mysql/                 # MySQLデータ（自動生成）
│   └── letsencrypt/          # SSL証明書（自動生成）
├── .env                       # 環境変数設定（git管理外）
├── .env.example              # 環境変数のサンプル
├── docker-compose.yml        # Docker Compose設定
├── LICENSE.txt               # ライセンスファイル
└── README.md                 # このファイル
```

## プラグインの有効化

プラグインを有効化するには、YOURLS管理画面から以下の手順で行います：

1. ブラウザで`http://localhost/admin/`にアクセスしてログイン
2. 上部メニューから「機能拡張を管理」をクリック
3. 有効化したいプラグインの「有効」ボタンをクリック

利用可能なサンプルプラグイン：
- `random-shorturls` - ランダムな短縮URLを生成
- `hyphens-in-urls` - URLにハイフンを使用可能に
- `random-bg` - ランダムな背景パターン
- `sample-toolbar` - カスタムツールバーのサンプル

## SSL証明書の自動更新

`certbot` コンテナが12時間ごとに `certbot renew` を自動実行し、有効期限30日以内の証明書を更新します。
更新後に nginx へ反映するため、定期的（週1回程度）に以下を実行してください。

```bash
docker compose exec nginx nginx -s reload
```

## トラブルシューティング

### ポート80が使用中

ポート80が既に使用されている場合、`.env`ファイルの`YOURLS_EXTERNAL_PORT`を変更してください。

```bash
YOURLS_EXTERNAL_PORT=8080
```

変更後、`http://localhost:8080/admin/` でアクセスしてください。

### データベース接続エラー

データベースコンテナの起動を確認してください：

```bash
docker compose ps
```

全てのコンテナが`Up`状態であることを確認します。

### 日本語が表示されない

1. `.env`ファイルで`YOURLS_LANG=ja_JP`が設定されていることを確認
2. `config/languages/ja_JP.mo`が存在することを確認
3. コンテナを再起動: `docker compose restart`

## バックアップ

### データベースのバックアップ

```bash
docker compose exec db mysqldump -u yourls -pyourls yourls > backup.sql
```

### データベースの復元

```bash
docker compose exec -T db mysql -u yourls -pyourls yourls < backup.sql
```

## 参考リンク

- [YOURLS公式サイト](https://yourls.org/)
- [YOURLS GitHub](https://github.com/YOURLS/YOURLS)
- [YOURLS-ja_JP](https://github.com/havill/YOURLS-ja_JP)
- [Docker公式ドキュメント](https://docs.docker.com/)
- [query-string-keeper](https://github.com/littleskyfish/query-string-keeper)

## コントリビューション

バグ報告や機能提案は、GitHubのIssuesでお願いします。

## ライセンス

MIT License - 詳細は[LICENSE.txt](LICENSE.txt)を参照してください。
- yourls
  MIT License - 詳細は[LICENSE.txt](LICENSE.txt)を参照してください。

- plugins
  [query-string-keeper](https://github.com/littleskyfish/query-string-keeper) - 詳細は [LICENSE](https://github.com/littleskyfish/query-string-keeper/blob/main/LICENSE)を参照してください。