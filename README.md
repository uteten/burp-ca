# burp用のCA認証局作成手順
====
# 環境
CentOS7

1. 鍵作成  
    Linuxで以下を実行  
    `cd /etc/pki`  
    `cp tls/misc/CA misc/CA.org`  
    `vi tls/misc/CA`  CADAY=を365など短く編集  
    `/etc/pki/tls/misc/CA -newca`  
    適当にエンターとパスフレーズを入力。  
    以下のファイルが生成される。  
    `/etc/pki/CA/private/cakey.pem`鍵  
    `/etc/pki/CA/cacert.pem`証明書  

2. PEMを作成  
    `mkdir tmp2`  
    `openssl rsa  -in /etc/pki/CA/private/cakey.pem -out tmp2/private.pem -outform PEM`  
    `openssl x509 -in /etc/pki/CA/cacert.pem -out tmp2/cacert.pem -outform PEM`  
    以下のファイルが生成される  
    `tmp2/cacert.pem`  
    `tmp2/private.pem`  

3. PFX作成(Burpインポート用)  
    `openssl pkcs12 -export -in cacert.pem -inkey private.pem -out ca.pfx`  

4. ca.pfxをBurpにインストール  
    メニューからProxy->Options  
    「Proxy Listeners」の「Import / export CA certificate」ボタンを押下し、Import>Certificate and private key from PKCS#12 keystore1  

    
