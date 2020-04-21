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