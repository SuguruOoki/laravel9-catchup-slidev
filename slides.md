---
# try also 'default' to start simple
theme: unicorn
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false

layout: intro
logoHeader: './images/techtrain.jpg'
website: 'https://github.com/SuguruOoki'
handle: 'suguru_ohki'
introImage: 'https://avatars.githubusercontent.com/u/16362021?v=4'

---

# Laravel9へ！

<div>
  <h3>2021/02Laravel9がリリースされました!振り返りをしてみましょう。</h3>
</div>

---

# Laravel9での把握すべきこと

---

## 概要

1. PHP 8.0 が最低限必要
2. Symfony6系を利用している
3. ~~次のLTSとなる~~ なりませんでしたw

|   version    | release date              |
| :----------: | :------------------------ |
| Laravel 9.0  | 2022年2月（February 2022） |
| Laravel 10.0 | 2023年2月（February 2023） |
| Laravel 11.0 | 2024年2月（February 2024） |

---

### 機能

|                                               |                                                                                                                                 |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| <kbd>Anonymous Migration</kbd>                | Migration自体にそれぞれ名前をつける必要ない                                                                                     |
| <kbd>New Query Builder Interface</kbd>        | 「Query\Builder」「Eloquent\Builder」「Eloquent\Relation」の勘違いを減らす                                                      |
| <kbd>PHP 8 String Functions</kbd>             | \Illuminate\Support\Strクラスの内部でのstr_contains() 、 str_starts_with() 、およびstr_ends_with()のphpの標準関数の利用が始まる |
| <kbd>From SwiftMailer to Symfony Mailer</kbd> | SymfonyがSwiftMailerを非推奨になったため、その解消                                                                              |

---

# Anonymous Migration

---

そもそもMigration自体にそれぞれ名前をつける必要があるんだっけ？という疑問から生まれた機能。

[該当Issueはこちら](https://github.com/laravel/framework/issues/5899)で参考の書き方は次のとおり。

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;
return new class extends Migration {
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::table('people', function (Blueprint $table)
        {
            $table->string('first_name')->nullable();
        });
    }
};
```

---

### Before

```php {all|7}
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class AddColumnFirstNameInPeopleTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::table('people', function (Blueprint $table) {
            $table->string('first_name')->nullable();
        });
    }
};
```

<arrow v-click="4" x1="400" y1="420" x2="230" y2="240" color="#564" width="3" arrowSize="1" />

---

### After

```php {all|2|7|7}
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::table('people', function (Blueprint $table)
        {
            $table->string('first_name')->nullable();
        });
    }
};
```

<arrow v-click="4" x1="400" y1="420" x2="230" y2="240" color="#564" width="3" arrowSize="1" />

---

# New Query Builder Interface

---

クエリビルダインターフェースが提供されるようになる。これにより補完周りがかなり便利に。
[こちら](https://github.com/laravel/framework/pull/37956) のプルリクエストにもあるように、補完が効いているときに、「Query\Builder」なのか「Eloquent\Builder」なのか「Eloquent\Relation」なのかによって補完が間違っているせいで一度エラーが出たなんて経験がある程度補完を使ったことがある人なら誰しもあるはず。そのエラーが出なくなるらしい。


```php
<?php

return Model::query()
	->whereNotExists(function($query) {
		// $query is a Query\Builder
	})
	->whereHas('relation', function($query) {
		// $query is an Eloquent\Builder
	})
	->with('relation', function($query) {
		// $query is an Eloquent\Relation
	});
```

---

# PHP 8 String Functions

---

\Illuminate\Support\Strクラスの内部での

* str_contains()
* str_starts_with()
* str_ends_with()

のphpの標準関数の利用が始まる。今までphpの標準関数に文字列の中に特定nの文字が含まれるのかなどといった関数がなかったため、Laravelでは、独自のラッパーメソッドを実装していた。今回でその独自実装でなくなり、標準関数の利用が始まる

プルリクエストは [こちら](https://github.com/laravel/framework/pull/38011)

```
Since PHP 8 will be the minimum, Tom Schlick submitted a PR to move to using str_contains(), str_starts_with() and str_ends_with() functions internally in the \Illuminate\Support\Str class.
```

---

# From SwiftMailer to Symfony Mailer

---


プルリクエストは[こちら](https://github.com/laravel/framework/pull/38481)

SymfonyがSwiftMailerを非推奨としたため、Laravel9で、すべてのメールトランスポートにSymfonyMailerを使用するように変更された

```
Symfony deprecated SwiftMailer and Laravel 9 makes the change to use Symfony Mailer for all the mail transports. This does open up a few breaking changes and you can review the PR for all the details. The Laravel 9 upgrade guide will include instructions once it's officially released.
```

---

# アップデートの鬼門

---

## LTSにおけるアップデート(6.x -> 9.x)について

1. Laravel8での変更が鬼門となりそう
2. モデルファクトリが8からクラスベースの書き方に変更があった。
3. しばらくは一旦ヘルパーで捌けるとはいえ、長期的なことを考えると、書き換えておいた方が良さそう

---

## Before

```php
<?php

/** @var \Illuminate\Database\Eloquent\Factory $factory */

use App\Models\User;
use Faker\Generator as Faker;
use Illuminate\Support\Str;

$factory->define(User::class, function (Faker $faker) {
    return [
        'name' => $faker->name,
        'email' => $faker->unique()->safeEmail,
        'email_verified_at' => now(),
        'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
        'remember_token' => Str::random(10),
    ];
});
```

---

## After

```php
use Illuminate\Database\Eloquent\Factories\Factory;

class UserFactory extends Factory
{
    protected $model = User::class;
    public function definition()
    {
        return [
            'name' => $this->faker->name,
            'email' => $this->faker->unique()->safeEmail,
            'email_verified_at' => now(),
            'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
            'remember_token' => Str::random(10),
        ];
    }
}
```


1. HasFactoryのtraitをModelに入れる必要がある
2. ファクトリにネームスペースが付くため、ネームスペースをcomposer.jsonに追加

---

## Rectorを使って乗り越えられるかもしれない。

---

## Rector とは

https://github.com/rectorphp/rector

![This is an image](images/rector.png)

---

[カオナビさんの素晴らしいアドベントカレンダー](https://zenn.dev/naopusyu/articles/cc68a0aa827bca)

---

layout: center
class: text-center

---

# Learn More

[Documentations](https://sli.dev) · [GitHub](https://github.com/slidevjs/slidev) · [Showcases](https://sli.dev/showcases.html)
