# 從資料庫中檢索和顯示元素

例如，如果我們想要建立一個分類頁面，第一個問題就是：如何檢索正確的分類？

## 第一種方法：將請求注入控制器

Symfony 有一個 `Request` 類，我們可以將其註入到任何控制器中。 一旦有了 `Request` 類別的實例，我們就可以存取請求的各種資訊。

例如，在這裡我們想取得一個 GET `id` 參數：

```php
/**
  * @Route("/category", name="category_item")
  */
public function item(Request $request): Response
{
  $id = $request->query->getInt('id');

  return $this->render('category/item.html.twig', [
    'id' => $id
  ]);
}
```

:::note REQUEST::QUERY
我們使用 query 屬性，它包含請求的所有 GET 參數。

在請求對像中，我們可以檢索許多[其他信息](https://symfony.com/doc/5.4/components/http_foundation.html#accessing-request-data)
:::
因此，一旦我們取得了要顯示的類別的 ID，我們就可以將類別儲存庫(`CategoryRepository`)注入控制器，以取得相關實體：

```php
/**
  * @Route("/category", name="category_item")
  */
public function item(Request $request, CategoryRepository $repo): Response
{
  $id = $request->query->getInt('id');
  $category = $repo->find($id);

  return $this->render('category/item.html.twig', [
    'category' => $category
  ]);
}
```

在這裡，我們新增了一個 URL 參數：id。

但是我們注意到，任何 URL 參數都可以被映射為控制器的參數。 我們還在方法中加入了一個 `int` 類型的參數。

因此，我們不再需要 `Request` 物件來取得作為 URL 參數傳遞的 ID，而可以直接使用我們的 repository 來取得類別。

Symfony 允許我們聲明一個 URL 參數並直接將其作為控制器的參數取得。 對於我們的例子來說，這樣做更好一些，但仍然不夠理想。 例如，在找不到類別時，我們沒有處理 404 錯誤。

Symfony 提供了更簡單、更有效的方法。

## 解決方案 - ParamConverter

實際上，使用 Symfony，我們可以聲明我們的路由，將一個 URL 參數（如`id`）整合進去，然後直接為期望的實體類型進行**類型提示(type-hinter)**，以便將相應的物件注入到我們的控制器中：

```php
/**
 * @Route("/category/{id}", name="category_item")
 */
public function item(Category $category): Response
{
  return $this->render('category/item.html.twig', [
    'category' => $category
  ]);
}
```

在這裡，具體發生了什麼事？

-   Symfony 能夠透過路由器辨識 URL `id` 參數。
-   透過閱讀 `Category` 的類型提示，Symfony 理解我們希望從資料庫中檢索一個元素（`Category`是一個 Doctrine 實體…）
-   因此，它執行 Doctrine 的**ParamConverter**，該轉換器允許從 URL 參數中檢索資料庫中的元素。
-   我們的 URL 參數名為 `id`：Doctrine 會尋找具有此 id 的 `Category`（注意：我們也可以傳遞一個名稱與其他類別欄位相對應的參數，這就是它的工作原理）
-   如果找到記錄，它就會被注入到控制器中。
-   否則，將產生 404 錯誤。

因此，我們大大減少了檢索類別所需的程式碼量，使控制器更清晰，URL 更易讀，並且現在在未找到記錄時產生一個 404 錯誤，這比以前的行為更為嚴謹，以前可能會將 `null` 類別傳遞給我們的模板。

:::info 參數轉換器(ParamConverter)
ParamConverter 用於從 URL 參數中自動擷取元素，並將其自動注入控制器。
:::

:::note 注意
當然，如果元素檢索邏輯比較複雜，就沒有必要使用參數轉換器(ParamConverter)。 完全可以聲明一個 URL 參數，然後使用儲存庫按照特定邏輯檢索記錄
:::
