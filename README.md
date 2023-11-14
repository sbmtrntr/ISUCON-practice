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

### ベンチマーク方法
```bash
cd /home/isucon/private_isu.git/benchmarker
./bin/benchmarker -u ./userdata -t http://{競技用インスタンスのIPアドレス}/
```

### permission deniedの解消方法
```bash
sudo chmod 777 ファイル名
```


### スロークエリログの設定方法
- /etc/mysql/myswl.conf.d/mysqld.cnf
```bash
[mysqld]
slow_query_log = 1
slow_query_log_file = /var/log/mysql/mysql-slow.log
long_query_time = 0
```

- 設定変更したらmysqlを再起動する
```bash
sudo systemctl restart mysql
```

### スロークエリログの確認方法
```bash
sudo mysqldumpslow /var/log/mysql/mysql-slow.log 
```

### グローバルIP 確認方法
```bash
curl https://ifconfig.me
```