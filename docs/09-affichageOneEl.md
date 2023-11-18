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
