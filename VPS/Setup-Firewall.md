#セキュリティ設定
 VPSは、インターネット上で「野ざらし」されているので、必要なポート以外は不用意に解放せず、設置しないと踏み台にされたりイロイロと酷い目にあってしまいます。  
 ファイヤーウォールを設定する必要があるのですが、少し時間がかかりました。  
 何と言うか、設定に時間がかかったと言うより、ubuntuではufwコマンドで設定するもんやと思っていた自分だったからですが、さくらVPSではiptablesで設定する必要があると、散々試してから理解できました。  
 では、ともかく設定しますが、ファイヤーウォール→SSH（シェルの使い勝手がVPSのコンソールで構わないのなら不要です。）で設定します。  

---
##iptablesを編集する
 iptablesはネットワーク関連の設定ファイルでテキストエディタで設定できます。  
 まずは、場所を探して見ましょう。

1. VPSにログインします。
1. iptablesは、バイナリーファイルですので、元になるiptables.rulesを編集、保存し再起動を行います。
1. iptables.rulesは以下の場所に保存されています。

_/etc/iptables/iptables.rules_  

このファイルを編集します。編集には管理者権限が入りますので、以下のように入力します。

```
sudo nano /etc/iptables/iptables.rules
```

VPS上では、Minecraftとホームページぐらいを動かす予定なので、そのポートとSSHを接続するためのポートを開放します。  
iptablesの詳細な説明は、専門書などで確認してください。
```
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
#（シャープから始まるとコメント）
#SSHはデフォルト TCP 22番ポート以外にする。
#-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport ***** -j ACCEPT
#UDP 53はDNS
-A INPUT -p udp --sport 53 -j ACCEPT
#TCP 80はHTTP
-A INPUT -p tcp --dport 80 -j ACCEPT
#-A INPUT -p tcp --dport 443 -j ACCEPT
#TCP 25565はMinecraft
-A INPUT -p tcp --dport 25565 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
```

これで、使用したいポートの開放設定ができました。これで再起動をかけるわけですが、その前にSSHの設定を行います。

##SSH（Secure Shell）の設定
 SSHは、安全な通信を行えるターミナルソフトです。
 さくらVPSは、OSインストール直後では、ほぼSSHのポートのみ開放されており、その他は閉塞されています。  
 まずSSHの設定をしましょう。

1. 管理者へ昇格します。  管理者への昇格は _su -_

