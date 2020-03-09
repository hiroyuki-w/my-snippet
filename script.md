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
    
