# PHP & Framework Quality Guideline

**PHP & Framework Quality Guideline DOT Indonesia** merupakan sebuah standar dan panduan bagi backend engineer DOT Indonesia atau vendor yang menggunakan PHP atau framework PHP sebagai server side programming.

Kunjungi [Development Stack & Tools](https://github.com/pt-dot/development-stack-tools) untuk melihat daftar aplikasi dan perangkat development yang dibutuhkan

---

## 1. General Standard

a. Hal-hal berikut ini seharusnya dimasukkan ke dalam list `.gitignore` dan tidak boleh di push ke dalam repository:

+ direktori `vendors`, `node_modules`
+ file upload dari user
+ file `.env`
+ Informasi credential penting atau krusial

b. Gunakan penamaan variable atau method yang singkat & jelas, serta tidak membingungkan.

**good**:

`$user`, `$storeData`, `$debetAccount`

**bad**:

`$aaaa`, `$name1`, `$thisistoloongvariableyoumaynotseeit`

c. Variable atau method menggunakan format `CamelCase`

d. Error message / debug message hanya boleh ditampilkan pada mode `development` atau `staging`. Gunakan **default error message** ketika di mode `production`

e. Tidak meletakkan endpoint atau informasi credential yang bersifat private secara hard code di dalam source code. Contoh:

```php
protected $secretKey = 'ThisIsN0tSuppOs3dToBeHere';

protected $ProdUrl = 'https://someprivateip/api';
```

Manfaatkan file `.env` untuk menyimpan value sensitif tanpa terekspose di dalam source code.

f. Source code tidak mengandung *backdoor* atau shell script yang berbahaya.

g. Semua form harus menggunakan `CSRF Protection`

---

## 2. PHP Coding Standard

a. Harus mengikuti PHP [PSR-1: Basic Coding Standard](http://www.php-fig.org/psr/psr-1/)

b. Harus Mengikuti PHP [PSR-2: Coding Style Guide](http://www.php-fig.org/psr/psr-2/)

PSR-2 overview:
```php
<?php
namespace Vendor\Package;

use FooInterface;
use BarClass as Bar;
use OtherVendor\OtherPackage\BazClass;

class Foo extends Bar implements FooInterface
{
    public function sampleMethod($a, $b = null)
    {
        if ($a === $b) {
            bar();
        } elseif ($a > $b) {
            $foo->bar($arg1);
        } else {
            BazClass::bar($arg2, $arg3);
        }
    }

    final public static function bar()
    {
        // method body
    }
}
```

c. Jika menggunakan autoloading, ikuti standard [PSR-4: Autoloading](http://www.php-fig.org/psr/psr-4/)

d. Agar source code lebih mudah dibaca dan dipahami sertakan `DockBlock` pada fungsi atau attribute. Manfaatkan DockBlock untuk menginformasikan proses, variable, dan output yang digunakan. Acuan penggunaan DockBlock dapat dilihat di [PHPDOC - DOCKBLOCK Basic Syntax](http://docs.phpdoc.org/references/phpdoc/basic-syntax.html)
    
Overview:
```php
<?php

class Foo {
    
    /**
    * @var string
    */
    protected $propertiesOne;

    /**
    * @var string
    */
    protected $propertiesTwo;

    /**
    * This is description of this class
    * this class may use recursion bla bla bla
    *
    * @param string $arg1
    * @param integer $arg2
    * @return array
    */
    public function bar($arg1, $arg2)
    {
        // some code

    }
}
```

e. Tidak boleh ada `Dead Code` saat merge ke branch `master`. `Dead Code` adalah source code (method, attributes, class) yang sudah tidak digunakan akan tetapi masih tersimpan di dalam codebase dan biasanya hanya dinonaktifkan menggunakan `comment`

Overview:

```php
<?php

class Foo {
    
    /**
    * This is description of this class
    * this class may use recursion bla bla bla
    *
    * @param string $arg1
    * @param integer $arg2
    * @return array
    */
    public function bar($arg1, $arg2)
    {
        // some code

    }
    
    //public function deadcode($arg1, $arg2)
    //{
    // some DEAD CODE
    //
    //}
}
```

f. Penggunaan PHP framework yang disarankan adalah [Laravel](https://laravel.com/) atau microframework [Lumen](https://lumen.laravel.com/).

---

# 3. Laravel / Lumen / PHP Framework Engineering Guideline

a. Untuk project yang bersifat longterm disarankan menggunakan framework versi LTS (Long Term Support) atau disesuaikan dengan kebutuhan.

b. Gunakan fitur `Database Transaction` untuk operasi persistence yang menggunakan lebih dari 1 tabel database.

overview:

```php

try {
    DB::beginTransaction();

    // first persistence operation

    // second persistence operation

    // operation success, then commit transaction
    DB::commit();
    return true;
} catch(\Exception $e) {
    // if error happened, rollback transaction
    DB::rollback();
    return false;
}
```

c. Untuk aplikasi yang kompleks, kumpulkan class `Model`, `Controller`, atau `Views` dalam direktori tersendiri sesuai dengan konteks / domain aplikasinya.

simple case:

```bash
├── app
│   ├── Http
│   │   ├── Controller
│   │   │   ├── Authentication
│   │   │   ├── OrderManagement
│   │   │   │   ├── OrderController.php
│   │   │   │   ├── ReportOrderController.php
├── Model
│   ├── User.php
│   ├── Order.php
├── resources
│   ├── views
│   │   ├── dashboard
│   │   ├── administrator
│   │   │   ├── index.blade.php
│   │   │   ├── create.blade.php
│   │   │   ├── edit.blade.php
│   │   │   ├── detail.blade.php
│   │   ├── ordermanagement

```


d. Untuk efisiensi dan efektivitas masa development, gunakan *library* yang sudah umum digunakan dengan kriteria sebagai berikut:

+ library aktif di maintain oleh kontributor / owner
+ Sesuai dengan kebutuhan engineering project
+ Sesuai dengan development stack yang sedang digunakan
+ Penggunaan library harus benar-benar dapat mempermudah proses development

e. Gunakan perintah berikut untuk optimasi saat deployment Laravel terutama di staging & production:

```bash
php artisan route:cache
php artisan config:cache
composer dumpautoload --classmap-authoritative
```
> khusus untuk `route:cache` tidak boleh ada fungsi closure pada file route

f. Manfaatkan fitur `event` untuk fungsi yang tidak dependen dengan hasil outputnya. Misal: kirim email, logging, push notification. [Dokumentasi event Laravel](https://laravel.com/docs/5.6/events)

g. Gunakan `eager loading` untuk optimasi penggunaan relationship di eloquent. Baca implementasi [Eager Loading](https://laravel.com/docs/5.6/eloquent-relationships#eager-loading).

h. Selalu gunakan `migration` untuk pembuatan atau modifikasi skema database saat development baik itu menambah kolom, edit kolom, hapus kolom, atau modifikasi index. Metode ini lebih baik daripada merubah satu file migration lalu menjalankan perintah `php artisan migrate:rollback` lalu `php artisan migrate` lagi. Baca implementasi [migration](https://laravel.com/docs/5.6/migrations)

i. Dalam kasus tertentu pengambilan data menggunakan `Eloquent` akan memakan terlalu banyak resource. Bentuk optimalisasinya bisa dari salah satu cara berikut:
    
+ Ganti `relationship` dengan menggunakan operasi `join` sehingga query cukup dijalankan satu kali.
+ Ganti operasi menggunakan `Query Builder`

j. Gunakan blok `try - catch` untuk handling exception terutama di operasi yang berkaitan dengan I/O seperti database, HTTP request, file, service layer.

overview:

```php
/**
* This is description of this class
*
* @param Request $request
* @return Response
*/
public function register(Request $request)
{
    try {
        $service = $this->applicationService->registerUser($user);
        return response()->json($service);
    } catch(Exception $e) {
        return response()->json(['error' => $e->getMessage()]);
    }
}
```

k. Semua konfigurasi credential dari pihak ketiga harus diset melalui `config/service.php` dan value harus diambil dari file `.env`

overview:

```php
<?php
// config/service.php
return [
    'zenziva'    => [
        'userkey'   => env('ZENZIVA_USER', ''),
        'passkey'   => env('ZENZIVA_PASS', ''),
        'subdomain' => '',
        'masking'   => false,
    ],

    'tcast'      => [
        'user'     => env('TCAST_USER', ''),
        'password' => env('TCAST_PASSWORD', ''),
        'senderid' => env('TCAST_SENDERID', ''),
    ]
];
```

l. Saat production mode pastikan hal berikut:

+ `APP_ENV=production`
+ `APP_DEBUG=false`
+ `APP_KEY` harus di generate ulang menggunakan perintah `php artisan key:generate`

---

## 4. Package & libraries

Berikut merupakan daftar library / package PHP & Laravel yang biasa digunakan dalam proses development.

1. [Laravel Debugbar](https://github.com/barryvdh/laravel-debugbar) - debug & monitor resource laravel
2. [Laravel Flash](https://github.com/laracasts/flash) - Flash message wrapper
3. [Laravel CORS](https://github.com/barryvdh/laravel-cors) - Cross Origin Resource Sharing header
4. [Laravel Excel](https://laravel-excel.maatwebsite.nl/) - Spreadsheet wrapper
5. [Socialite](https://github.com/laravel/socialite) - OAuth authentication untuk Facebook, Twitter, Google, Linkedin, Github, Bitbucket
6. [L5 Repository](https://github.com/andersao/l5-repository) - Laravel 5 repository abstraction
7. [Laravel Horizon](https://horizon.laravel.com/) - Dashboard untuk monitoring Laravel Queue dengan Redis
8. [Codeception](https://codeception.com/) - Testing suite functional, API, Acceptance, Unit, dll
9. [Doctrine DBAL](https://github.com/doctrine/dbal) - Doctrine Database Abstraction Layer untuk support laravel migration
10. [Predis Laravel](https://github.com/nrk/predis) - Redis client untuk PHP
11. [Laravel Sentry](https://sentry.io/for/laravel/) - Laravel error tracking menggunakan [sentry.io](https://sentry.io/for/laravel/)
12. [Laravelcollective HTML](https://laravelcollective.com/docs/master/html) - HTML & form laravel wrapper
13. [Faker](https://github.com/fzaninotto/Faker) - Untuk generate data dummy
14. [Laravel Datatable](https://github.com/yajra/laravel-datatables) - Laravel JQuery datatable library
15. [Laravel Tinker](https://github.com/laravel/tinker) - REPL untuk laravel
16. [Laravel PDF](https://github.com/niklasravnsborg/laravel-pdf), [laravel dompdf](https://github.com/barryvdh/laravel-dompdf) - Wrapper & generate pdf di Laravel
17. [Laravel Self Diagnosis](https://github.com/beyondcode/laravel-self-diagnosis) - self diagnosis test laravel
18. [Intervention Image](http://image.intervention.io/) - PHP image handling & manipulation
19. [JWT auth](https://github.com/tymondesigns/jwt-auth) - JWT wrapper untuk laravel & lumen
20. [Laravel Passport](https://laravel.com/docs/5.6/passport) - Oauth2 Server Laravel
21. [Laravel fractal wrapper](https://github.com/spatie/laravel-fractal) - Data transformasi wrapper laravel
22. [Activity Log](https://github.com/spatie/laravel-activitylog) - Pencatatan aktivitas aplikasi ke dalam log
23. [LDAP Authentication](https://github.com/pt-dot/ldap-auth) - integrasi auth dengan LDAP
24. [Laravel Backup](https://github.com/spatie/laravel-backup) - backup direktori laravel & dump database
25. [Laravel ER Diagram generator](https://github.com/beyondcode/laravel-er-diagram-generator) - Generate Entity Relationship Diagram dari Model Laravel

---

# Kontribusi

Internal engineer silakan berkontribusi untuk membuat guideline ini bisa lebih lengkap dan lebih baik. Caranya:

+ Fork repository ini
+ Buat branch baru di repository hasil fork
+ Edit file readme sesuai dengan kebutuhan lalu commit.
+ Ajukan pull request
+ AVP divisi atau VP of engineering akan melakukan review dan melakukan approval Pull Request.

Jika ada pertanyaan atau permintaan update silakan untuk mengajukan issue di repository terkait.
