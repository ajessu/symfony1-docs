generator.yml設定ファイル
=========================

symfonyのadminジェネレーターはモデルクラス用のバックエンドインターフェイスの作成を可能にします。
PropelもしくはDoctrineをORMとして使うことで機能します。

### 作成

adminジェネレーターモジュールは`propel:generate-admin`もしくは`doctrine:generate-admin`タスクによって作成されます:

    $ php symfony propel:generate-admin backend Article

    $ php symfony doctrine:generate-admin backend Article

上記のコマンドは`Article`モデルクラス用の`article` adminジェネレーターモジュールを作成します。

>**NOTE**
>`generator.yml`設定ファイルはPHPファイルとしてキャッシュされます; 
>処理は~`sfGeneratorConfigHandler`~[クラス](#chapter_14_config_handlers_yml)によって自動的に管理されます。

### 設定ファイル

上記のようなモジュールの設定は`apps/backend/modules/model/article/generator.yml`ファイルで行います:

    [yml]
    generator:
      class: sfPropelGenerator
      param:
        # パラメーターの配列

ファイルは2つのメインエントリー: `class`と`param`を格納します。
クラスはPropelに対して`sfPropelGenerator`でDoctrineに対しては`sfDoctrineGenerator`です。

`param`エントリーは生成されたモジュール用の設定オプションを格納します。
`model_class`はこのモジュールにバインドされるモデルクラスを定義し、`theme`オプションはデフォルトで使用するテーマを定義します。

しかしメインのコンフィギュレーションは`config`エントリの下にあります。
これは7つのセクションに分かれます:

  * `actions`: リストとフォームで見つかるアクションのデフォルトコンフィギュレーション
  * `fields`:  フィールド用のデフォルトコンフィギュレーション
  * `list`:    リスト用のコンフィギュレーション
  * `filter`:  フィルター用のコンフィギュレーション
  * `form`:    新規ページ/編集フォーム用のコンフィギュレーション
  * `edit`:    編集ページ用固有のコンフィギュレーション
  * `new`:     新規ページ用固有のコンフィギュレーション

最初に生成されるとき、すべてのセクションは空なものとして定義されます。
adminジェネレーターはすべての実行可能なオプションに対して適切なデフォルトを定義します:

    [yml]
    generator:
      param:
        config:
          actions: ~
          fields:  ~
          list:    ~
          filter:  ~
          form:    ~
          edit:    ~
          new:     ~

この章では`config`エントリーを介してadminジェネレーターをカスタマイズするために使うことのできるすべての利用可能なオプションを説明します。

>**NOTE**
>すべてのオプションはPropelとDoctrineの両方で利用可能で言及していなければ同じ動作をします。

### フィールド

多くのオプションはフィールドのリストを引数として受け取ります。
フィールドは実際のカラム名もしくは仮想的な名前になります。
両方のケースにおいてゲッターはモデルクラスで定義しなければなりません(`get`の後にキャメルケースのフィールド名が続く)。

adminジェネレーターはコンテキストに基づき、フィールドをレンダリングするためのスマートなやり方を知っています。
レンダリングをカスタマイズするには、パーシャルもしくはコンポーネントを作ります。
慣習では、パーシャルにはプレフィックスとしてアンダースコア(`_`)を、コンポーネントにはプレフィックスとしてチルダ(``)をつけます:

    [yml]
    display: [_title, ~content]

上記の例において、`title`フィールドは`title`パーシャルによってレンダリングされ、`content`フィールドは`content`コンポーネントによってレンダリングされます。

adminジェネレーターはパーシャルとコンポーネントに対していくつかのパラメーターを渡します:

  * `new`と`edit`ページに対して:

    * `form`:       現在のモデルオブジェクトに関連づけされているフォーム
    * `attributes`: ウィジェットに適用されるHTML属性の配列

  * `list`ページに対して:

    * `type`:       `list`
    * `MODEL_NAME`: 現在のオブジェクトのインスタンス、`MODEL_NAME`は小文字版のモデルクラスの名前。

`edit`もしくは`new`ページにおいて、2つのカラムレイアウトを維持したい場合(フィールドラベルとウィジェット)、パーシャルもしくはコンポーネントテンプレートは次のテンプレートに従います:

    [php]
    <div class="sf_admin_form_row">
      <label>
        <!-- Field label or content to be displayed in the first column -->
      </label>
      <!-- Field widget or content to be displayed in the second column -->
    </div>

### オブジェクトプレースホルダー

オプションのなかにはモデルオブジェクトプレースホルダーを受け取ることができるものがあります。
プレースホルダーは`%%NAME%%`のパターンに従う文字列です。 
`NAME`の文字列はオブジェクトのゲッターメソッドの名前に使われる妥当な文字列になります(`get`の後にキャメルケースバージョンの`NAME`文字列が続く)。 
たとえば、`%%title%%`は`$article->getTitle()`の値に置き換えられます。
現在のコンテキストに関連するオブジェクトに従ってプレースホルダーの値は実行時に動的に置き換えられます。

>**TIP**
>モデルが別のモデルへの外部キーを持つとき、PropelとDoctrineは関連オブジェクト用のゲッターを定義します。
>ほかのゲッターに関して、オブジェクトを文字列に変換する意味のある`__toString()`メソッドを定義していればプレースホルダーとして使うことができます。

### コンフィギュレーションの継承

adminジェネレーターのコンフィギュレーションはコンフィギュレーションカスケードの原則に基づきます。
継承ルールは次のとおりです:

 * `new`と`edit`は`form`を継承し`form`は`fields`を継承する
 * `list`は`fields`を継承する
 * `filter`は`fields`を継承する

### ~クレデンシャル~

`credential`オプション(下記を参照)を使うユーザークレデンシャルに基づいて、(リストとフォーム上の)adminジェネレーターのアクションを隠すことができます。
しかしながら、リンクもしくはボタンが現れないとしても、違法なアクセスからアクションが適切にセキュアな状態でなければなりません。
adminジェネレーターのクレデンシャル管理機能は表示のみを処理します。

`credential`オプションはlistページのカラムを隠すためにも使うことができます。

### アクションのカスタマイズ

設定が十分ではないとき、生成メソッドをオーバーライドできます:

 | メソッド               | 説明
 | ---------------------- | -------------------------------------
 | `executeIndex()`       | `list`ビューアクション
 | `executeFilter()`      | フィルターを更新する
 | `executeNew()`         | `new`ビューアクション
 | `executeCreate()`      | 新しいレコードを作成する
 | `executeEdit()`        | `edit`ビューアクション
 | `executeUpdate()`      | レコードを更新する
 | `executeDelete()`      | レコードを削除する
 | `executeBatch()`       | バッチアクションを実行する
 | `executeBatchDelete()` | `_delete`バッチアクションを実行する
 | `processForm()`        | Jobフォームを処理する
 | `getFilters()`         | 現在のフィルターを返す
 | `setFilters()`         | フィルターをセットする
 | `getPager()`           | listページャーを返す
 | `getPage()`            | ページャーページを取得する
 | `setPage()`            | ページャーページをセットする
 | `buildCriteria()`      | list用に`Criteria`をビルドする
 | `addSortCriteria()`    | list用にソートの`Criteria`を追加する
 | `getSort()`            | 現在のソートカラムを返す
 | `setSort()`            | 現在のソートカラムをセットする

### テンプレートのカスタマイズ

それぞれの生成テンプレートを上書きできます:

 | テンプレート                 | 説明
 | ---------------------------- | ---------------------------------------------
 | `_assets.php`                | テンプレートに使うCSSとJSをレンダリングする
 | `_filters.php`               | フィルターボックスをレンダリングする
 | `_filters_field.php`         | 単独のフィルターフィールドをレンダリングする
 | `_flashes.php`               | flashメッセージをレンダリングする
 | `_form.php`                  | フォームを表示する
 | `_form_actions.php`          | フォームのアクションを表示する
 | `_form_field.php`            | 単独のフォームフィールドを表示する
 | `_form_fieldset.php`         | フォームのフィールドセットを表示する
 | `_form_footer.php`           | フォームのフッターを表示する
 | `_form_header.php`           | フォームのヘッダーを表示する
 | `_list.php`                  | listを表示する
 | `_list_actions.php`          | listアクションを表示する
 | `_list_batch_actions.php`    | listバッチアクションを表示する
 | `_list_field_boolean.php`    | listの単独のブール型フィールドを表示する
 | `_list_footer.php`           | listのフッターを表示する
 | `_list_header.php`           | listのヘッダーを表示する
 | `_list_td_actions.php`       | 列用のオブジェクトアクションを表示する
 | `_list_td_batch_actions.php` | 列用のチェックボックスを表示する
 | `_list_td_stacked.php`       | 列用のstackedレイアウトを表示する
 | `_list_td_tabular.php`       | list用の単独フィールドを表示する
 | `_list_th_stacked.php`       | ヘッダー用の単独のカラム名を表示する
 | `_list_th_tabular.php`       | ヘッダー用の単独のカラム名を表示する
 | `_pagination.php`            | listページパジネーションを表示する
 | `editSuccess.php`            | `edit`ビューを表示する
 | `indexSuccess.php`           | `list`ビューを表示する
 | `newSuccess.php`             | `new`ビューを表示する

### 外見のカスタマイズ

生成されるテンプレートは多くの`class`と`id`属性を定義するのでadminジェネレーターの外見はとても簡単にカスタマイズできます。

`edit`もしくは`new`ページにおいて、それぞれのフィールドのHTMLコンテナーは次のクラスを持ちます:

  * `sf_admin_form_row`
  * フィールドの型に依存するクラス: `sf_admin_text`、`sf_admin_boolean`、
    `sf_admin_date`、`sf_admin_time`もしくは`sf_admin_foreignkey`。
  * `sf_admin_form_field_COLUMN`。`COLUMN`がカラムの名前です。

`list`ページにおいて、それぞれのフィールドのHTMLコンテナーは次のクラスを持ちます:

  * フィールドの型に依存するクラス: `sf_admin_text`、`sf_admin_boolean`、
    `sf_admin_date`、`sf_admin_time`、もしくは`sf_admin_foreignkey`。
  * `sf_admin_form_field_COLUMN`。`COLUMN`がカラムの名前です。

<div class="pagebreak"></div>

利用可能なコンフィギュレーションオプション
-------------------------------------------

 * [`actions`](#chapter_06_actions)

   * [`name`](#chapter_06_sub_name)
   * [`action`](#chapter_06_sub_action)
   * [`credentials`](#chapter_06_sub_credentials)

 * [`fields`](#chapter_06_fields)

   * [`label`](#chapter_06_sub_label)
   * [`help`](#chapter_06_sub_help)
   * [`attributes`](#chapter_06_sub_attributes)
   * [`credentials`](#chapter_06_sub_credentials)
   * [`renderer`](#chapter_06_sub_renderer)
   * [`renderer_arguments`](#chapter_06_sub_renderer_arguments)
   * [`type`](#chapter_06_sub_type)
   * [`date_format`](#chapter_06_sub_date_format)
 * [`list`](#chapter_06_list)

   * [`title`](#chapter_06_sub_title)
   * [`display`](#chapter_06_sub_display)
   * [`hide`](#chapter_06_sub_hide)
   * [`layout`](#chapter_06_sub_layout)
   * [`params`](#chapter_06_sub_params)
   * [`sort`](#chapter_06_sub_sort)
   * [`max_per_page`](#chapter_06_sub_max_per_page)
   * [`pager_class`](#chapter_06_sub_pager_class)
   * [`batch_actions`](#chapter_06_sub_batch_actions)
   * [`object_actions`](#chapter_06_sub_object_actions)
   * [`actions`](#chapter_06_sub_actions)
   * [`peer_method`](#chapter_06_sub_peer_method)
   * [`peer_count_method`](#chapter_06_sub_peer_count_method)
   * [`table_method`](#chapter_06_sub_table_method)
   * [`table_count_method`](#chapter_06_sub_table_count_method)

 * [`filter`](#chapter_06_filter)

   * [`display`](#chapter_06_sub_display)
   * [`class`](#chapter_06_sub_class)

 * [`form`](#chapter_06_form)

   * [`display`](#chapter_06_sub_display)
   * [`class`](#chapter_06_sub_class)

 * [`edit`](#chapter_06_edit)

   * [`title`](#chapter_06_sub_title)
   * [`actions`](#chapter_06_sub_actions)

 * [`new`](#chapter_06_new)

   * [`title`](#chapter_06_sub_title)
   * [`actions`](#chapter_06_sub_actions)

<div class="pagebreak"></div>

`fields`
--------

`fields`セクションはそれぞれのフィールドに対するデフォルトコンフィギュレーションを定義します。
このコンフィギュレーションはすべてのページに対して定義されページごとにオーバーライドできます(`list`、`filter`、`form`、`edit`と`new`)。

### ~`label`~

*デフォルト*: 人間にわかりやすいカラムの名前

`label`オプションはフィールドに使うラベルを定義します:

    [yml]
    config:
      fields:
        slug: { label: "URL shortcut" }

### ~`help`~

*デフォルト*: なし

`help`オプションはフィールド用に表示するヘルプテキストを定義します。

### ~`attributes`~

*デフォルト*: `array()`

`attributes`オプションはウィジェットに渡すHTML属性を定義します:

    [yml]
    config:
      fields:
        slug: { attributes: { class: foo } }

### ~`credentials`~

*デフォルト*: なし

`credentials`オプションは表示するフィールドに対してユーザーが持たなければならないクレデンシャルを定義します。
クレデンシャルはオブジェクトのリストに対してのみ強制されます。

    [yml]
    config:
      fields:
        slug:      { credentials: [admin] }
        is_online: { credentials: [[admin, moderator]] }

>**NOTE**
>クレデンシャルは`security.yml`設定ファイルと同じルールで定義されます。

### ~`renderer`~

*デフォルト*: なし

`renderer`オプションはフィールドをレンダリングするために呼び出すPHPコールバックを定義します。
定義されていれば、パーシャルもしくはコンポーネントのようにほかのフラグをオーバーライドします。

コールバックは`renderer_arguments`オプションで定義されるフィールドと引数の値で呼び出されます。

### ~`renderer_arguments`~

*デフォルト*: `array()`

`renderer_arguments`オプションはフィールドをレンダリングする際にPHPの`renderer`コールバックに渡す引数を定義します。 
`renderer`オプションが定義される場合のみ使われます。

### ~`type`~

*デフォルト*: バーチャルカラムの`Text`

`type`オプションはカラムの型を定義します。 
デフォルトでは、モデルで定義されるカラムの型を使いますが、バーチャルカラムを作る場合、デフォルトの`Text`型を有効な型の1つでオーバーライドできます:

  * `ForeignKey`
  * `Boolean`
  * `Date`
  * `Time`
  * `Text`
  * `Enum` (Doctrineのみで利用可)

### ~`date_format`~

*デフォルト*: `f`

`date_format`オプションは日付を表示するときに使うフォーマットを定義します。
これは`sfDateFormat`クラスによって認識されるフォーマットになります。
このオプションはフィールドの型が`Date`であるときは使われません。

フォーマットには次のトークンを使うことができます:

 * `G`: Era
 * `y`: year
 * `M`: mon
 * `d`: mday
 * `h`: Hour12
 * `H`: hours
 * `m`: minutes
 * `s`: seconds
 * `E`: wday
 * `D`: yday
 * `F`: DayInMonth
 * `w`: WeekInYear
 * `W`: WeekInMonth
 * `a`: AMPM
 * `k`: HourInDay
 * `K`: HourInAMPM
 * `z`: TimeZone

`actions`
---------

フレームワークはいくつかの組み込みのアクションを定義します。
これらすべてにプレフィックスとしてアンダースコア(`_`)がつけられます。 
それぞれのアクションはこのセクションで説明するオプションでカスタマイズできます。 
同じオプションは`list`、`edit`もしくは`new`エントリでアクションを定義する際に使うことができます。

### ~`name`~

*デフォルト*: アクションのキー

`name`オプションはアクションに使うラベルを定義します。

### ~`action`~

*デフォルト*: アクションの名前に基づいて定義されます。

`action`オプションはプレフィックスの`execute`なしで実行するアクションの名前を定義します。

### ~`credentials`~

*デフォルト*: なし

`credentials`オプションは表示するアクションに対してユーザーが持たなければならないクレデンシャルを定義します。

>**NOTE**
>クレデンシャルは`security.yml`設定ファイルと同じルールで定義されます。

`list`
------

### ~`title`~

*デフォルト*: サフィックスの"List"がつけられた人間にわかりやすいモデルクラスの名前

`title`オプションはlistページのタイトルを定義します。

### ~`display`~

*デフォルト*: すべてのモデルのカラム、スキーマファイルでの定義順

`display`オプションはlistで表示する順序つきカラムの配列を定義します。

カラム前の等号(`=`)は文字列を現在のオブジェクトの`edit`ページに向かうリンクに変換する規約です。

    [yml]
    config:
      list:
        display: [=name, slug]

>**NOTE**
>カラムを隠す`hide`オプションもご覧ください。

### ~`hide`~

*デフォルト*: なし

`hide`オプションはlistから隠すカラムを定義します。
カラムを隠すのに`display`オプションで表示されるカラムを指定するよりも、こちらの方が速いことがあります:

    [php]
    config:
      list:
        hide: [created_at, updated_at]

>**NOTE**
>`display`と`hide`オプションが両方とも提供される場合、`hide`オプションが無視されます。

### ~`layout`~

*デフォルト*: `tabular`

*可能な値*: ~`tabular`~もしくは~`stacked`~

`layout`オプションはlistを表示するのに使うレイアウトを定義します。

`tabular`レイアウトでは、それぞれのカラムの値は独自テーブルのカラムにあります。

`stacked`レイアウトでは、それぞれのオブジェクトは`params`オプション(下記を参照)で定義される単独文字列で表現されます。

>**NOTE**
>`stacked`レイアウトを使う際にも`display`オプションは必要です。
>このオプションはユーザーによってソート可能になるカラムを定義するからです。

### ~`params`~

*デフォルト値*: なし

`params`オプションは`stacked`レイアウトを使用する際に使うHTML文字列のパターンを定義するために使われます。
この文字列はモデルオブジェクトプレースホルダーを含むことができます:

    [yml]
    config:
      list:
        params:  |
          %%title%% written by %%author%% and published on %%published_at%%.

カラムの前の等号(`=`)は文字列を現在のオブジェクトの`edit`ページに向かうリンクに変換する規約です。

### ~`sort`~

*デフォルト値*: なし

`sort`オプションはデフォルトのsortカラムを定義します。
これは2つのコンポーネントから構成されます: カラムの名前とソートの順序: `asc`もしくは`desc`:

    [yml]
    config:
      list:
        sort: [published_at, desc]

### ~`max_per_page`~

*デフォルト値*: `20`

`max_per_page`オプションは1つのページを表示するオブジェクトの最大数を定義します。

### ~`pager_class`~

*デフォルト値*: Propelでは`sfPropelPager`、Doctrineでは`sfDoctrinePager`

`pager_class`オプションはlistを表示する際に使用するページャークラスを定義します。

### ~`batch_actions`~

*デフォルト値*: `{ _delete: ~ }`

`batch_actions`オプションはlistのオブジェクト選択用に実行できるアクションのリストを定義します。

`action`を定義しない場合、adminジェネレーターはプレフィックスが`executeBatch`であるキャメルケース版の名前のメソッドを探します。

実行されるメソッドは`ids`リクエストパラメーターを通して選択されたオブジェクトの主キーを受け取ります。

>**TIP**
>バッチアクションの機能はオプションを空の配列: `{}`にセットすることで無効にできます。

### ~`object_actions`~

*デフォルト値*: `{ _edit: ~, _delete: ~ }`

`object_actions`オプションはlistのそれぞれのオブジェクトで実行可能なアクションのリストを定義します。

`action`を定義しない場合、adminジェネレーターはプレフィックスが`executeList`であるキャメルケース版の名前のメソッドを探します。

>**TIP**
>オブジェクトアクションの機能はオプションを空の配列: `{}`にセットすることで無効にできます。

### ~`actions`~

*デフォルト値*: `{ _new: ~ }`

新しいオブジェクトの作成のように、`actions`オプションはオブジェクトを受け取らないアクションを定義します。

`action`を定義しない場合、adminジェネレーターは`executeListをプレフィックスとするキャメルケースの名前のメソッドを探します。

>**TIP**
>オブジェクトアクション機能はオプションを空の配列: `{}`にセットすることで無効にできます。

### ~`peer_method`~

*デフォルト値*: `doSelect`

`peer_method`オプションはlistで表示するオブジェクトを読み取るために呼び出すメソッドを定義します。

>**CAUTION**
>このオプションはPropelに対してのみ存在します。
>Doctrineに対しては、`table_method`オプションを使います。

### ~`table_method`~

*デフォルト値*: `doSelect`

`table_method`オプションはlistで表示するオブジェクトを読み取るために呼び出すメソッドを定義します。

>**CAUTION**
>このオプションはDoctrineに対してのみ存在します。
>Propelに対しては、`peer_method`オプションを使います。

### ~`peer_count_method`~

*デフォルト値*: `doCount`

`peer_count_method`オプションは現在のフィルター用のオブジェクトの個数を算出するために呼び出すメソッドを定義します。

>**CAUTION**
>このオプションはPropelに対してのみ存在します。
>Doctrineに対しては、`table_count_method`オプションを使います。

### ~`table_count_method`~

*デフォルト値*: `doCount`

`table_count_method`オプションは現在のフィルター用のオブジェクトの個数を算出するために呼び出すメソッドを定義します。

>**CAUTION**
>このオプションはDoctrineに対してのみ存在します。
>Propelに対しては、`peer_count_method`オプションを使います。

`filter`
--------

`filter`セクションはlistページに表示されるフォームをフィルタリングするためのコンフィギュレーションを定義します。

### ~`display`~

*デフォルト値*: 定義の順序で、フィルターフォームクラスで定義されたすべてのフィールド。

`display`オプションは表示するフィールドの順序つきリストを定義します。

>**TIP**
>フィルターフィールドは常にオプションで、表示するフィールドを設定するためにフィルターフォームクラスをオーバーライドする必要はありません。

### ~`class`~

*デフォルト値*: サフィックスが`FormFilter`であるモデルクラスの名前

`class`オプションは`filter`フォームに使うフォームクラスを定義します。

>**TIP**
>フィルタリング機能を完全に除外するには、`class`を`false`にセットします。

`form`
------

`form`セクションは`edit`と`new`セクションのためのフォールバックとしてのみ存在します(最初の継承ルールを参照)。

>**NOTE**
>フォームセクション(`form`、`edit`と`new`)に関して、`label`と`help`オプションはフォームクラスで定義されたものをオーバーライドします。

### ~`display`~

*デフォルト値*: フォームクラスで定義されるすべてのクラス。順序は定義された順序と同じ。

`display`オプションは表示するフィールドの順序つきリストを定義します。

このオプションはフィールドをグループに分類するためにも使うことができます:

    [yml]
    # apps/backend/modules/model/config/generator.yml
    config:
      form:
        display:
          Content: [title, body, author]
          Admin:   [is_published, expires_at]

上記のコンフィギュレーションは2つのグループ(`Content`と`Admin`)を定義します。
それぞれがフォームフィールドのサブセットを含みます。

>**CAUTION**
>モデルフォームで定義されるすべてのフィールドは`display`オプションに存在しなければなりません。
>そうではない場合、予期しないバリデーションエラーになる可能性があります。

### ~`class`~

*デフォルト値*: サフィックスが`Form`であるモデルクラスの名前

`class`オプションは`edit`と`new`ページに使うフォームクラスを定義します。

>**TIP**
>`new`と`edit`セクションの両方で`class`オプションを定義できますが、1つのクラスと条件ロジックを使い違いを考慮するほうがよいです。

`edit`
------

`edit`セクションは`form`セクションと同じオプションを受け取ります。

### ~`title`~

*デフォルト*: サフィックスが"Edit"である人間にわかりやすいモデルクラスの名前

`title`オプションはeditページのタイトルの見出しを定義します。
これはモデルオブジェクトのプレースホルダーを格納できます。

### ~`actions`~

*デフォルト値*: `{ _delete: ~, _list: ~, _save: ~ }`

`actions`オプションはフォームを投稿する際に利用可能なアクションを定義します。

`new`
-----

`new`セクションは`form`セクションと同じオプションを受け取ります。

### ~`title`~

*デフォルト*: サフィックスが"New"である人間にわかりやすいモデルクラスの名前

`title`オプションは新しいページのタイトルを定義します。
これはモデルオブジェクトのプレースホルダーを格納できます。

>**TIP**
>オブジェクトが新しい場合でも、タイトルの一部として出力したいデフォルトの値を格納できます。

### ~`actions`~

*デフォルト値*: `{ _delete: ~, _list: ~, _save: ~, _save_and_add: ~ }`

`actions`オプションはフォームを投稿する際に利用可能なアクションを定義します。