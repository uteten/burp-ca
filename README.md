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


====
# AndroidのプリインストールされたCA認証局に俺俺CAを追加する手順  
====

AndroiのCA認証局は以下のディレクトリにおいてある  
`/system/etc/security/cacerts`  
Rootを取っていればここにCAを追加すれば良い。ファイル名はsubject_hash_old以外は無視される。


1. Android用のCA認証局ファイルを作成  
    `openssl x509 -noout -subject_hash_old -in cacert.pem -inform pem`  
    66326da7  
    ここで表示される8桁のHASHに.0を追加した文字列をファイル名にする  
    `copy cacert.pem 66326da7.0`  
    `openssl x509 -inform PEM -text -noout -in 66326da7.0 >> 66326da7.0`

2. Androidに配置  
    上で作った66326da7.0をSDカードなどに入れてAndroidに接続しadbでファイルを配置  
    `> adb shell`  
    `$ su`  
    `# cd /system/etc/security/cacerts`  
    `# mount -o rw,remount /system`  
    `# cp  /mnt/media_rw/****/66326da7.0 .`  
    `# chmod 644 66326da7.0`  

3.Android端末で確認
    設定＞詳細背RT邸＞セキュリティ＞信頼できる認証情報  
    システムのタブに追加したCAがあればOK。配置したファイル名が異なると見えていても無視されるので注意
    
