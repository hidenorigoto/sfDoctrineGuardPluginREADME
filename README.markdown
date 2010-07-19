# sfGuardDoctrine plugin (for symfony 1.3) #

`sfDoctrineGuardPlugin` は、symfony の標準的なセキュリティ機能を使ってサイトの認
証と権限を管理できる symfony のプラグインです。

このプラグインにはカスタマイズ可能なモデル (ユーザーオブジェクト、グループオブジェ
クト、パーミッションオブジェクト)、モジュール (バックエンド、フロントエンド) があ
り、すぐに symfony アプリケーションをセキュアにできます。

## インストール ##

  * パッケージからプラグインをインストールします

        symfony plugin:install sfDoctrineGuardPlugin

  * または、Subversion からチェックアウトしてプラグインをインストールできます
  
        svn co http//svn.symfony-project.com/plugins/sfDoctrineGuardPlugin/trunk plugins/sfDoctrineGuardPlugin

  * 次に、`config/ProjectConfiguration.class.php` でプラグインを有効化します
  
        [php]
        class ProjectConfiguration extends sfProjectConfiguration
        {
          public function setup()
          {
            $this->enablePlugins(array(
              'sfDoctrinePlugin', 
              'sfDoctrineGuardPlugin',
              '...'
            ));
          }
        }

  * モデルを再構築します

        symfony doctrine:build-model
        symfony doctrine:build-sql

  * 次のコマンドでデータベースのテーブルを更新します (既存のデーブルを削除して、
    再作成します):

        symfony doctrine:insert-sql

    または、上記のすべてを次のコマンド 1 つで実行します

        symfony doctrine-build-all-reload frontend

    もしくは、`data/sql/plugins.sfGuardAuth.lib.model.schema.sql` に生成された
    SQL 文を使って新しいテーブルのみを作成できます。

  * デフォルトのフィクスチャーを読み込みます (この手順はオプションで、スーパー管
    理者ユーザーを作成します)

        mkdir data/fixtures/
        cp plugins/sfDoctrineGuardPlugin/data/fixtures/fixtures.yml.sample data/fixtures/sfGuard.yml

        symfony doctrine:data-load frontend # frontend は適切なアプリケーション名に置き換えてください

  * `settings.yml` で実際に使うモジュールを有効化します (オプション)
    * 管理用アプリケーション:  sfGuardUser、sfGuardGroup、sfGuardPermission を有
      効化できます

              all:
                .settings:
                  enabled_modules:      [default, sfGuardGroup, sfGuardUser, sfGuardPermission]

    * フロントエンドアプリケーション: sfGuardAuth を有効化できます

              all:
                .settings:
                  enabled_modules:      [default, sfGuardAuth]

  * キャッシュをクリアします

        symfony cc

  * オプションで、"Remember Me" フィルターを `filters.yml` のセキュリ
    ティフィルターの上に追加します:

        [yml]
        remember_me:
          class: sfGuardRememberMeFilter

        security: ~

### アプリケーションをセキュアにする ###

symfony アプリケーションをセキュアにするには:

  * `settings.yml` で `sfGuardAuth` モジュールを有効にします

        all:
          .settings:
            enabled_modules: [..., sfGuardAuth]

  * `settings.yml` で、デフォルトのログインモジュール、セキュアモジュールを変更
    します

        login_module:           sfGuardAuth
        login_action:           signin
        
        secure_module:          sfGuardAuth
        secure_action:          secure

  * `myUser.class.php` で親クラスを変更します

        class myUser extends sfGuardSecurityUser
        {
        }

  * `routing.yml` で次のようなルートを追加します (これはオプションです)

        sf_guard_signin:
          url:   /login
          param: { module: sfGuardAuth, action: signin }
        
        sf_guard_signout:
          url:   /logout
          param: { module: sfGuardAuth, action: signout }
        
        sf_guard_password:
          url:   /request_password
          param: { module: sfGuardAuth, action: password }

    各ルートの `url` パラメーターはカスタマイズできます。
    注意: `@homepage` という名前のルーティングは必須です。サインアウト時にこのル
          ートが使われます。

    これらのルートは、`sfGuardAuth` モジュールが有効になっている場合は、
    `routing.yml` に定義していなくても自動的に追加されます。
    ただし、`app.yml` 設定ファイルで次のように `sf_guard_plugin_routes_register`
    を false に設定すると、自動追加を無効にできます:

        all:
          sf_guard_plugin:
            routes_register: false

  * 個別のモジュール、またはアプリケーション全体の `security.yml` でセキュアにし
    ます

        default:
          is_secure: true

  * これで完了です。セキュアなページにアクセスしようとすると、ログインページへリ
    ダイレクトされます。
    デフォルトのフィクスチャーファイルをロードした場合は、ユーザー名 `admin`、パ
    スワード `admin` でログインしてみてください。

## ユーザー、パーミッション、グループの管理 ##

ユーザー、パーミッション、グループを管理できるように、`sfDoctrineGuardPlugin` で
はバックエンドアプリケーションに統合可能な 3 つのモジュールを用意しています。
これらのモジュールは symfony の Admin ジェネレーターで自動生成されたものです。

  * `settings.yml` でモジュールを有効化します

        all:
          .settings:
            enabled_modules: [..., sfGuardGroup, sfGuardPermission, sfGuardUser]

  * 次のようにデフォルトのルートでモジュールにアクセスできます:

    http://www.example.com/backend.php/sfGuardUser

## sfGuardAuth モジュールのテンプレートのカスタマイズ ##

デフォルトでは、`sfGuardAuth` モジュールはとてもシンプルな次の 2 つのテンプレー
トを使います:

  * `signinSuccess.php`
  * `secureSuccess.php`

これらのテンプレートをカスタマイズする場合は:

  * アプリケーション内に `sfGuardAuth` モジュールを作成します (`init-module` タ
    スクを使ってはいけません。直接 `sfGuardAuth` ディレクトリのみを作成してください)

  * `sfGuardAuth/templates` ディレクトリに、カスタマイズしたいテンプレートと同じ
    名前のテンプレートファイルを作成します

  * これで、デフォルトのテンプレートではなく、新しく作成したテンプレートファイル
    を使ってレンダリングされます

## `sfGuardAuth` モジュールのアクションのカスタマイズ ##

sfGuardAuth のメソッドをカスタマイズ、または追加したい場合は:

  * アプリケーションに `sfGuardAuth` モジュール用のディレクトリを作成します
    (ディレクトリのみを作成します)

  * モジュールの `actions` ディレクトリに `actions.class.php` ファイルを作成し、
    `BasesfGuardAuthActions` クラスを継承するようにします。
    (`BasesfGuardAuthActions` は symfony でオートロードされないので、インクルー
    ドしてください)

        [php]
        require_once(sfConfig::get('sf_plugins_dir').'/sfDoctrineGuardPlugin/modules/sfGuardAuth/lib/BasesfGuardAuthActions.class.php');
    
        class sfGuardAuthActions extends BasesfGuardAuthActions
        {
          public function executeNewAction()
          {
            return $this->renderText('This is a new sfGuardAuth action.');
          }
        }

## `sfGuardSecurityUser` クラス ##

このクラスは symfony の `sfBasicSecurityUser` クラスを継承し、symfony アプリケー
ションで `user` オブジェクトとして使われます。(インストールの手順で `myUser` の
基底クラスを変更しました)

このオブジェクトにアクセスするには、アクションでは `$this->getUser()`、テンプ
レートでは `$sf_user` を使います。

`sfGuardSecurityUser` にはいくつかの追加のメソッドがあります:

  * `signIn()` および `signOut()` メソッド
  * `getGuardUser()` メソッドは、`sfGuardUser` オブジェクトを返します
  * `sfGuardUser` オブジェクトに直接アクセスする一連のプロキシメソッド

たとえば、現在のユーザー名を取得するには次のようにします:

    $this->getUser()->getGuardUser()->getUsername()

    // プロキシメソッドを使うと
    $this->getUser()->getUsername()

## スーパー管理者フラグ ##

`sfDoctrineGuardPlugin` にはスーパー管理者という概念があります。スーパー管理者に
設定されているユーザーの場合は、すべての権限チェックがバイパスされます。

スーパー管理者フラグはウェブからは設定できず、データベースで直接変更するか、次の
タスクを使います:

    symfony guard:promote admin

## バリデーター ##

`sfDoctrineGuardPlugin` に用意されているバリデーター `sfGuardUserValidator` は、
自分のモジュールで使うことができます。

このバリデーターは、`sfGuardAuth` におけるユーザー名とパスワードの検証、およびユ
ーザーの自動サインインに使われています。

## 外部メソッドでユーザーのパスワードをチェックする ##

LDAP サーバーや .htaccess ファイルを使っているためデータベースにパスワードを保存
したくない場合や、別のテーブルにパスワードを保存したい場合、独自の呼び出し可能な
`checkPassword` (static メソッド、または関数) を用意して `app.yml` に設定します:

    all:
      sf_guard_plugin:
        check_password_callable: [MyLDAPClass, checkPassword]

symfony が `$this->getUser()->checkPassword()` メソッドを呼び出す時に、設
定したメソッドまたは関数を呼び出します。関数は 2 つの引数をとります。1 つ目はユ
ーザー名で、2 つ目はパスワードです。関数からは true または false を返すようにし
ます。関数のテンプレートは次のようになります:

    [php]
    function checkLDAPPassword($username, $password)
    {
      $user = LDAP::getUser($username);
      if ($user->checkPassword($password))
      {
        return true;
      }
      else
      {
        return false;
      }
    }

## パスワード保存用のアルゴリズムを変更する ##

デフォルトでは、パスワードは `sha1()` ハッシュで保存されます。呼び出し可能なメソ
ッドまたは関数を `app.yml` することで、任意のアルゴリズムに変更できます:

    all:
      sf_guard_plugin:
        algorithm_callable: [MyCryptoClass, MyCryptoMethod]

または:

    all:
      sf_guard_plugin:
        algorithm_callable: md5

アルゴリズムはユーザーごとに保存されるため、後からアルゴリズムを変更する場合に既
存のユーザーのパスワードを再生成する必要はありません。

## "Remember Me" Cookie の名前や有効期限を変更する ##

デフォルトでは、"Remember Me" 機能により `sfRemember` という名前の
Cookie が 15 日間保存されます。これは `app.yml` で変更できます:

    all:
      sf_guard_plugin:
         remember_key_expiration_age:  2592000   # 秒単位で 30 日
         remember_cookie_name:         myAppRememberMe

## `sfGuardAuth` のリダイレクト処理をカスタマイズする ##

ユーザーがログインに成功した後、ユーザーのプロフィールページへリダイレクトしたり、
ログアウトサイトを定義できます。

`app.yml` でリダイレクトの設定を変更します:

    all:
      sf_guard_plugin:
        success_signin_url:      @my_route?param=value # デフォルトではリファラーが使われます
        success_signout_url:     module/action         # デフォルトではリファラーが使われます