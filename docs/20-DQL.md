# DQL 和查詢建構器

有了 Doctrine，我們就有了描述表結構的實體，而儲存庫則為我們提供了與資料庫就實體進行通訊的方法。

## 儲存庫方法

有了基本的儲存庫，我們就可以呼叫 Doctrine 的 `EntityRepository` 類別所提供的方法：

-   `findAll` 用於檢索表中的所有記錄
-   `find` 根據 ID 檢索記錄
-   `findBy` 用於根據特定條件檢索一個集合
-   等等。

這樣做的好處是，我們可以在控制器或服務中對儲存庫進行類型提示，然後輕鬆檢索記錄或記錄集。

例如，對於一個產品集合，如果只想要特價的可見產品，就可以使用控制器中的 `findBy` 方法：

```php
$products = $productRepository->findBy(['visible' => true, 'discount' => true]);
```

但我們可能有更複雜的需求：例如，在主頁控制器中，如果我想要所有可見的特價產品，限購 5 件，按價格遞增排序，並且創建時間不超過一年，我該怎麼辦？

顧名思義, 查詢建構器可以讓我們建立更複雜的查詢。

## DQL：Doctrine 查詢語言

使用 Doctrine，我們可以使用 **DQL**（**D**octrine **Q**uery **L**anguage）來建構查詢。

從 `EntityManagerInterface` 的實例中，可以呼叫 `createQuery` 方法，將其中傳遞 DQL 語法。

讓我們回到上面的範例：我想要所有特價的可見產品，限 5 件，按價格遞增排序，創建時間不超過一年。

在 `ProductRepository` 的方法中，我可以取得實體管理器並建立查詢：

```php
$products = $this->getEntityManager()->createQuery("SELECT p FROM " . Product::class . " p WHERE p.visible = true AND p.discount = true AND p.date_created > DATE_SUB(CURRENT_DATE(), 1, 'YEAR') ORDER BY p.priceVatFree")
  ->setMaxResults(5)
  ->getResult();
```

我們可以立即註意到幾點：

-   DQL 總是使用**實體**而不是表。 這裡討論的是我們程式碼中的類別。
-   如果套用 `WHERE` 子句，我們將再次引用相關的實體。 因此，我們將 `Product` 別名為 `p`。
-   DQL 為我們提供了一些可能有用的函數：例如，我們可以使用 `DATE_ADD` 和 `DATE_SUB` 來執行日期操作，這兩個函數與 MySQL 的 `DATE_ADD` 和 `DATE_SUB` 函數非常相似。
-   我們也有 `CURRENT_DATE()` 函數可用來取得目前日期。
-   如果要套用限制，那麼應該用 `setMaxResults` 方法取代 SQL 的 `LIMIT` 關鍵字。
-   要取得結果，我們使用 `createQuery` 產生的查詢上的 `getResult()` 方法。

DQL 提供了許多可能性，您可以在其[文件](https://www.doctrine-project.org/projects/doctrine-orm/en/2.14/reference/dql-doctrine-query-language.html)中找到更多資訊。

## 查詢建構器

也可以使用另一種完全物件導向的語法來產生查詢。 我們可以從任何儲存庫中使用 `QueryBuilder`，它的作用是在不必直接編寫 DQL 的情況下產生查詢。

讓我們再以產品為例：

```php
$products = $this->createQueryBuilder('p')
  ->andWhere('p.discount = :discount')
  ->andWhere('p.visible = :visible')
  ->andWhere('p.date_created > :last_year')
  ->setParameter('discount', true)
  ->setParameter('visible', true)
  ->setParameter('last_year', (new DateTime())->modify('-1 year'))
  ->orderBy('p.priceVatFree', 'ASC')
  ->setMaxResults(5)
  ->getQuery()
  ->getResult();
```

:::note GETRESULT()
`getResult()` 方法的回傳類型設定為 `mixed`。

我們可以加入 PHPDoc 註解 `@return` 來指示我們回傳一個 `Product` 陣列。 在儲存庫中：`

```php
/**
 * @return Product[]
 */
public function getHomepageProducts(): array
{
  return $this->createQueryBuilder('p')
    ->andWhere('p.discount = :discount')
    ->andWhere('p.visible = :visible')
    ->andWhere('p.date_created > :last_year')
    ->setParameter('discount', true)
    ->setParameter('visible', true)
    ->setParameter('last_year', (new DateTime())->modify('-1 year'))
    ->orderBy('p.priceVatFree', 'ASC')
    ->setMaxResults(5)
    ->getQuery()
    ->getResult();
}
```

## 連接

有時，從多個表中選擇資料可能是有用的。

例如，如果我想顯示一篇文章列表，並且對於每篇文章，顯示它們的類別名稱，那麼我可以修改文章的顯示範本：

```php
> templates/article/_article_card.html.twig
/**
 * @return Product[]
 */
public function getHomepageProducts(): array
{
  return $this->createQueryBuilder('p')
    ->andWhere('p.discount = :discount')
    ->andWhere('p.visible = :visible')
    ->andWhere('p.date_created > :last_year')
    ->setParameter('discount', true)
    ->setParameter('visible', true)
    ->setParameter('last_year', (new DateTime())->modify('-1 year'))
    ->orderBy('p.priceVatFree', 'ASC')
    ->setMaxResults(5)
    ->getQuery()
    ->getResult();
}
```

在控制器中顯示所有文章時，我注入了 `ArticleRepository` 類別並使用 `findAll()` 方法：

```php
$articles = $articleRepository->findAll();
```

在顯示頁面的時候，Doctrine 會在我**需要的時候查詢每個類別**，以便顯示其名稱。

由於有 150 篇文章和 15 個類別，我們最終會收到大量請求。 但是，Doctrine 會進行最佳化：如果某個類別已經被查詢過，它會重複使用，以避免再次查詢。 但是，我們可以透過在查詢文章時直接執行**聯結**來避免這些額外的查詢。 這樣我就可以透過查詢直接準備資料。

要使用 Doctrine 執行聯接，讓我們再次從 "物件 "而不是資料庫 "表 "的角度來思考。

讓我們在 `ArticleRepository` 中建立一個名為 `findAllArticlesWithCategory()` 的方法：

```php
> src/Repository/ArticleRepository.php
/**
 * @return Article[]
 */
public function findAllArticlesWithCategory(): array
{
    return $this->createQueryBuilder('a')
        ->join('a.category', 'c')
        ->addSelect('c')
        ->getQuery()
        ->getResult();
}
```

為了關聯類別，我們使用了 QueryBuilder 的 `join` 方法。 在參數中，我們重新使用了實體 Article 的別名 a，以存取其**屬性**：這裡是 `category`。 然後，我們在第二個參數中為實體 `Category` 指定了一個別名 `c`。

接下來，我們使用了 `addSelect` 方法：該方法用於獲取類別的資訊並將其新增至 MySQL 傳回的回應中。 預設情況下，當我們使用 `createQueryBuilder` 時，`SELECT` 指令中唯一整合的實體是 `Articl`e。 因此，我們告訴 Doctrine 我們希望**添加**類別。

在控制器中，我們現在可以使用 Repository 的 `findAllArticlesWithCategory` 方法：

如果刷新頁面，那麼查詢數量就會減少：類別資訊已經包含在 MySQL 傳回的結果中，因此 Doctrine 可以直接找到它們，而無需執行額外的查詢。

:::note 最佳化
當應用程式出現運行速度減慢或效能問題時，這種類型的最佳化非常有用。 減少請求次數可以加快頁面顯示的處理時間。

然而，並非總是需要直接進行最佳化。 在這種情況下，存在 "預先優化 "的風險，這種優化並無用處，而且會在開發過程中浪費時間。
:::
