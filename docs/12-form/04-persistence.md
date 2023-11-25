# 關於資料持久化

我們在上一點中看到，一旦表單通過驗證，我們就可以持久化實體，然後刷新(`flush`)它。

為此，我們使用了 `$em` 變量。

這個變量其實是以參數的形式直接注入控制器的。

為了簡要解釋這種注入是如何運作的，我們需要再次談談 Symfony 中的依賴注入系統。

例如，為了向控制器注入服務，我們可以使用**類型提示**。

使用類型提示時，Symfony 會在**服務容器**中尋找對應的服務（類別）。

因此，如果找到了服務，它就能在我們需要的地方注入一個服務實例。

然而，可以透過類型提示注入的服務必須是可**自動組裝**的。

:::note 自動組裝

簡而言之，透過 Symfony 的**自動組裝**所使用的**自動組裝服務**可以在控制器的方法或類別的建構子中進行類型提示。

在我們的應用程式中，我們在 src/中創建的所有類別都註冊為服務，並進行了**自動組裝**：我們可以在控制器或類別的建構函數參數中進行類型提示。

:::

因此，要考慮的問題是：為了能夠與我的資料庫通信，我可以在控制器參數中進行什麼類型的類別提示？

使用 Symfony 控制台，我們可以探索自動組裝的服務。

在這裡，我們想要一個 `entity manager` 實體管理器。

我們將使用命令 `debug:autowiring`。 可以向其傳遞一個搜尋參數，以便顯示對應的一個或多個類型：

```bash
php bin/console debug:autowiring entity
```

作為輸出，Symfony 會顯示 ：

```bash
Autowirable Types
=================

 The following classes & interfaces can be used as type-hints when autowiring:
 (only showing classes/interfaces matching entity)

 EntityManager interface
 Doctrine\ORM\EntityManagerInterface (doctrine.orm.default_entity_manager)
 Doctrine\ORM\EntityManagerInterface $defaultEntityManager (doctrine.orm.default_entity_manager)
```

因此，我們將能夠對 Doctrine 套件的 `EntityManagerInterface` 介面進行類型提示：

```php
public function index(
  Request $request,
  EntityManagerInterface $em
): Response {
  $newsletter = new Newsletter();
  $form = $this->createForm(NewsletterType::class, $newsletter);

  $form->handleRequest($request);

  if ($form->isSubmitted() && $form->isValid()) {
    $em->persist($newsletter);
    $em->flush();
  }
  //...
}
```

我們順便找到了另一個類型提示 `Request`。

:::info 自動組裝
自動組裝的作用是識別類型提示並為我們提供相應的服務。 這樣，就能夠輕鬆地在應用程式中使用服務。
:::

> 在這個模組中，我們不會詳細討論為什麼我們必須注入一個介面而不是一個類別給實體管理器。

在我們的應用程式中， 預設配置使我們的所有服務都可以自動組裝。 這就是為什麼我們可以直接將我們的儲存庫注入到我們的控制器中，而無需手動配置任何內容。

透過讀取類型提示，Symfony 會尋找是否存在一個服務，其 ID 是這個完全限定的類別名稱（FQCN）。 由於我們的服務的預設 ID 是它們的 FQCN，因此我們無需配置任何內容即可注入我們的服務。
