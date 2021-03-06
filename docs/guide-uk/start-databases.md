Робота з базами даних
======================

Цей розділ описує, як створити нову сторінку, яка буде відображати назву країни, отриману з таблиці бази даних `country`. Для досягнення цієї мети, вам необхідно буде налаштувати з'єднання з базою даних, отримати необхідні дані з допомогою класу [Active Record](db-active-record.md), створити [подію](structure-controllers.md),
і зобразити все в [представленні](structure-views.md).

З допомогою даного посібника ви дізнаєтесь як:

* Налаштувати з’єднання з базою даних
* Оголосити Active Record класс
* Створювати запити використовуючи Active Record клас
* Відображати дані в представленні з нумерацією сторінок

Зверніть увагу, що для того, щоб закінчити цей розділ, ви повинні мати базові знання і досвід використання баз даних. Зокрема, ви повинні знати, як створювати бази даних, як виконувати SQL-запити , використовуючи клієнти баз даних.


Підготовка бази даних <a name="preparing-database"></a>
--------------------

Для початку створіть базу даних `yii2basic`, з якої і будете надалі отримувати дані. Ви можете використовувати SQLite, MySQL, PostgreSQL, MSSQL або Oracle бази даних, Yii має вбудовану підтримку для багатьох баз даних.

Далі, створіть таблицю `country`, і внесіть декілька прикладів. Можете використати наступний SQL-запит, як приклад:

```sql
CREATE TABLE `country` (
  `code` CHAR(2) NOT NULL PRIMARY KEY,
  `name` CHAR(52) NOT NULL,
  `population` INT(11) NOT NULL DEFAULT '0'
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO `country` VALUES ('AU','Australia',18886000);
INSERT INTO `country` VALUES ('BR','Brazil',170115000);
INSERT INTO `country` VALUES ('CA','Canada',1147000);
INSERT INTO `country` VALUES ('CN','China',1277558000);
INSERT INTO `country` VALUES ('DE','Germany',82164700);
INSERT INTO `country` VALUES ('FR','France',59225700);
INSERT INTO `country` VALUES ('GB','United Kingdom',59623400);
INSERT INTO `country` VALUES ('IN','India',1013662000);
INSERT INTO `country` VALUES ('RU','Russia',146934000);
INSERT INTO `country` VALUES ('US','United States',278357000);
```

На даний момент, у вас є база даних `yii2basic` і таблиця `country` з трьома колонками, що містять десять рядків даних.

Налаштування підключення до БД <a name="configuring-db-connection"></a>
---------------------------

Перш ніж продовжити, переконайтеся, що у вас налаштовано [PDO](http://www.php.net/manual/en/book.pdo.php) PHP розширення і PDO драйвер для вашої БД (наприклад `pdo_mysql` для MySQL). Це є основною вимогою якщо ваш додаток використовує реляційну базу даних.

Згідно того, що у вас встановлено, відкрийте файл `config/db.php` і замініть на коректні дані вашої БД. За замовчуванням, файл містить наступне:

```php
<?php

return [
    'class' => 'yii\db\Connection',
    'dsn' => 'mysql:host=localhost;dbname=yii2basic',
    'username' => 'root',
    'password' => '',
    'charset' => 'utf8',
];
```

Файл конфігурації `config/db.php`  є типовим інструментом на основі файлів [налаштування](concept-configurations.md). Даний файл конфігурації визначає параметри, які необхідні для створення і ініціалізації [[yii\db\Connection]] примірника, через який ви можете робити SQL-запити до основної бази даних.

З’єднання з БД описане вище може бути доступне в коді програми за допомогою виразу `Yii::$app->db`.

> Інформація: Файл конфігурації `config/db.php` буде включений до конфігурації головного додатка `config/web.php`, який визначає як має бути проініційований сам [додаток](structure-applications.md).
  Для отримання додаткової інформації, будь ласка, зверніться до розділу [Налаштування](concept-configurations.md).


Створення Active Record <a name="creating-active-record"></a>
-------------------------

Для відображення і отримання даних з таблиці `country` створіть [Active Record](db-active-record.md) клас з іменем `Country`, і збережіть в файл `models/Country.php`.

```php
<?php

namespace app\models;

use yii\db\ActiveRecord;

class Country extends ActiveRecord
{
}
```

Клас `Country` наслідує [[yii\db\ActiveRecord]]. Вам не потрібно писати ніякого коду всередині нього! Всього лише за допомогою описаного вище коду, 
Yii отримає відповідне ім'я таблиці з імені класу.

> Інформація: Якщо немає прямого співпадіння з імені класу і таблиці, ви можете використати метод [[yii\db\ActiveRecord::tableName()]] щоб задати відповідне ім'я таблиці.

Використовуючи клас `Country`, ви можете легко маніпулювати даними з таблиці `country`, як показано в наступному фрагменті:

```php
use app\models\Country;

// отримати всі рядки з таблиці country і замовити їх по "name"
$countries = Country::find()->orderBy('name')->all();

// отримати рядок, по основному ключу "US"
$country = Country::findOne('US');

// відобразити "United States"
echo $country->name;

// оновити назву країни на "U.S.A." і зберегти в БД
$country->name = 'U.S.A.';
$country->save();
```

> Інформація: Active Record є потужним засобом для доступу і управління даними в базі даних в об'єктно-орієнтованому стилі. Ви можете знайти більш детальну інформацію в розділі [Active Record](db-active-record.md). Крім того, ви також можете взаємодіяти з базою даних, використовуючи для доступу метод передачі даних нижнього рівня під назвою [Data Access Objects](db-dao.md).


Створення події <a name="creating-action"></a>
------------------

Щоб відобразити дані про країну кінцевим користувачам, необхідно створити нову подію. Замість розміщення нової події в контролері `site`, який ви використовували в попередніх розділах, є сенс створити новий контролер спеціально для всіх подій пов’язаних з даними таблиці країн. Створіть навий контролер з іменем `CountryController` і подію `index`, як показано нижще.

```php
<?php

namespace app\controllers;

use yii\web\Controller;
use yii\data\Pagination;
use app\models\Country;

class CountryController extends Controller
{
    public function actionIndex()
    {
        $query = Country::find();

        $pagination = new Pagination([
            'defaultPageSize' => 5,
            'totalCount' => $query->count(),
        ]);

        $countries = $query->orderBy('name')
            ->offset($pagination->offset)
            ->limit($pagination->limit)
            ->all();

        return $this->render('index', [
            'countries' => $countries,
            'pagination' => $pagination,
        ]);
    }
}
```

Збережіть цей код у файл `controllers/CountryController.php`.

Подія `index` викликає метод `Country::find()`. Цей Active Record метод будує запит і отримує всі дані з таблиці `country`.
Щоб обмежити кількість країн, які будуть отримуватись в кожному запиті, сам запит розбивається на сторінки, за допомогою 
[[yii\data\Pagination]] об'єкта. Об’єкт `Pagination` служить двом цілям:

* Встановлює `положення` і `ліміт` для SQL-запиту так, щоб він повертав лише одну сторінку даних за один раз (не більше 5 рядків в сторінці).
* Використовується в цілях відображення пейджера, що складається зі списку кнопок сторінок, як буде описано в наступному підрозділі.

Наприкінці, подія `index` повертає представлення `index` і передає дані по країнах, з розбивкою на сторінки.


Створення представлення <a name="creating-view"></a>
---------------

В директорії `views` створіть спочатку підкаталог `country`. Цей каталог буде використовуватись для всіх представленнь контролера `country`. В каталозі `views/country`, створіть файл з іменем`index.php` 
що містить наступне:

```php
<?php
use yii\helpers\Html;
use yii\widgets\LinkPager;
?>
<h1>Країни</h1>
<ul>
<?php foreach ($countries as $country): ?>
    <li>
        <?= Html::encode("{$country->name} ({$country->code})") ?>:
        <?= $country->population ?>
    </li>
<?php endforeach; ?>
</ul>

<?= LinkPager::widget(['pagination' => $pagination]) ?>
```

Дане представлення містить два розділи для відображення даних по країнам. У першій частині, відображаються дані про країни у вигляді невпорядкованого списку HTML. У другій частині, [[yii\widgets\LinkPager]] віджет з використанням інформації про нумерацію сторінок.
`LinkPager` віджет показується у вигляді списку кнопок. При натисканні на будь-якій з них будуть оновлюватись дані країн у відповідній сторінці.


Спробуєм <a name="trying-it-out"></a>
-------------

Щоб побачити все, що було створено під час роботи, відкрийте в браузері наступний URL:

```
http://hostname/index.php?r=country/index
```

![Перелік країн](../guide/images/start-country-list.png)

Спочатку, ви побачите сторінку з переліком п'яти країн. Нижче країн, ви побачите пейджер з чотирма кнопками. 
Якщо ви натиснете на кнопку "2", ви побачите сторінку ще п'ять країн з бази даних: другу сторінку записів. 
Придивившись більш уважно, ви побачите, що URL в браузері також змінюється на

```
http://hostname/index.php?r=country/index&page=2
```

За лаштунками, [[yii\data\Pagination|Pagination]] надає всю необхідну функціональність для розбивки набору даних на сторінки:

* Спочатку, [[yii\data\Pagination|Pagination]] представляє першу сторінку, яка відображає країни запитом SELECT з умовою `LIMIT 5 OFFSET 0`. В результаті, будуть відображені перші знайдені п'ять країн.
* [[yii\widgets\LinkPager|LinkPager]] віджет відображає кнопки сторінок з URL-адресами створеними за допомогою [[yii\data\Pagination::createUrl()|Pagination]]. URL-адреси будуть містити параметр запиту `page`, який представляє різні номери сторінок.
* Якщо ви натиснете кнопку "2", спрацює новий запит, який буде відправлений на `country/index` з подальшим опрацюванням.
  [[yii\data\Pagination|Pagination]] зчитає параметр `page` з URL-запиту і встановить номер поточної сторінки в 2-ку.
  Таким чином, новий запит буде мати пункт `LIMIT 5 OFFSET 5` і поверне наступні п'ять країн для відображення.


Резюме <a name="summary"></a>
-------

В цьому розділі ви дізналися, як працювати з базою даних. Ви також дізналися, як вибирати і відображати дані на сторінках за допомогою [[yii\data\Pagination]] і [[yii\widgets\LinkPager]].

У наступному розділі ви дізнаєтеся, як використовувати потужний інструмент генерації коду, що називається [Gii](tool-gii.md), який допоможе вам швидко здійснювати деякі часто необхідні функції, такі як Create-Read-Update-Delete (CRUD) операції для роботи з даними в таблицях баз даних. Насправді, код, який ви щойно написали, Yii може автоматично сгенерувати з допомогою функції Gii.
