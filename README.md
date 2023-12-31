# ISUCON 勉強メモ

## モニタリング
### Linuxサーバにおける単純なモニタリングコマンド
- top ... CPU使用率
  - %CPU(s) ... CPU利用率
  - id ... アイドル率
- free ... メモリ使用率 
  - free --human ... 人が読みやすい形にしてくれる
- stress ... サーバに負荷をかける
  - stress -cpu 2 
### SaaSのモニタリングツール
- Prometheus
  - node_exporter ... リソース取得エージェント
  - ISUCONでは使わない？
### 注意点
- グラフ軸の単位・数値に注意
- 同じサーバでモニタリングツールを使うと負荷が変わる

## 負荷試験
サーバを起動してからアプリが立ち上がるまで数十分かかる

### グローバルIP 確認方法
```bash
curl https://ifconfig.me
```

### ベンチマーク方法
```bash
cd /home/isucon/private_isu.git/benchmarker
./bin/benchmarker -u ./userdata -t http://{競技用インスタンスのIPアドレス}/
```

### スロークエリログ
#### 設定方法
- /etc/mysql/myswl.conf.d/mysqld.cnf
```bash
[mysqld]
slow_query_log=1
slow_query_log_file=/var/log/mysql/mysql-slow.log
long_query_time=0
```

- 設定変更したらmysqlを再起動する
```bash
sudo systemctl restart mysql
```

#### 保存時 permission deniedの解消方法
```bash
sudo chmod 777 ファイル名
sudo chmod 644 ファイル名
```
- 777だとスロークエリログ吐かなくなるので注意
- もっといいやり方ありそう
  - sudo nano ファイルの方が楽

#### 確認方法
```bash
sudo mysqldumpslow /var/log/mysql/mysql-slow.log 
```

- 新しくログを取りたい時は次を打つ
```bash
sudo rm /var/log/mysql/mysql-slow.log
sudo mysqladmin flush-logs
```

### mysql
#### ログイン
```bash
sudo mysql -u root
```

#### データベース一覧確認
```bash
SHOW DATABASES;
```

#### データベースの構造確認
```bash
SHOW CREATE TABLE <テーブル名>\G;
```
- ERROR 1049 (42000): Unknown database 'comments'が出る時
```bash
use isuconp;
show tables; # 対象の表があるか確認
```

#### クエリの実行動作を確認
- EXPLAIN
```bash
EXPLAIN SELECT * FROM comments WHERE post_id = 9995 ORDER BY created_at DESC LIMIT 3\G
```

#### post_idカラムにインデックスを付与
```bash
ALTER TABLE comments ADD INDEX post_id_idx(post_id);
```
- もう一度クエリの実行動作を確認

### プロセスの並列処理
- webapp/ruby/unicorn_config.rbのworker_processを1から4に変更
```bash
worker_processes 4
preload_app true
listen "0.0.0.0:8080"
```
- アプリケーションの再起動
```bash
systemctl restart isu-ruby
```
