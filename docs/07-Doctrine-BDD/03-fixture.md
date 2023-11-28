# Fixtures - 測試數據

一旦我們有了實體並創建了資料庫，我們希望能夠向資料庫中插入資料。

在創建新文章的表單和介面之前，我們還可以要求軟體包產生**測試資料**：一組初始記錄將建立在資料庫中，使我們能夠開始工作。

這樣，如果我們取得了這個應用程式的程式碼，我們就不必花時間手動輸入測試資料。

這種類型的資料被稱為 **fixtures**。

## fixtures 軟體包

安裝下列開發依賴項：`orm-fixtures`（這是 Flex 的別名）。

```bash
composer require --dev orm-fixtures
```

:::info symfony 軟體包和別名
`orm-fixtures` 是 `doctrine/doctrine-fixtures-bundle` 軟體包的別名。

:::

Symfony Flex 在安裝過程中**運行的配方**建立了一個 `src/DataFixtures/AppFixtures.php` 文件。

我們將在該文件中建立物件並將其保存到資料庫中。

## 建立 fixtures

最初，我們的測試資料檔案基本上看起來像這樣：

```php
<?php

namespace App\DataFixtures;

use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Persistence\ObjectManager;

class AppFixtures extends Fixture
{
    public function load(ObjectManager $manager): void
    {
        // $product = new Product();
        // $manager->persist($product);

        $manager->flush();
    }
}
```

因此，在類別的`load`方法中，我們要將所有要儲存到資料庫中的實體實例化。

例如，

```php
<?php
//...
public function load(ObjectManager $manager)
{
  $article = new Article();
  $article->setTitle('Mon premier article !')
    ->setDateCreated(new DateTime())
    ->setContent("Le contenu de l'article");

  $manager->flush();
}
//...
```

然後，我們可以使用以下命令產生測試資料庫：

```php
php bin/console doctrine:fixtures:load
# ou plus court
php bin/console d:f:l
```

如果運行該命令，**什麼也不會發生**。 這是完全正常的。 若要在資料庫中建立記錄，必須先**持久化**記錄。

## 實體持久化

在上面的程式碼摘錄中，我們來看看將實體儲存到資料庫的各個步驟 :

-   實例化一個 `Article` 類型的新對象
-   使用流體介面透過 setter 為其各個屬性指派值
-   刷新（`flush`）所做的更改，使其在資料庫中執行

當我們要在資料庫中建立新記錄時，我們要經歷這些步驟。 在這裡，我們缺少了一個最重要的步驟：在使用 flush 驗證變更之前**持久化**我們的專案。

為此，在呼叫 `flush` 方法之前，我們將要求我們的管理器（`ObjectManager` 實例）對實體進行**持久化**。

```php
$article = new Article();
$article->setTitle('Mon premier article !')
  ->setDateCreated(new DateTime())
  ->setContent("Le contenu de l'article");

$manager->persist($article); // <-- ICI
$manager->flush();
```

:::danger 刷新前的持久化
在呼叫 flush 方法之前，持久化階段，即呼叫實體管理器的 `persist` 方法，是必不可少的。 簡而言之，在創建物件後，它**告訴您的管理器您希望它處理此實體**。 由於它還不存在（剛剛在程式碼中手動創建，而不是從現有資料來源中檢索），因此當您呼叫管理器的 `flush` 方法時，它將被持久化，即在資料庫中建立。
:::

:::info flush 方法
呼叫 `flush` 方法可以將您對實體管理器的所有變更**推送**到資料庫。 對於我們的範例，這意味著在要求我們的管理器處理實體之後，將其具體插入到資料庫中，即執行插入所需的 SQL 程式碼。 這使我們能夠向管理器請求多個操作（插入、修改、刪除），然後透過一次呼叫 `flush` 來組合所有請求。 如果我們每次向管理器發出請求都必須執行一次查詢，那就太麻煩了。

:::

:::note 管理器
如果我們在執行 fixture 時切換到追蹤模式，就可以看到 `$manager` 實例實際上是 `DoctrineORM\EntityManager` 類別的實例。 因此，我們有了一個實體管理器，它將幫助我們管理與資料庫的交換

:::

## 產生隨機數據

目前，我們的測試資料每次執行時只會建立一篇文章。

此外，每次執行測試資料時，它們並不會添加到資料庫中，而是會**取代**資料庫中已有的內容。 因此，每次執行測試資料時都會**清空**資料庫，以取得新的資料以便使用。

我們希望的是，能夠自動為我們產生數據，而無需花費大量時間來編寫測試所需的文章。

### Faker 庫

為了實現這一目標，我們將委託這項工作給一個函式庫：[`fakerphp/faker`](https://github.com/FakerPHP/Faker)。

因此，我們將使用以下命令將其新增至開發依賴項：

```bash
composer require --dev fakerphp/faker
```

要使用它，只需按照文件操作即可。 我們將建立一個 `Faker\Generator` 類別的實例，然後用它來詢問名字、姓氏、文字等...

```bash
$faker = \Faker\Factory::create();

$article = new Article();
$article->setTitle($faker->realText(50))
  ->setDateCreated($faker->dateTimeBetween('-2 years'))
  ->setContent($faker->realTextBetween(250, 500));
$manager->persist($article);

$manager->flush();
```

### 生成文章集合

如果我們想產生五十篇隨機文章，那麼我們只需使用一個循環來創建這五十篇文章：

```bash
$faker = \Faker\Factory::create();

for ($i = 0; $i < 50; $i++) {
  $article = new Article();
  $article->setTitle($faker->realText(50))
    ->setDateCreated($faker->dateTimeBetween('-2 years'))
    ->setContent($faker->realTextBetween(250, 500));
  $manager->persist($article);
}

$manager->flush();
```

:::danger 最後執行 FLUSH

我們已經了解了使用 `persist` 方法持久化資料的重要性。

一旦我們請求我們的管理器來處理創建的 50 個實體, 我們在**最後**執行 `flush` 方法，

實際上，我們不想在應用程式和資料庫之間執行往返 50 次，而是希望執行一次，以建立這 50 筆記錄。

:::
