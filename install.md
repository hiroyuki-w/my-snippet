# サーバでメール送信する為の最低限設定(CentOS7)
postfixを利用して、メール送信だけする場合の必要最低限の設定
- 手順
  - インストール
    ```
    yum -y install postfix
    ```

  - /etc/postfix/main.cf編集
    
    - 追記  
    +myhostname = xxxx.co.jp
        
    - 変更  
    -inet_protocols = all  
    +inet_protocols = ipv4  
    
    - 追記  
    +masquerade_domains = xxx.co.jp
    

  - 自動起動設定＆起動
    ```
    systemctl enable postfix
    service postfix start
    ```