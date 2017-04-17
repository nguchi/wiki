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

1. 管理者へ昇格します。  管理者への昇格は
```su -```
で、行えます。
1. パスワードを確認して来ますので、答えてください。
1. 設定ファイルを編集します。  設定ファイルは _/etc/ssh/sshd_config_にあります。  同じディレクトリにssh_configファイルがありますが、これはクライアントを使う時のファイルですので、ここでは設定しません。

```
nano /etc/ssh/sshd_config
```
で、編集を始めます。

中身の説明は、専門書などで調べてください。こんな感じになります。  （とりあえず、パスワード認証有効の状態です。きちんと設定が終われば無効にするのが良いです。）
```
# Package generated configuration file
# See the sshd_config manpage for details

# What ports, IPs and protocols we listen for
#Port 22
Port *****
# Use these options to restrict which interfaces/protocols sshd will bind to
#ListenAddress ::
#ListenAddress 0.0.0.0
Protocol 2
# HostKeys for protocol version 2
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_dsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key
#Privilege Separation is turned on for security
UsePrivilegeSeparation yes

# Lifetime and size of ephemeral version 1 server key
KeyRegenerationInterval 3600
ServerKeyBits 1024

# Logging
SyslogFacility AUTH
LogLevel INFO

# Authentication:
LoginGraceTime 120
#PermitRootLogin prohibit-password
PermitRootLogin no
StrictModes yes

RSAAuthentication yes
PubkeyAuthentication yes
#AuthorizedKeysFile     %h/.ssh/authorized_keys

# Don't read the user's ~/.rhosts and ~/.shosts files
IgnoreRhosts yes
# For this to work you will also need host keys in /etc/ssh_known_hosts
RhostsRSAAuthentication no
# similar for protocol version 2
HostbasedAuthentication no
# Uncomment if you don't trust ~/.ssh/known_hosts for RhostsRSAAuthentication
#IgnoreUserKnownHosts yes

# To enable empty passwords, change to yes (NOT RECOMMENDED)
PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
ChallengeResponseAuthentication no

# Change to no to disable tunnelled clear text passwords
PasswordAuthentication yes
PermitRootLogin no
# Kerberos options
#KerberosAuthentication no
#KerberosGetAFSToken no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
X11Forwarding yes
X11DisplayOffset 10
PrintMotd no
PrintLastLog yes
TCPKeepAlive yes
#UseLogin no
#MaxStartups 10:30:60
#Banner /etc/issue.net
# Allow client to pass locale environment variables
AcceptEnv LANG LC_*
Subsystem sftp /usr/lib/openssh/sftp-server
# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the ChallengeResponseAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via ChallengeResponseAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and ChallengeResponseAuthentication to 'no'.
UsePAM yes
```

servicesファイルもついてに変更しておきましょう。
servicesは、一般的なネットワークサービスの名称とポート番号を設定しているファイルで、sshのポートも定義されているため、VPSで利用するポートに変更しておきます。  servicesは _/etc/services_ にあります。

これを編集します。
```
nano /etc/services
```

サービス名 ssh を探し、設定したポート番号に変更しましょう。
```
ftp-data        20/tcp
ftp             21/tcp
fsp             21/udp          fspd
#ssh            22/tcp                          # SSH Remote Login Protocol
#22がsshの既定値なので、変更する。
ssh             *****/tcp
#ssh            22/udp
ssh             22029/udp
telnet          23/tcp
smtp            25/tcp          mail
```
設定が終われば、再起動します。

```
reboot
```
で、再起動します。
