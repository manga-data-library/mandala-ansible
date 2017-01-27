# mandala-ansible

Manga Data Libraryのサーバ群を構築するプロジェクトです。  
本プロジェクトは以下の2つで構成されています。

- Vagrant
- Ansible


# ローカル環境構築手順

CentOS 7の仮想OS上に、Mandalaのローカル開発環境を構築します。  
構築はVagrantとVirtualBoxを用いて行います。

ホストOSには40GB以上のディスク領域が必要です。
- https://atlas.hashicorp.com/centos/boxes/7


## 動作説明

Provisioningで以下のAnsible Playbookが実行されます。  
※ホストOSにAnsibleは不要です。  

    mandala-ansible/ansible/local.yml

実行後に作成されるmandalaの各リポジトリは、以下のディレクトリに配置されます。

    mandala-ansible/repositories


## 実行要件

以下のソフトウェアとVagrantのプラグインをホストOSにインストールしてください。

### ソフトウェア

- Vagrant 1.8.7
- VirtualBox 5.0.X

### Vagrantプラグイン

- vagrant-vbguest
- vagrant-reload

Vagrantプラグインのインストール手順は以下になります。

```
$ vagrant plugin install <プラグイン名>
```


### for Windows Users

ホストOSにWindowsを利用する場合は、追加で以下の作業を行ってください。

#### RSyncのインストール

CygwinでRSyncをインストールしてください。  
Cygwinを利用しない場合は、Chocolateyでのインストールがお奨めです。  
Chocolateyをインストール後、以下のコマンドでインストールできます。

```
> choco install rsync
```

#### 環境変数の追加

環境変数に以下を追加してください。

- ```PATH```に```C:\ProgramData\chocolatey\lib```を追加（;区切り）
- ```CYGWIN```に```nodosfilewarning```を追加（スペース区切り）


## 実行手順

ホストOSのコンソールで、以下のコマンドを実行してください。  

    $ vagrant up

rsyncの自動同期を行う場合は、ゲストOS起動後に以下のコマンドをホストOSで実行してください。

    $ vagrant rsync-auto


### 実行時間

すべてのタスクが完了するまで、約1時間ほどかかります。  


# Ansible Playbook

Mandalaの実行環境を構築する、AnsibleのPlaybookです。

    mandala-ansible/ansible


## 実行要求

Vagrant以外から実行する場合は、実行環境にAnsible 2.2.0+ をインストールしてください。


## 開発環境構築の手動実行方法

開発環境のサーバに```mandala-ansible/ansible```をコピーし、当該ディレクトリ上で以下のコマンドを実行してください。

    ansible-playbook -i local local.yml

※SSHのagent forwardingを使ってgit cloneしているので、GitHubの秘密鍵がssh-addで登録されている必要があります。
vagrant sshしてからansible-playbookコマンドを叩く場合も、ホストOS側で事前にssh-addの実行をお願いします。


## 環境の説明

Playbookの実行により、以下がCentOS 7上に配置されます。

### ミドルウェア

#### Mandalaで利用するもの

- MySQL 5.7
- OpenJDK 1.8.+
- sbt 最新
- git 2.10.0
- ruby 2.3.1
- rails 5
- flyway 4.0.3


#### Mandalaで現在利用していないもの

- Jenkins 2.+
- ant 1.9.7
- SchemaSpy 5.0.0
- nginx 最新
- go 1.7.4
- gRPC(protocol-buffer)
- elasticsearch 5.1+
- kibana 5.1+


### Mandalaのリポジトリ

group_vars/all.ymlで定義されたapplication_dir配下（デフォルト：/home/vagrant/repositories）に、
Mandalaの以下のリポジトリが作成されます。

- [mandala-db-migration](https://github.com/manga-data-library/mandala-db-migration)
- [mandala-master-api](https://github.com/manga-data-library/mandala-master-api)
- [mandala-analyze-twitter](https://github.com/manga-data-library/mandala-analyze-twitter)
- [mandala-tsubuyaki](https://github.com/manga-data-library/mandala-tsubuyaki)
- [mandala-scripts](https://github.com/manga-data-library/mandala-scripts)


### MySQL

開発に利用するDBとして、"mandala"が作成されます。

また、以下のユーザがMySQLに作成されます。

| User   | Password  | Host      |
| :----- | :-------- | :-------- |
| root   | root      | localhost |
| admin  | admin     | localhost, 127.0.0.1, % |

初期設定されるパスワードは、group_vars/all.ymlの
mysql_root_password、mysql_admin_passwordで定義されています。


#### ログファイル

MySQLのログファイルは、以下のディレクトリに配置されます。（group_vars/all.ymlの変数で設定）

    mysql_log_dir: /var/log/mysql


### nginx

#### SSLの秘密鍵、証明書について

Ansibleは、本番環境で利用するSSLの秘密鍵、証明書を作成・配置を行いません。
必要に応じて`{{ nginx_conf_dir }}/ssl`ディレクトリ配下にファイルを配置してください。

開発環境構築用のPlaybookを利用した場合は、
以下の手順で作成された秘密鍵、証明書が上記ディレクトリに配置されます。

```
$ openssl genrsa 2048 > server.key
$ openssl req -new -key server.key > server.csr
（問い合わせに対しては全てEnterのみ）
$ openssl x509 -days 3650 -req -signkey server.key < server.csr > server.crt
```