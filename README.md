# AWS Cloud9 開発環境に Ruby と Rails をインストールする

# 概要

AWS Cloud9 開発環境を作成すると、Ruby が使える状態になっていますが、最新バージョンではない場合があります。
そこで、rbenv をインストールし、Ruby の最新バージョンをインストールします。
そのあと Ruby on Rails の最新バージョンもインストールします。

また、Cloud9 で新規ファイルを作成する手順、Git の基本的な使い方、Bitbucket に保存する手順も説明します。

# Cloud9 開発環境の作成

新しく Cloud9 開発環境を作成します。

* 「Environment type」は「Create a new instance for environment (EC2)」を選択します。
* 「Instance type」は「t2.micro (1 GiB RAM + 1 vCPU)」を選択します。
* 「Platform」は「Ubuntu Server 18.04 LTS」を選択します。

少し時間がかかるので待ちます。

# rvm を無効化

rvm という Ruby 管理ツールが導入されています。
その設定を無効にするため、ファイルを3つ修正します。

## ~/.bash_profile

vi などのエディタを使って ~/.bash_profile を開き、以下の行の行頭に#を挿入します。

修正前

    [[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm" # Load RVM into a shell session *as a function*

修正後

    #[[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm" # Load RVM into a shell session *as a function*

## ~/.bashrc

vi などのエディタを使って ~/.bashrc を開き、以下の2行の行頭に#を挿入します。

修正前

    [[ -s "$HOME/.rvm/environments/default" ]] && source "$HOME/.rvm/environments/default"

    export PATH="$PATH:$HOME/.rvm/bin"

修正後

    #[[ -s "$HOME/.rvm/environments/default" ]] && source "$HOME/.rvm/environments/default"

    #export PATH="$PATH:$HOME/.rvm/bin"

## ~/.profile

vi などのエディタを使って ~/.profile を開き、以下の2行の行頭に#を挿入します。                                       

修正前

    export PATH="$PATH:$HOME/.rvm/bin"
    
    [[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm" # Load RVM into a shell session *as a function*

修正後

    #export PATH="$PATH:$HOME/.rvm/bin"
    
    #[[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm" # Load RVM into a shell session *as a function*

修正が終わったらシェルを起動しなおします。exit を実行してターミナル画面を閉じから、Alt + T を押してターミナル画面を開きます。
ruby -v を実行して Command not found と表示されれば、無効化はうまくいっています。

# rbenv のインストール

rbenv をインストールします。
以下のコマンドを実行します。

    $ git clone https://github.com/rbenv/rbenv.git ~/.rbenv
    $ cd ~/.rbenv && src/configure && make -C src
    $ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
    $ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
    $ git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build

ここまで終わったらシェルを起動しなおします。exit を実行してターミナル画面を閉じから、Alt + T を押してターミナル画面を開きます。

rbenv を実行して以下のようなメッセージが表示されればうまくいっています。

    $ rbenv
    rbenv 1.1.2-30-gc879cb0
    Usage: rbenv <command> [<args>]
    
    Some useful rbenv commands are:
       commands    List all available rbenv commands
       local       Set or show the local application-specific Ruby version
       global      Set or show the global Ruby version
       shell       Set or show the shell-specific Ruby version
       install     Install a Ruby version using ruby-build
       uninstall   Uninstall a specific Ruby version
       rehash      Rehash rbenv shims (run this after installing executables)
       version     Show the current Ruby version and its origin
       versions    List installed Ruby versions
       which       Display the full path to an executable
       whence      List all Ruby versions that contain the given executable
    
    See `rbenv help <command>' for information on a specific command.
    For full documentation, see: https://github.com/rbenv/rbenv#readme
    ubuntu:~/environment $ 

# Ruby のインストール

rbenv を使ってRubyをインストールします。
今回はバージョン 2.7.1 をインストールします。

以下のコマンドを実行します。

    $ rbenv install 2.7.1

Ruby のソースコードをダウンロードしてコンパイルするため、少し時間がかかります。
インストールできたら、ruby コマンドが実行できるようにします。

以下のコマンドを実行します。

    $ rbenv global 2.7.1

ruby コマンドが実行できるかどうか確認します。
以下のコマンドを実行して、Ruby のバージョンが表示されたらうまくいっています。

    $ ruby -v
    ruby 2.7.1p83 (2020-03-31 revision a0c7c23c9c) [x86_64-linux]    

# Ruby on Rails のインストール

Ruby on Rails をインストールします。
以下のコマンドを実行します。

    $ gem install rails

rails コマンドが実行できるかどうか確認します。

以下のコマンドを実行して、Rails のバージョンが表示されたらうまくいっています。

    $ rails -v
    Rails 6.0.3.1

Rails バージョン 6 から Webpacker というミドルウェアを利用する構成になりました。
Webpacker をインストールするためには、Node.js と、Node.js の環境で利用するパッケージマネージャ Yarn が必要になります。
Node.js はインストールされています。
Yarn をインストールします。

以下のコマンドを実行します。

    $ curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
    $ echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
    $ sudo apt update
    $ sudo apt -y install yarn

# Rails アプリのサンプル作成

Rails アプリのサンプルを作成して、動作確認します。

Rails アプリを新規作成するコマンドは「rails new <アプリ名>」です。
rails-test というアプリを作成してみます。

以下のコマンドを実行します。

    $ rails new rails-test

rails-test というディレクトリが作成され、その中にひながたとなるファイルが自動生成されます。
このアプリ内に Webpacker をインストールします。

rails-test ディレクトリに移動して、「rails webpacker:install」コマンドを実行します。
以下のコマンドを実行します。

    $ cd rails-test
    $ rails webpacker:install

# Rails アプリの起動と Cloud9 のプレビュー機能

この状態でアプリの起動を確認することができます。
rails-test ディレクトリで「rails server」コマンドを実行します。

    $ rails server

なお rails server は rails s という省略形が使えます。 

    $ rails s

Cloud9 には起動したアプリをプレビューする機能が備わっています。

Cloud9 のメニューバーに「Preview」があるのでクリックすると、「Preview Running Application」という選択肢があるのでクリックします。

「Blocked host: ～」というエラー画面が表示されます。
これは Rails 6 から装備されたセキュリティ機構が働いているためです。
Cloud9 のプレビューからのアクセスを許可する設定を追加する必要があります。
config/environments/development.rb の中に「config.hosts << ".amazonaws.com"」という設定を追加します。

ウィンドウ左側の rails-test ディレクトリの一覧表示の中の config/environments/development.rb ダブルクリックして開きます。
この設定ファイルの中身は、

    Rails.application.configure do
      # 設定内容
    end

という構造になっています。
今回の設定を追加する場所は、end の直前でよいと思います。

「config.hosts << ".amazonaws.com"」を development.rb の end の直前に以下のような感じで追加します。

    Rails.application.configure do
      # 設定内容
      config.hosts << ".amazonaws.com"
    end

rails-test を起動した状態だと思いますので、Ctrl-C を押して一度終了し、再度 rails server を実行してください。

「Preview」から「Preview Running Application」をクリックすると、「接続が拒否されました」という画面が出ると思いますが、
ここで右上にある「Pop Out into ther Next Window」というボタンをクリックします。

「Yay! You're on Rails!」という画面が表示されたら成功です。

確認できたらプレビューのために開いたウィンドウを閉じます。

rails server を起動したターミナルで Ctrl-C を押して終了します。

# Ruby プログラムの作成

新しい Ruby プログラムを作成してみます。

FizzBuzz プログラムを作ってみます。

Cloud9 のメニューバーの「File」をクリックすると「New File」という選択肢があるのでクリックします。

「Untitled 1」というウィンドウが開きます。

「puts "fizzbuzz.rb"」と入力します。

メニューバーの File から Save をクリックします。

Filename という欄にファイル名を入力します。ここでは fizzbuzz.rb とします。

保存場所を決めます。fizzbuzz というフォルダを作って保存することにします。

Folder という欄に「/fizzbuzz」と入力します。

Save をクリックします。

左側のディレクトリツリーに「fizzbuzz」というディレクトリが追加され、その中に fizzbuzz.rb というファイルが作成されます。

# Git で管理

fizzbuzz.rb を Git で管理します。

ターミナルの画面で、fizzbuzz ディレクトリに移動します。

    $ cd ~/environment/fizzbuzz

git init コマンドを実行します。

    $ git init

これで fizzbuzz ディレクトリを Git で管理できるようになります。

fizzbuzz.rb を git 管理下に入れるため、git add コマンドを実行します。

    $ git add fizzbuzz.rb

fizzbuzz.rb が git 管理下に入ったかどうか確認するため、git status コマンドを実行します。

    $ git status
    On branch master
    
    No commits yet
    
    Changes to be committed:
      (use "git rm --cached <file>..." to unstage)
    
            new file:   fizzbuzz.rb
    
    $ 

「new file:   fizzbuzz.rb」と表示されたらうまくいっています。

git 管理下に入ったので、保存します。git commit コマンドを実行します。

    $ git commit -m "first commit"

今後 fizzbuzz.rb ファイルを修正したら、

    $ git add fizzbuzz.rb
    $ git commit -m "(何かメッセージ)"

のコマンドを繰り返して変更を保存していきます。

# Bitbucket に保存

git コマンドは Cloud9 開発環境で実行しているので、ファイルは Cloud9 開発環境の中に保存されています。

これらのファイルは Cloud9 開発環境を削除すると同時に失われます。

そこで外部のソースコード管理サービスに保存することにします。

ここでは Bitbucket を利用します。

## Bitbucket に新しいリポジトリを作成

ブラウザで Bitbucket の画面を開きます。

「リポジトリ」というボタンをクリックします。

「Create repository」というボタンをクリックします。

「プロジェクト」欄で新しいプロジェクト名(なんでも構いません)を入力します。

「リポジトリ名」に「fizzbuzz」と入力します。

「リポジトリの作成」をクリックします。

## ローカルリポジトリの内容をプッシュ

「バケツに何かを入れましょう」という画面が表示されます。

「ローカルの Git リポジトリを Bitbucket に移行しましょう」という手順を実行します。

ステップ2のところに「git remote add origin ～」というコマンドが書かれていると思います。

そのコマンドが、

    $ git remote add origin https://ワークスペースID@bitbucket.org/ワークスペースID/fizzbuzz.git

となっていれば、それをコピーしておきます。

もしコマンドが

    $ git remote add origin https://git@bitbucket.org:ワークスペースID/fizzbuzz.git

という形式の場合は、以下の手順でワークスペースIDを確認してください。

Bitbucket の画面の左下に「Your profile and settings」というアイコンがありますので、それをクリックします。

「RECENT WORKSPACES」という一覧が出ますので、自分の名前もしくはユーザー名をクリックします。

メニューに「設定」が表示されますのでクリックします。

「Workspace settings」という画面が表示されます。この中に「Workspace ID」という項目があります。
ここに書かれているのがワークスペースIDです。

Cloud9 のターミナルに戻り、fizzbuzz ディレクトリに移動して、

    $ git remote add origin https://ワークスペースID@bitbucket.org/ワークスペースID/fizzbuzz.git

というコマンドを実行します。

続いて git push -u origin master コマンドを実行します。

    $ git push -u origin master

「Password for 'https://ワークスペースID@bitbucket.org/ワークスペースID/fizzbuzz.git': 」というメッセージが出ますので、Bitbucket のパスワードを入力します。

Bitbucket のブラウザ画面に戻って fizzbuzz リポジトリの画面をリロードすると、
Cloud9 に保存していた fizzbuzz.rb ファイルが登録されていることを確認できます。
ファイル名をクリックしてファイルの内容を表示して、自分が作ったファイルであることを確認します。

今後 fizzbuzz.rb ファイルを修正して Cloud9 に commit したら、

    $ git push -u origin master

を実行して Bitbucket に保存していきます。
