# 表單的建立和管理

## 表單的創建

一旦我們有了表示表單結構的 `NewsletterType` 類，我們就可以在控制器中建立表單：

```php
// 在控制器方法中
$newsletter = new Newsletter();
$form = $this->createForm(NewsletterType::class, $newsletter);
```

`createForm` 方法可以在任何繼承自 `AbstractController` 抽象類別的類別的方法中使用。

該方法的目的是建立一個空實體以及與該實體連接的表單。

透過 `createForm` 方法，我們可以指定要檢索的表單類型，以及要連結到該表單的實體。

## 管理表單

建立表單後，我們就可以管理輸入的任何資料：

```php
$newsletter = new Newsletter();
$form = $this->createForm(NewsletterType::class, $newsletter);

+$form->handleRequest($request);
```

為了管理表單，我們將執行 handleRequest 方法，並將**傳入的請求**作為參數傳遞給它。

:::note 類型提示
為了取得接收到的請求，我們可以在控制器參數中對 Symfony 的 `HttpFoundation` 元件中的請求(`Request`)類型進行鍵入提示。
:::

此 `handleRequest` 方法將自行探索 POST 變數的內容。 如果找到與表單結構相對應的信息，它就能檢索到這些信息，並將其直接映射到實體上！
