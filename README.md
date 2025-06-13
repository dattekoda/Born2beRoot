# Born2beRoot

環境構築済み

## sudo設定

`su`でroot userとしてログインする。  
`apt-get update -y`でそれまでダウンロードしたパッケージ情報をアップデート。  
`apt-get upgrade -y`でアップデートした情報を獲得。  
`apt install sudo`でsudoをインストール。  

### apt と apt-get の違い。
> - apt-getは古くからあるやり方で低レベル。インターフェースは安定しており変わらないことが保証される。helper関数は少ないやり方で味のない出力。
> - apt はより新しく、高レベルのAPIでdebian 8 / Ubuntu 16.04 で登場した。見やすいUIで進捗バーなどを標準搭載している。またhelper関数も充実。ただ、安定した出力は保証されないため、オートメーション化で使用することは非推奨。

### aptitude
> - 高レベルのパッケージ管理ツール。依存関係を解消する便利な機能を搭載する。古くからあるツールでaptより前から存在する。

`gpasswd -a khanadat sudo`でユーザーをsudo groupに追加できる。  
`usermod -aG sudo khanadat`でもユーザーをsudo groupに追加できる。 
→これもできるが、タイポして`usermod -a sudo khanadat`でkhanadat以外のsudo groupのユーザーがいなくなる可能性があるため危険。　　
`getent group sudo`はsudoにいるユーザーを確認するコマンド。  

### `sudoers file`
> sudoコマンドを使って誰がどのコマンドを実行できるかを定義する設定ファイル。場所は `/etc/sudoers` 、ディレクトリ `/etc/sudoers.d`に細かい設定ファイルを置くことも多い。
> どのユーザーがどのコマンドを、どのユーザー権限で実行できるかを制御できる。パスワード入力の要不要や、実行ログの保存方法なども設定可能。
> 直接ファイルを開くのではなく、`visudo`コマンドで編集するように注意する。文法チェックが自動で入るので、誤った設定でシステム管理者権限を失うリスクを減らせる。

## Git と Vimのインストール
`apt-get install git -y`でGitをインストール。
`git --version`でGitのインストールを確認。

同様に、
`apt-get install vim -y`でVimをインストール。
`vim --version`でVimのインストールを確認。

## SSHのインストールと構成

### SSHとは
> Secure Shellの略。ネットワーク越しに、安全に「シェル」（コマンド操作環境）を使ったり、ファイル転送をしたりするためのプロトコル（通信規約）のこと。
> 暗号化された通信で、（RSAなどの）公開鍵暗号方式をまずはじめに使い、その後は対象鍵暗号（AESなど）で通信をまるごと暗号化する。これによって、中間者攻撃や盗聴を防ぐことができる。
> パスワード認証や、公開鍵認証（サーバーに公開鍵を置いて、秘密鍵を持っているクライアントのみを署名して認証する仕組み、パスワードよりも安全で自動化に向いている）、さらに二要素認証などにも対応可能で認証方式が豊富。
> 「暗号化、認証付きで安全にリモート操作したりファイルをやり取りしたりできる仕組み」でTelnetやFTPのような平文（暗号化なし）の古いツールの代わり
> 
> 主な用途として、サーバーへのリモートログイン、(`ssh user@hostname`)、リモートでのコマンド実行、ファイル転送（scp, sftp)
> ポートフォワーディング（インターネットから特定のポート番号に宛てて届いたパケットを予め設定しておいたLAN側の機器に転送する機能）
> 
> 構成要素は
> クライアント（ssh)：自分の端末で動くプログラム
> サーバー（sshd)：接続先で待ち受けるデーモン。通常はTCPのポート番号が22。

`sudo apt install openssh-server`でsshサーバーを管理するパッケージをインストール。  
`sudo sytemctl status ssh`でssh serverの状態を確認できる。  
`sudo vim /etc/ssh/sshd_config`でsshサーバーの設定を編集する。  
`#Port22`の部分を見つけてこの数字を`4242`に変更後、行頭の`#`を削除。  
そして、保存後Vimを抜ける。  

### TCP/IPとは
インターネットやLAN上でコンピュータ同士がデータをやり取りするためのプロトコル（通信規約）の一郡のこと。正式名称は「Transmission Control Protocol/ Internet Protocol」。大きく４つのレイヤーに分かれる。
- 1.ネットワークインターフェース層（リンク層）
> 物理的なネットワーク（Ethernet, Wi-Fi, PPPなど）にデータを載せる役割がある。フレーム（Ethernetフレームなど）単位で送受信を行う。
- 2.インターネット層（IP層）
> IP層が主役で、データを「パケット」と呼ばれる単位に分けて、送信元・宛先IPアドレスを付与する。経路選択（ルーティング）をして最適経路で届ける。
- 3.トランスポート層
> TCP（Transmission Control Protocol）：接続指向型で信頼性・順序保証がある。パケットの再送、誤り検出、フロー制御などを行う。
> UDP（User Diagram Protocol）：接続不要型で信頼性は低い。リアルタイム性重視の動画配信やオンラインゲームなどで使われる。
- 4.アプリケーション層
> HTTP（ウェブ）、SMTP（メール）、FTP（ファイル転送）、DNS（ドメイン名とIPアドレスを対応付けて管理するシステム）など、ユーザが直接使うサービスのプロトコル群

例）
> 1. アプリケーション層で「HTTPリクエストを送る」
> 2. トランスポート層でデータを細かく分割し、シーケンス番号をつけて通信を開始する。
> 3. インターネット層が送受信先のIPアドレスをパケットに書き込む。
> 4. リンク層で実際のネットワーク機器にフレーム(を流す。
> 5. 受け手では、逆の手順でカプセル化を解除して最終的にアプリケーション層へデータが届く。


### Port番号とは
> TCP/IPプロトコルにおいて「どのアプリケーションにデータを届けるか」を識別するための16ビット（0 - 65535）の番号。例えば、IPアドレスが建物の住所だとすると、ポート番号はその建物内の部屋番号に当たる。パケットがホストに届くとき、宛先ポート番号を見て適切なアプリケーション（Webサーバー、SSHデーモンなど）に振り分ける。
> 同一ホスト上での識別として同じIPアドレスでも、異なるポート番号を使うことで復数のサービスを同時に動かせる。

`sudo grep Port /etc/ssh/sshd_config`でポート設定が合っているかを確認する。  
最後に、`sudo service ssh restart`で、SSH Serviceをリスタートする。
## ufwのインストールと構成
### ファイアウォールとは
> ネットワークの出入り口に設置され、不正アクセスや不審な通信からネットワークを保護するセキュリティシステム。予めルールを設定しておいてそれに従いどの通信を許可するかを判別する。

### UFWとは
> Debianで標準的に利用できる、「iptablesを簡単に設定するツール」。UFWを利用することで、「外部からの接続は基本的に受け付けない」、「SSHだけは許す」といった設定をiptablesに比べて格段に少ない操作で実現できる。
> iptablesはもともと、Linuxでファイアウォールを設定するための標準的なツール。iptablesの各種モジュールを利用してNetFilterを設定した場合、ファイアウォールとして求められる、ほぼありとあらゆる設定を行うことができる。また、多くの設定方法が存在しさらにその設定の自由度も高い。
> しかし、iptablesはその自由度と機能の豊富さから、設定を行うのに専門の知識が必要になる。少なくともいきなりiptablesを利用して自由自在に設定を行うことは困難となる。なぜならその設定は根本的な部分から行うことができ非常に複雑になるから。
> そこでufw（Uncomplicated FireWall）を使うことでソフトウェアファイアウォールとしての機能を、できるだけ単純なインターフェースで設定できる。

`apt-get install ufw`でまず最初にUFWをインストールする。  
`sudo ufw enable`でUFWを有効化する。  
`sudo ufw status numbered`でUFWの状態を確認できる。  
`sudo ufw allow ssh`で、ルールを設定できる。このときsshの通信が許可される。  
`sudo ufw allow 4242`でポート番号4242の通信が許可される。  
※同様に、`sudo ufw delete 1`でルール1を削除できる。  
`sudo ufw status numbered`でもう一度確認するとルールが追加されているはず。  
22/tcp: IPv4、22/tcp: IPv6の意味。  

## SSH接続の実行
オラクルのVirtualBoxマネージャーを開き、実行中のOSを選択→設定からネットワーク、高度を選択、ポートフォワーディングを選んで、右上の＋ボタンからルールを追加

下表のようにルールを設定したら、Virtual Machine上に戻る。  
![](image.png)  
`sudo systemctl restart ssh`でSSH Serverを再起動する。　  
`sudo service sshd status`でSSHの状態を確認する。  
Ubuntu上の普段使っているターミナルを立ち上げて、`ssh khanadat@127.0.0.1 -p 4242`と入力して、SSH接続する。yesと入力後に過去に設定したユーザーパスワードを入力してアクセスする。  
もし、失敗したときは一度`rm ~/.ssh/known_hosts`で過去の履歴を削除したあとに`ssh khanadat@127.0.0.1`と入力することでリトライができる。  
最後に`exit`とターミナルで入力し、SSH接続を終了する。  
## パスワードポリシーの変更
`sudo apt-get install libpam-pwquality`と入力して、Password Quality Checking Libraryをインストールする。  
`sudo vim /etc/pam.d/common-password`と入力。  
`password              requisite                               pam_pwquality.so retry=3`の行を見つけて、以下のようにルールを追加する。  

`password              requisite                               pam_pwquality.so retry=3　minlen=10 ucredit=-1 lcredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root`  
こうすることで最小の長さを10文字、大文字、小文字、数字が最低1文字ずつ含まれる、同じ文字は最大3文字連続だけ許可、ユーザー名を含むものは禁止、パスワードの新しいものが古いものと少なくとも7文字異なっていなければならない、ルートユーザーにも強制適用となる。  
保存して、vimを抜ける。  

次に`sudo vim /etc/login.defs`を入力する。  
ここで`PASS_MAX_DAYS 9999 PASS_MIN_DAYS 0 PASS_WARN_AGE 7`の項を見つけて、`PASS_MAX_DAYS 30 PASS_MIN 2 PASS_WARN_AGE 7`のように変更する。  
最後に、`reboot`で、設定が適用されるように再起動する。  
## グループの作成
### グループとは
復数のユーザーをまとめてファイルやデバイスへのアクセス権を管理する仕組み。
> - グループ情報は`etc/group`にテキスト形式で記録されて個々で定義されたグループ名やグループ識別番号、メンバー情報をもとにシステムがアクセス制御を行う。
> - GID（グループ識別番号）とは各グループに割り当てられた一位の数値ID。LinuxではこのGIDを参照して、「このファイルはどのグループが所有しているか」「どのグループに属するユーザーに権限を与えるか」を判断する。
> - プライマリグループとはユーザー作成時に必ずひとつ設定されて、`etc/passwd`の4番目フィールドに記録されている。
> - セカンダリグループとはあとから追加できるグループで、最大15個まで登録可能。
> - ユーザーは同時に復数のグループに所属でき、ファイルやコマンド実行時の属するグループによってアクセス権が決まる。
> - Unix/Linuxでは各ファイルに「所有者ユーザー」、「所有グループ」、「その他」の3クラスがあり、それぞれに読み（r）書き（w）実行（x）権限を設定できる。グループ所有権を使うと同じグループメンバーだけに特定の権限を付与できる。
```
#グループ作成
sudo groupadd GROUP_NAME
#ユーザーをグループに追加
sudo adduser ユーザー名 グループ名
#または
sudo usermod -aG グループ名 ユーザー名
#グループ削除
sudo groupdel グループ名
```

`sudo groupadd user42`でグループを追加  
`sudo groupadd evaluating`で評価グループを作成  
最後に、`getent group`でグループが作成されたかどうかを確認する。  

## ユーザーを作成し、グループに配属する。
`cut -d: -f1 /etc/passwd`でローカルユーザーを確認する。  
`sudo adduser new_username`でユーザーを追加して、パスワードを以前設定したルールに則った形で登録する。  
`sudo usermod -aG user42 khanadat`で自分のユーザーを`user42`に登録する。  
`sudo usermod -aG evaluating new_username`で追加したユーザーを`evaluating`グループに追加する。  
`getent group user42`、`getent group evaluating`でグループ情報を確認する。  
`groups khanadat`、`groups new_username`でユーザーがどのグループに属しているかを確認する。  
最後に`chage -l new_username`でパスワードルールがそのユーザーで有効化されているか確認する。  
`su - new_username`でそのユーザーにログインできる。  

## sudoersグループの設定
`cd ~/../../`でディレクトを移動する。  
`cd var/log`に移動  
`makedir sudo`からsudoディレクトリを作成する。（もし既にあれば、それをそのまま使う）  
`cd sudo && touch sudo.log`でsudo.logファイルを作成する。  

### sudo.logファイル
> sudoのコマンド実行ログ（いつ誰が何を実行したか）をこのファイルに書き込む。

`sudo nano /etc/sudoers`でsudoersファイルを編集する。
下記のように`Defaults`設定を追加する。
```
Defaults  env_reset
Defaults  mail_badpass
Defaults  secure_path="/usr/local/sbin:/usr/local/bin:/usr/bin:/sbin:/bin"
Defaults  badpass_message="Password is incorrect. Please try again!"
Defaults  passwd_tries=3
Defaults  logfile="/var/log/sudo/sudo.log"
Defaults  requiretty
```
### 以下説明
> - `env_reset`: sudoを実行するときは、ユーザーの環境変数を一旦まっさらにリセットして、許可されたものだけを引き継ぐ。PATHやLD_LIBRARY_PATHなどを悪意のあるものに書き換えられているときのリスクを防ぐ。リセット後は`env_keep`で明示的に許可した変数だけが残る。
> - `mail_badpass`: sudoのパスワード認証に失敗したときsudoersの設定で指定された管理者アドレス(`mailto`)にメールを送信する。異常なログイン試行を早期に検知することが目的。送信先を変えたい場合は`Default mailto="admin@example.com"のように変更できる。
> - `secure_path=...`: sudo実行時の`$PATH`を固定する。sudoで動くコマンドはこの5つのディレクトリにある実行ファイルだけを探す。他の場所に置いてある危険なコマンドを実行されないようにするための安全策
> - `badpass_message="Pass...`:  パスワードが間違っていたときに表示するメッセージ。デフォルトは"Sorry, try again"
> - `passwd_tries=3`: パスワードを3回間違えたらsudoを打ち切られる。総当たり攻撃を遅らせる効果がある。
> - `logfile="/var...`: いつ・誰が・どんなコマンドを実行したかを全部sudo.logに残す。後でトラブル調査や監査ができるようにする。
> - `requiretty`: sudoは「実際の端末」からしか使えない。cronやスクリプトの中などTTY（接続先端末）がない環境ではsudoを禁止する設定。人間が直接ターミナルで叩いたときだけsudoが動くので、勝手な自動実行を防げる。
> - デフォルトでは`/var/log/sudo-io`にすべてのログの履歴が残ってるのでそのログを`/var/log/sudo`に保存されるように書き換えるのが本問の本質。
> - ちなみに中身はzipファイルのことがあるがそれを解凍して参照するコマンドが`zcat ttyout`コマンドのように打つ。

## Crontabの構成
### Crontabとは
> cronジョブ実行のほかに、編集、リスト、除去を行う。cronジョブとは、cronデーモンによってスケジュールされ、一定の周期で実行されるジョブ。etc/ctontabファイルにタスクの周期などの情報を保存する。  
`apt-get install -y net-tools`でnetstat tools(ネットワークの接続状態や統計を表示するコマンド群)をインストールする。  
`cd /usr/local/bin/`を叩く。  
`touch monitoring.sh`で.shファイルを作成。  
最後に`chmod 777 monitoring.sh`で権限を設定する。  
777は全権限を付与するという意味。（オーナー、グループ、その他に対してread, write, execute)  

## monitoring.shの設定
```
#!/bin/bash
arc=$(uname -a)
pcpu=$(grep "physical id" /proc/cpuinfo | sort | uniq | wc -l)
vcpu=$(grep "^processor" /proc/cpuinfo | wc -l)
fram=$(free -m | awk '$1 == "Mem:" {print $2}')
uram=$(free -m | awk '$1 == "Mem:" {print $3}')
pram=$(free | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}')
fdisk=$(df -BG | grep '^/dev/' | grep -v '/boot$' | awk '{ft += $2} END {print ft}')
udisk=$(df -BM | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} END {print ut}')
pdisk=$(df -BM | grep '^/dev/' | grep -v '/boot$' | awk '{ft += $2} {ut += $3} END {printf("%d"), ut/ft*100}')
cpul=$(top -bn1 | grep '^%Cpu' | awk -F',' '{print $4}' | awk '{printf("%.1f%%"), 100.0 - $1}')
```
### パイプ
> パイプとは `|` のこと。左の出力が入力として入ってきて右に入力として受け流されるイメージ。関数A→（出力A）→|→（入力B）→関数B

### `awk`コマンド
> 表示されている行、列を絞り込んで抽出したり更に計算も可能な関数。
> `awk '$1 == "Mem:" {print $2}`は一列目が"Mem:"の、2列目を抽出するという意味。
> `awk '{ft+=$2} END {print ft}`は2列目をftに足し算していって最後それが終わったら、ftを出力するという意味。Cでお馴染みのprintf関数のようなものも使える（文法は少し異なる点に注意）。

### `grep`コマンド
> 基本的な抽出関数。文字列に合致する行を取り出したり、削除(-vオプション)できたりする。`grep '^/dev'`は行頭に`/dev`があるものをすべて出力する。逆に`grep '/boot$'`は行末に`/boot`があるものをすべて出力する。

### `wc`コマンド
> word countコマンド。そのままの意味で文字数を数える。出力は3つで左から行数、ワード数、バイト数。-lオプションは行数のみ出力する。

### `df`コマンド
> ”disk free"の略でシステム上のファイルシステム（ディスク）の容量と使用状況を確認するコマンド。ファイルシステム名、総容量、使用済み容量、空き容量、使用率、マウント先などが保存されている。
> -BG, BMオプションで、ギビバイト、ミビバイト表示される。

### `top`コマンド
> CPUの使用状況をリアルタイムで確認できる。-bn1オプションで1ページだけを切り抜いて出力する。

### `free`コマンド
> メモリの使用状況を確認できる。-mオプションでメガバイト表示。

## monitoring.shの設定2
```
lb=$(uptime -s)
lvmu=$(if [ $(lsblk | awk '{print $6}' | grep "lvm" | wc -l) -eq 0 ]; then echo no; else echo yes; fi)
ctcp=$(ss -Ht state established | wc -l)
ulog=$(users | wc -w)
ip=$(hostname -I)
mac=$(ip link show | grep "ether" | awk '{print $2}')
cmds=$(journalctl _COMM=sudo | grep COMMAND | wc -l)
wall "  #Architecture: $arc
        #CPU physical: $pcpu
        #vCPU: $vcpu
        #Memory Usage: $uram/${fram}MB ($pram%)
        #Disk Usage: $udisk/${fdisk}Gb ($pdisk%)
        #CPU load: $cpul
        #Last boot: $lb
        #LVM use: $lvmu
        #Connections TCP: $ctcp ESTABLISHED
        #User log: $ulog
        #Network: IP $ip ($mac)
        #Sudo: $cmds cmd"
```
### `who`コマンド
> いまシステムにログインしているユーザー情報を表示するシンプルなLinux/Unix コマンド。主に`var/log/utmp`（ユーザーのログイン記録ファイル）を読んで、一行につき1セッションずつ出力する。おもな表示項目はユーザー名、端末、ログイン日時、リモートホストなど。`-b`オプションで最後のシステム起動時刻（ブート時刻）を表示する。

### `ss`コマンド
> "socket statistics"の略でシステム上のネットワークソケットの状態や統計情報を表示するツール。以前よく使われていた`netstat`の代替としてより高速かつ詳細な情報を得られるようになっている。`-t`オプションでTCPソケット情報のみを得る。`-H`オプションで、ヘッダーを省略する。

### `users`コマンド
> `who`コマンドと違いログイン中のユーザー情報をコンパクトに取得したいときに使える。ユーザー数をカウントするときなどに使える。

### `hostname`コマンド
> Linux/Unix システム上でそのマシンのホスト名を表示したり、一時的に変更したりするためのコマンド。`-I`オプションで、全てのネットワークインターフェースに割り当てられたIPv4アドレスを表示する。

### `ip`コマンド
> Linuxの最新ネットワーク設定ツール「iproute2」パッケージに含まれるコマンドで、ネットワークインターフェースやアドレス、ルーティング、トンネルなどを管理・表示するために使う。従来の`ifconfig`/`route`/`arp`を統合、拡張した強力なツール。`link`オプションでインターフェースの状態を確認、管理できて、`show`オプションで全デバイス、あるいは指定デバイスの情報を表示する。

### `journalctl`コマンド
> systemdが管理する「ジャーナルログを参照、検索するためのコマンド。従来のsyslogやmessagesのようなテキストファイルではなくバイナリ形式で保存されたログを扱う。

### `wall`コマンド
> 現在システムにログインしている全ユーザーの端末（TTY）にメッセージを一斉送信するためのコマンド。主にシステム管理者がメンテナンス通知や緊急告知を行う際に使われる。write allの略。

## monitoring.shの設定3
 続いて、Ubuntuのshellから`ssh khanadat@127.0.0.1 -p 4242`でSSH接続をする。

### `ssh khanadat@127.0.0.1 -p 4242`とは
> ローカルホスト(127.0.0.1)に指定してユーザー名をkhanadatにして認証を行うことを意味する。`-p 4242`でポート番号を4242に指定している。  
ログインできたら、`cd usr/local/bin`を叩く。  
`nano monitoring.sh`で`monitoring.sh`に先程のコマンド全てをそのままペーストする。  
保存ができたらnanoを抜けて、`exit`を叩いてSSHから切断する。  

VirtualBoxに戻り、`sudo visudo`からsudoersファイルを編集する。
`%sudo ALL=(ALL:ALL) ALL`の行を見つけたらその真下に  
`khanadat ALL=(root) NOPASSWD: /usr/local/bin/monitoring.sh`のように追加する。  
> ユーザーkhanadatは、このマシン上で`/usr/local/bin/monitoring.sh`というスクリプトを`root`権限で実行できる。その際パスワード入力は不要であるという意味。  
そしたら、保存して、sudoersファイルを抜ける。  
`sudo reboot`でVirtualBoxを再起動して、`sudo /usr/local/bin/monitoring.sh`から.shファイルを実行テストする。  

`sudo crontab -u root -e`でcrontabを開いて、ルールを追加する。  
crontabの最後の行に`*/10 * * * * /usr/local/bin/monitoring.sh`を追加する。  
これは10分ごとにmonitoring.shを実行するという意味。  

## 質問対策
### 仮想マシンはどう動くのか？その目的は？
> 仮想マシンは既存OS上で仮想的なハードウェア環境をソフトウェアでエミュレートして他のOSを立ち上げる仕組みです。サーバーを立ち上げる、クラウド環境を構築するベアメタル型と開発やテスト環境やや学習、検証用途として利用できるホスト型があります。前者はハードウェアを直接制御するため高速だが、導入が複雑。後者は導入が簡単だがホストOSを介してゲストを動かすので遅い。

### なぜ、DebianまたはCentOSを使ったのか？
> Debianは、依存関係の解決が比較的安定していて、パッケージ管理が簡単だからです。

### DebianとRockyの違いは？
> Debianはコミュニティ主体で開発された非営利プロジェクトでRockyはRed Hut系のCentOSの後継でRocky社が主体として開発された。前者は自由度の高いパッケージ管理ができてよりユーザーフレンドリーなUIを持ち、豊富なライブラリとファイルシステムと環境に恵まれる。もし、企業向けのサーバーなどを考えるときは、企業向けのサポートが充実している後者を選ぶべき。

### AptitudeとAPTの違いは？
> - Aptitudeは高レベルのパッケージ管理ツールで、APTは高レベルのパッケージ管理ツールから利用できる低レベルのパッケージ管理ツール。
> - Aptitudeのほうが賢く使われていないパッケージなどを自動で削除したり依存関係のあるパッケージのインストールのサジェスト機能などを搭載している。
> - APTはコマンドラインで支持されたとおりにだけ動く。

### AppArmorとは？
> Linuxのセキュリティシステムで使われているアプリケーション。AppArmorを使うことで、任意のプログラムに対して意図しないファイルやデバイスのアクセスを阻害したり、サブプロセスに対するセキュリティ制約をかけたりできる。「名前ベースの強制アクセス制御で、LSMを用いて実装されている仕組み」と紹介されることがあり、「セキュリティ設定を対象となるファイルパスをもとに設定」される。そのため、「あるファイルパスやそこから起動するプロセスについて、特定の処理・機能を許可する・しない」をプロファイルとして定義できるのがAppArmorの最大の特徴と言える。対して、よく対比されるSELinuxの場合は「ファイルやプロセスのオブジェクトに対して設定としてラベルを適用する」形を取る。つづいて、強制アクセス制御方式（MAC）とは、管理者でさえもファイルやディレクトリの「パーミッション」の設定に「強制的に影響を受ける」仕組み。これに対して管理者であれば何でもできるような仕組みを人アクセス制御方式（DAC）という。

### パスワードポリシー
> セキュリティを担保するためにある。

### LVMとは
> Logical Volume Managerのこと。復数のハードディスクやパーティションにまたがった記憶領域を一つのボリュームグループにまとめて単一の論理ボリューム（LV）として扱える機能。

### UFWとは
> Uncomplicated FireWallと呼び、どのポート番号を閉じ、どのポート番号を開けるかを管理できるツール。セキュリティ対策にもなる。

### SSHとは
> SSHはSecureなShellのことで遠隔地から暗号化された状態でシェルを操作するメカニズム。

### Cronとは
> スケジュールされたタスクを自動実行したいときに使われる機能。分、時間、日、月、曜日を指定して、ある間隔ごと、ある日にのように設定してタスクを実行できる。
> `cd usr/local/bin`にmonitoring.shが保存されている。
> `sudo crontab -u root -e`でcron jobを編集できる。
> `change script to */1 * * * * sleep 30s && script path`のように設定すれば、毎分30秒に実行できる。

### Evaluationで使えるコマンド一覧
```
sudo ufw status
sudo systemctl status ssh
getent group sudo
getent group user42
sudo adduser new_user
sudo groupadd groupname
sudo usermod -aG groupname username
sudo chage -l username
hostnamectl
hostnamectl set-hostname new_hostname
sudo nano /etc/hosts
lsblk
dpkg -l | grep sudo -
sudo ufw status numbered
sudo ufw allow PORT_NUMBER
sudo ufw delete rule number
ssh khanadat@127.0.0.1 -p 4242
```
