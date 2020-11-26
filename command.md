# ポートフォワード

- 自PCのlocalhost:8080への通信を、リモートサーバA(yahoo.com)から見た192.168.0.1:3306へ接続
  
  - 用途
    
    グローバルIPがないDBサーバへアクセスするような場合など
  
  - コマンド例
    
    ```
    ssh -L 8080:192.168.0.1:3306 username@yahoo.com
    ```
    
    ```
    ssh -i ~/.ssh/rsa -L 8080:192.168.0.1:3306 username@yahoo.com
    ```

# awkコマンド

- タブ区切りファイルを指定のフォーマットにして出力
  
  - 用途
    
    TSVファイルからSQL文を生成するなど
  
  - 元データ例(input.tsv)
    
    ```
    37      107     2017-07-31 20:47:50
    38      108     2017-07-31 20:47:50
    39      109     2017-07-31 20:47:50
    ```
  
  - 期待する結果
    
    ```
    UPDATE mst_quality SET created_at = '2017-07-31 20:47:50' WHERE id = 107;
    
    UPDATE mst_quality SET created_at = '2017-07-31 20:47:50' WHERE id = 108;
    
    UPDATE mst_quality SET created_at = '2017-07-31 20:47:50' WHERE id = 109;
    ```
  
  - コマンド例
    
    ```
    cat input.txt | awk -F"\t" '{printf "UPDATE mst_quality SET created_at = \047%s\047 WHERE id = %s;\n",$3,$1}'
    ```

# Gitコマンド

- ブランチ間のコミットログを比較。
  
  - 用途
    
    kochiraniaru_branchブランチにはあって、kochiraninai_branchにないコミットのログを表示
  
  - コマンド例
    
    ```
    git log --no-merges kochiraninai_branch..kochiraniaru_branch
    ```
# rsyncコマンド

- ローカルにリモートのファイルをダウンロードして同期
    
  - 上書き同期
    ```
    #テスト実行プション
    rsync -ahv --dry-run -e "ssh -p 2222 -i /home/foo/.ssh/private.rsa" username@hostname:/var/www/html/idoumoto/ /var/www/html/idousaki/
    ```
  
  - 移動元にないファイルは削除
  
    ```
    rsync -ahv --dry-run --delete --username@hostname:/var/www/html/idoumoto/ /var/www/html/idousaki/
    ```
# MySQL

  - 新規ユーザ作成
    - どこからでも接続可
      ```
      CREATE USER 'develop'@'%' IDENTIFIED BY 'password';
      ```
    - ローカルからのみ接続可
      ```
      CREATE USER 'develop'@'localhost' IDENTIFIED BY 'password';
      ```
  - 権限付与
    - 全権限
      ```
      GRANT ALL ON *.* TO 'develop'@'localhost';
      FLUSH PRIVILEGES;
      ```
    - 全DB＆テーブルに対して参照のみ権限
      ```
      GRANT SELECT ON *.* TO 'develop'@'localhost';
      FLUSH PRIVILEGES;
      ``` 
    - 特定DBの全テーブルに対して参照のみ権限
      ```
      GRANT SELECT ON sample_db.* TO 'develop'@'localhost';
      FLUSH PRIVILEGES;
      ``` 
    - 一般的なアプリで利用する権限(マイグレーション実行する場合)
      ```
      GRANT SELECT,UPDATE,DELETE,INSERT,ALTER,CREATE,DROP,REFERENCES,INDEX,LOCK TABLES ON *.* TO 'develop'@'localhost';
      FLUSH PRIVILEGES;
      ``` 
    - 一般的なアプリで利用する権限(マイグレーション実行しない場合)
      ```
      GRANT SELECT,UPDATE,DELETE,INSERT,LOCK TABLES ON *.* TO 'develop'@'localhost';
      FLUSH PRIVILEGES;
      ``` 
# ファイル操作系コマンドを確認しながら実行
間違うと甚大な被害が出るコマンドや、大量処理さえるコマンドを、確認しながら安全に実行する
 - ディレクトリ複製  
   権限も同一にし再帰的にコピー。`cp -ra /hoge/*`でコピーするとドットで始まるファイルが対象外のための回避策。
   ```
   pwd
   mkdir to-directory
   cp -ra /hogehoge/from-directory/. to-directory
   ```
   
 - 削除ファイルを確認してから、ディレクトリを再帰的的削除
   ```
   pwd
   ls -lda /hoge/all-delete-directory/*
   rm -rf  /hoge/all-delete-directory/*
   ``` 
 - 圧縮と解凍
 余分なディレクトリを作らず、アーカイブしたディレクトリと同構成で展開する
   ```
   #圧縮
   cd  /hoge/archive-dir
   ls
   tar -zcvf ~/hogehoge.tar.gz ./
   
   #解凍
   pwd
   mkdir open-dir
   cd open-dir
   ls
   tar -zxvf ~/hogehoge.tar.gz -C ./
   ``` 
