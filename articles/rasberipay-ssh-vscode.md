---
title: "初心者でも安心！Raspberry PiをVSCodeで簡単リモート操作する方法"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Raspberry Pi", "vscode", "ssh", "公開鍵", "暗号鍵"]
published: true
---

# 目的

- ラズパイで直接 vim や vi コマンドを使って、ファイルを書き換えたり、プログラミングを書いたりすることを扱いづらい。
- 自 PC の方がブラウザで検索して、コピーアンドペーストがしやすい

# SSH の公開鍵認証とは

[このサイトが一番わかりやすい](https://qiita.com/angel_p_57/items/2e3f3f8661de32a0d432)

# 実戦

## Raspberry Pi の SSH の port 番号(22 番)を有効にする

### 1. Raspberry Pi のターミナルの開き、以下のコマンドを実行する

```bash
sudo raspi-config
```

### 2. Interface Option を選択する

![open-ssh](/images/rasveripay-ssh-vscode/open-ssh.png)

### 3. ssh を選択して ssh 接続を有効にする

![ssh-enable](/images/rasveripay-ssh-vscode/ssh-enable.png)

## ラズベリーパイの IP アドレスを取得する

```bash
ip addr
```

![ip](https://sozorablog.com/wp-content/uploads/2022/01/raspberry-pi-ip-addr.png)

## 自身の使っている PC に公開鍵、秘密鍵を作成してラズベリーパイに送る

### 1. ssh の保存するためのディレクトリーを作成

```bash
mkdir ~/.ssh/raspberrypi
```

### 2. 鍵の作成（秘密鍵の id_rsa と、公開鍵の id_rsa.pub の作成)

```bash
ssh-keygen -t rsa
```

```bash
#　デフォルト設定では、/Users/username/.ssh/ id_rsaに作成されてしまうので
# /Users/username/.ssh/raspberrypi/id_rasを入力して1.で作成したフォルダーのパスに選択する
zGenerating public/private rsa key pair.
Enter file in which to save the key (/Users/username/.ssh/id_rsa):/Users/username/.ssh/raspberrypi/id_rsa
```

```bash
# この後、いくつかの質問に聞かれるが、Enterを押すだけで良い
Your public key has been saved in /Users/nasunagisa/.ssh/raspberrypi/id_ras.pub
The key fingerprint is:
SHA256:a6fQ58V92Mbp5jBd/er5F+BfviOrcwLlsBlpnJOkBOE nasunagisa@mac.local
The key's randomart image is
```

:::message alert
username は各自で変更すること
:::

### 3. ラズベリーパイに公開鍵を送る

```bash
# ラズベリーパイのipアドレスが192.168.1.10
# usernameは各自で変えること
scp ~/.ssh/raspberrypi/id_rsa.pub username@192.168.1.10:~
```

:::message alert
username は各自で変更すること
:::

## ラズベリーパイの受け取った公開鍵の設定

scp ~/.ssh/raspberrypi/id_rsa.pub username@192.168.1.10:~は port 番号 22 番で接続するが、今の状態だと誰でも接続ができてしまう。そのため、公開鍵をラズパイに送って SSH の公開鍵認証を行なってセキュリティを上げる。

```bash
# sshファイルの作成
sudo mkdir ~/.ssh
```

```bash
#　先ほど作成させたフォルダーに公開鍵を移動
sudo mv ~/id_rsa.pub ~/.ssh/authorized_keys
```

```bash
# sshの権限の変更
sudo chmod 600 ~/.ssh/authorized_keys
sudo chmod 700 ~/.ssh
```

```bash
# sshの設定ファイルを開く
sudo vi /etc/ssh/sshd_config
```

```bash
# 以下の部分がコメントアウトとになっているのでコメントをアウトをはずして保存する

# AuthorizedKeysFile .ssh/authorized_keys .ssh/authorized_keys2
AuthorizedKeysFile .ssh/authorized_keys .ssh/authorized_keys2
```

## 自 PC の vscode から接続を行う

### 1. vscode を開き、拡張機能「Remote - SSH」「Remote Explorer」を入れる

### 2. リモートエクスプローラーから ssh の hosting 設定を行う。

![setting-vscode](/images/rasveripay-ssh-vscode/seggint-vscode.png)

### 3. 以下のファイルが開くのでラズパイのホスト情報を登録し、接続を行う

![host-ssh](/images/rasveripay-ssh-vscode/host.png)

```bash
  Host softwareRaspai
  HostName 192.168.1.10(ラズパイのipアドレス)
  User username(ラズパイで設定しているusername)
  IdentityFile ~/.ssh/raspberrypi/id_rsa
```

# まとめ

- 公開鍵と秘密鍵を理解するのに、とても良い手順になってるのでぜひ一つ一つコマンドを理解しながらやっていただきたいです

# その他

## スクリーンショットの撮り方

1. windous 配列の場合は、Print Screen ボタンを押す
2. home/user/画像　フォルダーに日付時間で保存される。

## raspberrypi のパスワードを忘れた場合

```bash
# raspberrypiのターミナルで以下のコマンドでパスワードを再設定する
sudo passwd pi
```

## ラズパイのスクリーンショットを自 PC にコピーする方法

保存したいファイルを右クリックしてダウンロードを選択する
:::message alert
ドラックアンドドロップでは、変なファイルに変換されてしまう
:::
