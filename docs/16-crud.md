# 自動化 CRUD

:::info 提示
CRUD = Create/Read/Update/Delete
:::

使用 Maker，我們可以為一個實體產生完整的管理介面：

```bash
php bin/console make:crud
```

在執行此命令時，控制台會要求輸入相關實體的名稱，然後它將產生一個完整的控制器類，包含用於建立、檢視、修改和刪除資料的控制器方法。因此，我們可以為我們的實 `Article` 產生一個 CRUD，所有必要的檔案將會被建立。

## 控制器、表單和視圖

查看 Maker 建立的文件，我們會發現一個控制器類別包含 5 個方法：
產生的控制器類別包括以下方法：

-   index：文章列表
-   new：新文章
-   show：文章詳情
-   edit：編輯文章
-   delete：刪除文章

總體而言，在所有控制器中，我們可以理解發生了什麼。 在 `Form` 資料夾中，我們可以找到用於文章的表單定義。

最後，在 `templates` 資料夾中，我們可以找到每個控制器的所有視圖，以及我們想透過 CRUD 執行的每個操作。

我們將重點放在兩點：控制器類別中的 `Route` 屬性，以及同一類別中 `delete` (刪除)方法中的 CSRF 令牌。

## 類別上的 `Route` 屬性

我們看到 CRUD 類別上方的 `Route` 屬性，很快就會明白，類別上方的 Route 屬性是用來為類別中的所有控制器定義**前綴**的。

```bash
#[Route('/article/crud')]
class ArticleCrudController extends AbstractController
{
}
```

因此，在每個控制器（類別的每個方法）中，我們將只使用該前綴後的 URL 部分。

例如，對於 index ：

```php
#[Route('/', name: 'app_article_crud_index', methods: ['GET'])]
```

但如果我們透過命令列查看路由以查找關聯的路由：

```bash
php bin/console debug:router crud_index
```

然後，在路由資訊中，我們可以找到我們的前綴：

```bash
+--------------+------------------------------------------------------------+
| Property     | Value                                                      |
+--------------+------------------------------------------------------------+
| Route Name   | app_article_crud_index                                     |
| Path         | /article/crud/                                             |
...
```

## CSRF 令牌

在 delete 方法中發現了一個新元素：

```php
if ($this->isCsrfTokenValid('delete'.$article->getId(), $request->request->get('_token'))) {
   $articleRepository->remove($article, true);
}
```

如果在刪除之前有令牌驗證，那麼這個令牌必須在之前已經產生過。 這正是防禦跨站請求偽造（CSRF）攻擊的工作原理。

### 原理

CSRF（**C**ross **S**ite **R**equest **F**orgery 跨站請求偽造）攻擊是指從網頁應用程式的外部偽造請求（主要是 POST 請求，例如在表單中）。 例如，即使不透過登入表單，我們仍然可以向表單目標發送 POST 請求。 為此，我們可以使用另一個客戶端（cURL、Postman、另一個 Web 應用程式等），該用戶端會直接定位到表單目標的 URL。

為了避免這種情況，我們要確保 POST 請求事先透過表單：：

-   在顯示表單時，我們產生一個**CSRF 令牌**並將其儲存在會話中。
-   當我們提交表單時，CSRF 令牌將被傳送到伺服器。
-   然後，伺服器可以將其收到的令牌與會話中的令牌進行比較。
-   任何來自其他客戶端的請求都不會經過表單：它們將無法提供令牌，或提供的是無效的令牌。

因此，如果令牌無效或未提供, 我們將防止執行某個操作（這裡是刪除）。

如果重新啟動伺服器，我們可以存取產生的 URL 以查看資料庫中的文章清單。 但是，如果我們想要建立或編輯文章，情況就會變得有點複雜，因為不支持連結到 category 的鏈接。

## 調整表單

Maker 指令為 `Articl` 實體建立了一個表單，但沒有調整應用於實體的每個屬性的類型。

對於 Category 類別，會造成問題。 我們需要更明確地說明如何產生表單。

在文章表單中，我們需要產生一個類別列表，從中選擇文章的類別。

因此，我們可以[在 Symfony 網站的表單類型清單](https://symfony.com/doc/5.4/reference/forms/types.html)中找到適合連結到 `Category` 類別實體的 `Type`。

在 "選擇欄位"（Choice Fields）部分，我們看到了一個 `EntityType`。 點擊後後會發這是一個特別的 `ChoiceType`。 基本的 ChoiceType 代表一個元件，它可以呈現選擇元素(`select`)、複選框(`checkbox`)或單選框(`radio`)輸入，讓我們從一個集合中選擇一個或多個項目。

而 `EntityType` 類型則著重於 Doctrine 實體：它可以從指定實體中檢索物件列表。

因此，我們在 `ArticleType` 中說明我們要使用的類型。 在 `buildForm` 中:

```php
$builder
  ->add('title')
  ->add('date_created')
  ->add('content')
  ->add('category', EntityType::class, [
    'class' => Category::class,
    'choice_label' => 'name'
  ]);
```

如果刷新頁面，就可以了。 此時會出現一個類別列表，我們可以選擇與文章相關聯的類別。

:::info 深入了解
自動產生 CRUD 可以幫助我們節省時間，但也有一些軟體包（在 Symfony 中稱為 **bundles** 捆綁包）可以用來產生非常完整的管理介面。 其中最受歡迎的是 [EasyAdmin](https://symfony.com/bundles/EasyAdminBundle/current/index.html) 包。
:::
