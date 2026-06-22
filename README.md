# elabftw-docker
elabftw を docker で動かすためのレポジトリ。

## セットアップと起動

1. **コンテナの起動**
   ```sh
   docker compose up -d
   ```

2. **データベースの初期化 (初回のみ)**
   コンテナが起動したあと、データベースを初期化します。
   ```sh
   docker compose exec web bin/init db:install
   ```

3. **サンプルデータの投入 (任意)**
   eLabFTW の `db:populate` は YAML 設定ファイルのパスを必須引数として受け取ります。
   このコマンドは現在のデータベースを削除してからサンプルデータを投入するため、既存データを残したい場合は実行しないでください。

   ```sh
   docker compose exec web bin/init db:populate -y /elabftw/src/tools/populate-config.yml.dist
   ```

   最小限のデータ投入にする場合は `-f` を指定します。
   ```sh
   docker compose exec web bin/init db:populate -y -f /elabftw/src/tools/populate-config.yml.dist
   ```

4. **アクセス**
   既定では `https://localhost:8080` に eLabFTW コンテナの HTTPS ポートを割り当てています。
   ブラウザで上記URLにアクセスしてください。（警告が表示される場合は、自己署名証明書によるものなのでそのまま続行してください）

## 便利コマンド

### コンテナ内で任意のコマンドを実行
`web` コンテナ内で任意のコマンドを実行できます。
```sh
docker compose exec web <command>
```
例（利用可能なコマンドの一覧を表示）:
```sh
docker compose exec web bin/init list
```

### ELN import/export

#### Export
Team ID `1` を ELN archive として export する例です。

```sh
docker compose exec web bin/console export:eln 1
```

実行後、eLabFTW が `/elabftw/exports/` 配下に作成した `.eln` ファイル名が表示されます。
表示されたコンテナ内のパスを `docker compose cp` でホスト側にコピーします。

```sh
docker compose cp web:/elabftw/exports/export-elabftw-2026-05-26_09-59-42-team-1.eln .
```

#### Import
ホスト側にある ELN archive をコンテナにコピーしてから、取り込み先 Team ID を指定して import します。
以下は `./export-elabftw-2026-05-26_09-59-42-team-1.eln` を Team ID `2` に import する例です。

```sh
docker compose cp ./export-elabftw-2026-05-26_09-59-42-team-1.eln web:/elabftw/export-elabftw-2026-05-26_09-59-42-team-1.eln
docker compose exec web bin/console import:eln -vv export-elabftw-2026-05-26_09-59-42-team-1.eln 2
```

import 完了時に `Delete ELN file? (y/N)` と確認された場合、コンテナ内のコピーしたアーカイブファイルを削除するには `y` を入力します。

### ローカル DB への接続
ホスト側からコンテナ内の MySQL に接続します。ポートは `3307` にマッピングされています。
```sh
mysql -h 127.0.0.1 -P 3307 -u elabftw -pfBTaVxFPymN3ujsBQuOIewUktW8XuSU elabftw
```
※ パスワードは `docker-compose.yml` 内の `MYSQL_PASSWORD` で設定されているデフォルト値です。変更している場合は適宜置き換えてください。

コンテナ内の mysql クライアントを直接使用して接続することもできます。
```sh
docker compose exec mysql mysql -u elabftw -pfBTaVxFPymN3ujsBQuOIewUktW8XuSU elabftw
```

## クリーンアップ

コンテナを停止し、削除します。
```sh
docker compose down
```
