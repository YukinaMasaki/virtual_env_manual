# 環境構築手順書 

## バージョン一覧
|OS、言語 etc...|バージョン|
|:------|-----:|
|CantOS|7|
|Mysql|5.7|
|Nginx|1.19.4|
|PHP|7.3|
|Laravel|6.2|

## 構築手順  
### 仮想環境構築〜ゲストOSへログインするまで〜
___
#### VirtualBoxのインストール
下記のサイトからdmgファイルをダウンロード後、インストール  
*Virtual Boxはver6.0.14、OS X hosts を選択  
[VirtualBox公式](https://www.virtualbox.org/wiki/Download_Old_Builds_6_0)  

以下のコマンドを実行してVirtualBoxのウィンドウが表示されるか確認 
``` shell
$ virtualbox
```
#### Vagrantのインストール
```shell
$ brew cask install vagrant
$ vagrant -v  // バージョン確認
```
#### vagrant boxのダウンロード
LinuxのCentOSのバージョン7のbox名 centos/7 を指定して実行  
```shell
$ vagrant box add centos/7  
→ Enter your choice: 3  // 選択肢が表示されるので、virtualbox　を選択
```
下記の表示が出たら成功
``` 
Successfully added box 'centos/7' (v1902.01) for 'virtualbox'!
```
#### Vagrantの作業ディレクトリを用意する
自分の作業用ディレクトリ下にvagrant_test という名前でディレクトリを作成  
```shell
$ mkdir vagrant_test  // ディレクトリ作成  
$ cd vagrant_test  // 作成したディレクトリに移動  
$ vagrant init centos/7  // vagrant boxを使用する  
```
【実行後下記のようになればOK】  
```
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```
#### Vagrantfileの編集
```shell
$ vi Vagrantfile
```
①、②、③の#を外してコメントイン、③はさらに編集
```
変更点①  
config.vm.network "forwarded_port", guest: 80, host: 8080
変更点②  
config.vm.network "private_network", ip: "192.168.33.19"  
変更点③  
config.vm.synced_folder "../data", "/vagrant_data"
↓ 以下に編集  
config.vm.synced_folder "./", "/vagrant", type:"virtualbox" 
```
#### Vagrant プラグインのインストール
```shell
$ vagrant plugin install vagrant-vbguest  
```
 Saharaを導入する場合は下記のサイトを参照  
[VagrantにSaharaを導入](https://qiita.com/sudachi808/items/09cbd3dd1f5c25c23eaf)

#### Vagrantを使用してゲストOSの起動
```shell
$ vagrant up
```
エラーなど状況確認は`vagrant status`コマンド
#### ゲストOSへのログイン
```shell
$ vagrant ssh  // [vagrant@localhost ~]$の表示だと成功
```
>vagrant sshコマンドは vagrant ssh-config コマンドを実行して表示される種々の情報をもとに ssh というコマンドを内部的に実行してログインしています。  
\
`vagrant ssh`ではなく `ssh` コマンドを使用してログインできます。  
現在ゲストOSにログイン中の方は、exit コマンドでログアウトしてください。  
下記のコマンドを `vagrant ssh-config`の実行結果として表示される情報に書き換えて実行  
`$ ssh vagrant@127.0.0.1 -i /Users/xxxx/.vagrant/machines/default/virtualbox -p 2222`  
[vagrant@localhost ~]$  // [vagrant@localhost ~]$の表示だと成功

### 仮想環境構築〜開発に必要なソフトウェアなどをインストール〜
___
#### パッケージをインストール
```shell
$ sudo yum -y groupinstall "development tools"
```
#### PHPのインストール (PHP Ver7.3)
```shell
$ sudo yum -y install epel-release wget
$ sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
$ sudo rpm -Uvh remi-release-7.rpm
$ sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip
$ php -v 
```
[CentOSにPHP7.3をインストールする](https://www.suzu6.net/posts/152-centos7-php-73/)  
#### composerのインストール
```shell
$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
$ php composer-setup.php
$ php -r "unlink('composer-setup.php');"

// どのディレクトリにいてもcomposerコマンドを使用を使用できるようfileの移動を行います  
$ sudo mv composer.phar /usr/local/bin/composer
$ composer -v
```
#### Laravelのインストール (Laravel Ver6.０以上)
現在ゲストOSにログインしている人は一旦`exit`コマンドを実行してログアウト  
```shell
$ cd vagrant_test  // vagrant_testディレクトリ下にインストール  
$ composer create-project laravel/laravel --prefer-dist laravel_app 6.0
$ php artisan -v  // Laravel Ver6.2 で大丈夫

$ php artisan serve   
```
【ブラウザでの確認　http://127.0.0.1:8000】
#### Nginxのインストール  
`$ sudo vi /etc/yum.repos.d/nginx.repo`

書き込み内容↓ 
``` 
[nginx]  
name=nginx repo  
baseurl=http://nginx.org/packages/mainline/centos/\$releasever/\$basearch/  
gpgcheck=0  
enabled=1  
```
書き終えたら保存して、以下のコマンドを実行しNginxのインストールを実行 
``` shell
$ sudo yum install -y nginx 
$ nginx -v
```
#### ファイヤーウォールの起動
```
$ sudo systemctl start firewalld.service
$ sudo firewall-cmd --add-service=http --zone=public --permanent
$ sudo firewall-cmd --reload  // 新たに追加を行ったのでそれをファイヤーウォールに反映させるコマンドも合わせて実行 
```
#### Nginx起動
```shell
$ sudo systemctl start nginx 
```
【ブラウザ確認　http://192.168.33.19】  

**Laravelのwelcomeページが表示されず、 Forbidden 403 エラー出たら、、**  
SELinuxの設定を変更　「SELinux コンテキスト」の不一致によりエラーが出ているので、SELinuxを無効化  

`$ sudo vi /etc/selinux/config`
```
This file controls the state of SELinux on the system.  
 SELINUX= can take one of these three values:  
 enforcing - SELinux security policy is enforced.  
 permissive - SELinux prints warnings instead of enforcing.  
 disabled - No SELinux policy is loaded.  
SELINUX=enforcing  

こちらの記述を下記のように書き換えて、保存  
SELINUX=disabled  
```
設定を反映させるためにゲストOSを再起動する必要がある  
```shell
$ exit 
$ vagrant reload  
$ vagrant ssh  // リロードが完了したら再度ゲストOSにログイン  
$ sudo systemctl start nginx
```
【ブラウザ確認　http://192.168.33.19】
#### データベースのインストール、起動 (Mysql Ver5.7)
```shell
$ sudo wget http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm  
$ sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm  
$ sudo yum install -y mysql-community-server  
$ mysql --version  // versionの確認ができたら、インストール完了  
$ sudo systemctl start mysqld  // Mysql起動 
```
MySQLの設定ファイルを変更 (シンプルなパスワードに初期設定できるように)
```shell
$ sudo vi /etc/my.cnf
```
下記の一行を追加
```
validate-password=OFF　
```
passwordを調べ、接続しpassswordの再設定を行う
```shell
$ sudo cat /var/log/mysqld.log | grep 'temporary password' 
→　2017-01-01T00:00:00.000000Z 1 [Note] A temporary password is generated for root@localhost: パスワード  
$ mysql -u root -p  
// 先ほど確認したパスワード入力  
$ mysql > set password = "新たなpassword"; (パスワード設定) 
```  

#### データベース、テーブルの作成
データベースの作成
```shell
$ mysql > create database laravel_app;  
$ mysql > quit  
```
Laravel側でDBを使用するための記述　Laravelディレクトリ下の `.env` を編集  
```
DB_DATABASE=laravel_app  
DB_USERNAME=root          
DB_PASSWORD=新しく作成したパスワード  
```
テーブルの作成  
```shell
$ cd laravel_app 
$ php artisan migrate
``` 
新しいマイグレーションファイルの作成
```shell
$ php artisan make:migration create_todos_table --create=todos  
```
database/migrations/に yyyy_mm_dd_xxxxxx_create_todos_table.phpファイルがあるので編集　
```blade
public function up()  
 {  
    　Schema::create('todos', function (Blueprint $table) {  
        　　$table->increments('id');  
          　　$table->string('content');  /* 追加 */  
          　　$table->timestamps();  
        　});  
  } 
```
保存後、下記のコマンド実行
```shell
$ php artisan migrate
```
DBに初期データの投入
```shell
$ php artisan make:seeder TodosTableSeeder  
```
database/seeds/TodosTableSeeder を編集  
```blade
<?php  
use Illuminate\Database\Seeder;  
use Carbon\Carbon; // 追記  

class TodosTableSeeder extends Seeder  
{  
    　省略  
    　public function run()  
    　{  
      　　// ここから追記  
        　　DB::table('todos')->truncate();  
        　　DB::table('todos')->insert([  
            　　　[  
                　　　　'content'    => 'Laravel Lessonを終わらせる',  
                　　　　'created_at' => Carbon::create(2018, 1, 1),  
                　　　　'updated_at' => Carbon::create(2018, 1, 4),  
            　　　],  
            　　　[  
                　　　　'content'    => 'レビューに向けて理解を深める',  
                　　　　'created_at' => Carbon::create(2018, 2, 1),  
                　　　　'updated_at' => Carbon::create(2018, 2, 5),  
            　　　],  
        　　　]);  
        　　　// ここまで追記  
    　}  
}
```
database/seeds/DatabaseSeeder.php を編集   
```blade
 public function run()  
    　{  
       　　 // $this->call(UsersTableSeeder::class);  
       　　　 $this->call(TodosTableSeeder::class); // 追加  
    　}  
```
DBに反映させる
```shell
$ php artisan db:seed  

→ Seeding: TodosTableSeeder  // 表示されたら反映してる  
```

### Laravel 〜Application作成〜 
___
##### Controllerの作成
`$ php artisan make:controller TodoController --resource`  
##### ルーティング機能
route/web.php 編集  
```blade
Route::get('/', function () {  
    　return view('welcome');  
});  
Route::resource('todo', 'TodoController');  // 追記  
```
##### Viewの作成
```shell
$ mkdir resources/views/layouts resources/views/todo
``` 
上記のコマンドで2つのディレクトリ作成
- テンプレートになるfileが格納されるディレクトリ  
- 各ページのfileが格納されるディレクトリ  

`$ touch resources/views/layouts/app.blade.php` ファイル作成後、編集  
```blade
<!DOCTYPE html>  
<html lang="ja">  
<head>  
  　<meta charset="utf-8">  
  　<meta http-equiv="X-UA-Compatible" content="IE=edge">  
  　<meta name="viewport" content="width=device-width, initial-scale=1">  
  　<title>Laravel</title>  
  　<link href="{{ asset('/css/app.css') }}" rel="stylesheet">  
  　<!-- Fonts -->  
  　<link href='//fonts.googleapis.com/css?family=Roboto:400,300' rel='stylesheet' type='text/css'>  
  　<link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.7.2/css/all.css" integrity="sha384-fnmOCqbTlWIlj8LyTjo7mOUStjsKC4pOpQbqyi7RrhN7udi9RwhKkMHpvLbHG9Sr" crossorigin="anonymous">  
</head>  
<body>  
  　<div class="container py-5">  
    　　@yield ('content')    
  　</div>  
  　<!-- Scripts -->  
  　<script src="//cdnjs.cloudflare.com/ajax/libs/jquery/2.1.3/jquery.min.js"></script>  
  　<script src="//cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/3.3.1/js/bootstrap.min.js"></script>  
</body>  
</html>  
```
#### Viewの分割
`$ touch resources/views/todo/index.blade.php` ファイル作成後、編集 
```blade
@extends ('layouts.app')   
@section ('content')   

<h1 class="page-header">ToDo一覧\</h1>  
<p class="text-right">  
  　<a class="btn btn-success" href="/todo/create">ToDoを追加\</a>  
</p>   
<table class="table">  
  　<thead class="thead-light">  
    　　<tr>  
      　　　<th>やること</th>  
      　　　<th>作成日時</th>  
      　　　<th>更新日時</th>  
      　　　<th></th>  
      　　　<th></th>  
    　　</tr>  
  　</thead>  
  　<tbody>  
    　　@foreach ($todos as $todo)  
      　　　<tr>  
        　　　　<td class="align-middle">{{ $todo->content }}</td>  
        　　　　<td class="align-middle">{{ $todo->created_at }}</td>  
        　　　　<td class="align-middle">{{ $todo->updated_at }}</td>  
        　　　　<td>\<a class="btn btn-primary" href="{{ route('todo.edit', $todo->id) }}">編集</a></td>  
        　　　　<td>  
        　　　　  {!! Form::open(['route' => ['todo.destroy', $todo->id], 'method' => 'DELETE']) !!}  
        　　　　　    {!! Form::submit('削除', ['class' => 'btn btn-danger']) !!}  
        　　　　  {!! Form::close() !!}  
        　　　　</td>  
      　　　</tr>  
    　　@endforeach   
  　</tbody>  
</table>  

@endsection 
```

**Viewで使用するFormタグ変更する**  
`$ composer require laravelcollective/html:6.2.0` 

`$ vi config/app.php`  
```blade
'providers' => [  
    　// ...  
    　Collective\Html\HtmlServiceProvider::class, // 追記  
    　// ...  
  ],  
  　'aliases' => [  
    　// ...  
      　　'Form' => Collective\Html\FormFacade::class,  // 追記  
      　　'Html' => Collective\Html\HtmlFacade::class,  // 追記  
    　// ...  
  ],  
```
`$ touch resources/views/todo/create.blade.php resources/views/todo/edit.blade.php` 

`create.blade.php` を編集
```blade
@extends ('layouts.app')  
@section ('content')   

<h2 class="mb-3">ToDo作成</h2>  
<form>  
  　<div class="form-group">  
    　　<input type="text" class="form-control" placeholder="ToDo内容">  
  　</div>  
  　<button type="submit" class="btn btn-success float-right">追加</button>  
</form>  

@endsection
```
`edit.blade.php` を編集
```blade
@extends ('layouts.app')  
@section ('content')  

<h2 class="mb-3">ToDo編集\</h2>  
<form >
  　<div class="form-group">  
    　　<input type="text" class="form-control" placeholder="ToDo内容">  
  　</div>  
  　<button type="submit" class="btn btn-success float-right">更新</button>  
</form>  

@endsection
```
#### Controllerを仕上げていく  
DBへの操作が行えるように Model のfileを作成  

`$ php artisan make:model Todo`  

app以下の`Todo.php`ファイル作成されたので編集  
```blade
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;  

class Todo extends Model  
{  
   　 protected $fillable = ['content']; // 追記  
}  
```
`$ vi app/Http/Controllers/TodoController.php`  
```blade
<?php

namespace App\Http\Controllers;  

use Illuminate\Http\Request;  
use App\Todo;  // 追記  

class TodoController extends Controller  
{  
    　// ここから追記  
    　private $todo;  
\
    　public function __construct(Todo $instanceClass)  
    　{  
        　　$this->todo = $instanceClass;  
    　}  
    　// ここまで追記  
```
**indexメソッド**を編集  
```blade
// 省略  
　public function index()  
    　{  
       　　 return view('todo.index');  
      　}  
// 以下省略
```  
**Createメソッド**を編集
```blade
// 省略  
    　public function create()  
    　{  
        　　return view('todo.create');  // 追記  
    　}  
// 以下省略  
```
**Storeメソッド**を編集  
```blade
// 省略  
  　  public function store(Request $request)  
  　  {  
    　　    // 以下 returnまで追記  
  　　      $input = $request->all();  
  　　      $this->todo->fill($input)->save();  
  　　      return redirect()->to('todo');  
  　  }  
// 以下省略  
```
**Editメソッド**を編集  
```blade
// 省略  
    　public function edit($id)  
    　{  
     　　   $todo = $this->todo->find($id);  // 追記  
      　　  return view('todo.edit', compact('todo'));  // 追記  
    　}  
// 以下省略  
```
**Updateメソッド**を編集  
```blade
// 省略  
   　public function update(Request $request, $id)  
    　{  
      　　  $input = $request->all();  
      　　  $this->todo->find($id)->fill($input)->save();  
      　　  return redirect()->to('todo');  
    　}  
// 以下省略  
```
**Destroyメソッド**を編集  
```blade
// 省略  
   　public function destroy($id)  
    　{  
      　　  $this->todo->find($id)->delete();  
      　　  return redirect()->to('todo');     
    　}  
// 以下省略  
```
### laravel 〜ログイン機能の実装〜
___
```shell
$ composer require laravel/ui:^1.0 --dev  // laravel/uiをインストール  
$ php artisan ui vue --auth  // artisanコマンドを実行  
$ npm install  // npmパッケージをインストール  
$ npm run dev  // インストールしたパッケージをビルドする  
```
[Laravel 6.x 認証　Readouble](https://readouble.com/laravel/6.x/ja/authentication.html)  
[Laravel 6.0 で「make:auth」が利用できなくなったので、対応方法記載します。](https://note.com/koushikagawa/n/n1b5bb4a69514)  
[Laravel6.x以降でログイン機能をインストールする方法](https://blog.capilano-fw.com/?p=4576#laravelui)  
[Laravel 6 $ composer require laravel/uiを実行するとエラーが発生する](https://qiita.com/miriwo/items/767b5f69ffaba85ebf6e)  
[npm run watch時に「sh: 1: cross-env: not found」のエラー発生](https://teratail.com/questions/120295)  

**既存のページに機能を反映** 

`$ vi app/Http/controllers/TodoController.php`
```blade
<?php  

namespace App\Http\Controllers;  
 省略  

　    public function __construct(Todo $instanceClass)  
　    {  
　　        $this->middleware('auth');  // 追記  
　　        $this->todo = $instanceClass;  
　    }  
```
**認証 〜ユーザーとデータの紐付け〜**  

`$ php artisan make:migration add_user_id_to_todos_table --table=todos`

作成したファイルを編集
```blade
<?php  

use Illuminate\Support\Facades\Schema;  
// 省略  
　    public function up()  
　    {  
　　       Schema::table('todos', function (Blueprint $table) {  
　　　            $table->integer('user_id');  // 追記  
　　        });  
　    }  
// 省略   
　    {  
　　        Schema::table('todos', function (Blueprint $table) {  
　　　            $table->dropColumn('user_id');  // 追記  
　　        });  
　    }  
} 
```
`$ php artisan migrate`　【マイグレーションを実行】  

Modelに対して以下のように追記 (Todo.php)
```blade
<?php  
//  省略  
　    protected $fillable = [  
　　        'content',  
　　        'user_id'  // 追記  
　    ];  
　    // ここから  
　    public function getByUserId($id)  
　    {  
　　        return $this->where('user_id', $id)->get();  
　    }  
　    // ここまで追記  
}  
```
保存と編集の処理を追加
```blade
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;  
use App\Todo;  
use Auth;  // 追記  

// 省略  
　    public function index()  
　    {  
　　        $todos = $this->todo->getByUserId(Auth::id());  // 追記  
　　        return view('todo.index', compact('todos'));  
　    }  
　    // 省略  
　    public function store(Request $request)
　    {
　　        $input = $request->all();
　　        $input['user_id'] = Auth::id();  // 追記
　　        $this->todo->fill($input)->save();
　　        return redirect()->to('todo');
 　   }  
　    // 省略  
}
```

### Laravelを動かすための設定変更
___
**Nginxの設定ファイルを編集**

`$ sudo vi /etc/nginx/conf.d/default.conf`
```blade
server {  
　　  listen       80;  
  server_name  192.168.33.19; # Vagranfileでコメントを外した箇所のipアドレスを記述してください。  
    
　  root /vagrant/laravel_app/public; # 追記  
　  index  index.html index.htm index.php; # 追記  
  
　  #charset koi8-r;  
　  #access_log  /var/log/nginx/host.access.log  main;  
  
　  location / {  
　　      #root   /usr/share/nginx/html; # コメントアウト  
　　      #index  index.html index.htm;  # コメントアウト  
　　      try_files $uri $uri/ /index.php$is_args$args;  # 追記  
　  }  
  
  # 省略  
  
  # 該当箇所のコメントを解除し、必要な箇所には変更を加える  
  # 下記は root を除いたlocation { } までのコメントが解除されていることを確認してください。  
  
　  location ~ .php$ {  
　　  #    root           html;  
　　      fastcgi_pass   127.0.0.1:9000;  
　　      fastcgi_index  index.php;  
　　      fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更  
　　      include        fastcgi_params;  
　  }  
  
  # 省略
```

**php-fpmの設定ファイルの編集**

`sudo vi /etc/php-fpm.d/www.conf`    
```
;24行目近辺  
user = apache  
↓ 以下に編集  
user = nginx  

group = apache  
↓ 以下に編集  
group = nginx
```
**Permissionの設定**  
```shell
$ cd /vagrant/laravel_app 
$ sudo chmod -R 777 storage  
```
【Permission】とエラーが出たら`ls -la` を実行して操作権限を確認  

操作権限を変更しても反映されなかったら下記のサイトを参考にする  
[Vagrantの共有ディレクトリでchmodしてもパーミッションが変更されない。](https://mrkmyki.com/vagrant%E3%81%AE%E5%85%B1%E6%9C%89%E3%83%87%E3%82%A3%E3%83%AC%E3%82%AF%E3%83%88%E3%83%AA%E3%81%A7chmod%E3%81%97%E3%81%A6%E3%82%82%E3%83%91%E3%83%BC%E3%83%9F%E3%83%83%E3%82%B7%E3%83%A7%E3%83%B3)

#### 動作確認 (作成、更新、削除)
```shell
$ sudo systemctl restart nginx (再起動)   
$ sudo systemctl start php-fpm  
```
【ブラウザ確認　http://192.168.33.19】  

確認できたら終了。

## 所感
大変でした。
特に同じように作成しても違うところでエラーが起こり、驚きました。