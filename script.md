# MySQLバックアップ
- 転送先に公開鍵認証設定がありパスワードなしでSSH接続できるようにしてあることが前提
- 自身のサーバ内に日別バックアップを作成しgzip圧縮
- 別のサーバにrsyncコピーする

```
#!/bin/sh

#保持期限
period=7

#保存ファイル名
filename=`date +%y%m%d`

###################
#転送元設定
###################

#mysql接続先ホスト
dbhost='localhost'
#mysqlユーザとパスワード
dbuser='targetdbuser'
dbpass='targetdbpass'

#バックアップするDB名
dbname='targetdbname'

#自身のサーバの保存ディレクトリ
dirpath='/home/user_hoge/backup'

###################
#転送先設定
###################
#転送先サーバ
remotehost='111.222.333.444'
#転送先ディレクトリ
remotedir='/home/remote_user_hoge/backup'
#転送先への接続ユーザ
remoteuser='remote_user_hoge'
#転送先の秘密鍵パス
rsapath='~/.ssh/id_rsa'

###################
#スクリプト実行
###################

#バックアップ
mysqldump -h $dbhost -u $dbuser -p$dbpass | gzip > $dirpath/$filename.sql.gz

#期限切れファイル削除
oldfile=`date --date "$period days ago" +%y%m%d`
rm -f $dirpath/$oldfile.sql.gz

#別サーバへコピー
rsync -av -e "ssh -i $rsapath" --delete $dirpath/ $remoteuser@$remotehost:$remotedir

```

#MySQLデータのサーバ間同期
  同期元サーバで指定した任意のDBを、同期先のサーバに指定した名前のDB名で同期する
- リモートのサーバのMySQLログインできるよう、ポート開放やアカウントに権限が前提
- 同期先のDBは同期時にバックアップをとったあと、DROP処理が走ります

```
#!/bin/sh

set -e

##########################################
#接続先と、同期先DBのバックアップディレクトリ設定
##########################################

#同期元
from_dbhost='192.168.1.100'
from_dbuser='from_user'
from_dbpass='xxxxxxxx'
from_dbport='3306'
from_dbname='from_db'

#同期先
to_dbhost='192.168.1.101'
to_dbuser='to_user'
to_dbpass='xxxxxxxx'
to_dbport='3306'
to_dbname='to_db'
to_db_charset='utf8mb4'

#同期前のバックアップディレクトリ
dirpath='/home/vagrant/dbbackup'

###################
#スクリプト実行
###################
echo "start================"
mkdir -p $dirpath 2>/dev/null

echo "dump start"
#同期元ダンプ
mysqldump -h $from_dbhost -u $from_dbuser -p$from_dbpass -P $from_dbport $from_dbname > $dirpath/dump_sync_tmp.sql

#同期先backup
echo "backup exists db"
mysqldump -h $to_dbhost -u $to_dbuser -p$to_dbpass -P $to_dbport $to_dbname > $dirpath/$to_dbname`date +"%Y%m%d%H%M%s"`.sql

#同期先DB削除
echo "drop db"
mysql -h $to_dbhost -u $to_dbuser -p$to_dbpass -P $to_dbport -e "drop database if exists $to_dbname"

#同期先DB再作成
echo "create db"
mysql -h $to_dbhost -u $to_dbuser -p$to_dbpass -P $to_dbport -e "create database $to_dbname DEFAULT CHARACTER SET $to_db_charset"

#同期先インポート
echo "import db"
mysql -h $to_dbhost -u $to_dbuser -p$to_dbpass -P $to_dbport $to_dbname < $dirpath/dump_sync_tmp.sql
echo "sync end==========="

```