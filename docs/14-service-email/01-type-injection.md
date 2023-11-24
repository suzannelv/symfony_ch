# 識別可注入的類型

## 在容器中

使用 Symfony 的應用程式控制台（`php bin/console`），我們可以要求 Symfony 容器顯示與關鍵字 "Mailer" 相關的可用服務：

```bash
php bin/console debug:container Mailer
```

在顯示的清單中，我們可以找到許多包含關鍵字的服務。 首先，讓我們透過輸入其編號來顯示名稱為 "mailer" 的服務的詳細資訊。

我們可以看到相關的類別確實位於 `symfony/mailer` 組件中，因此這可能是我們希望在我們的控制器中註入的服務。

此外，它的服務標識是 `mailer.mailer`。

但是，該服務被標記為不可**自動組裝**。 因此，我們不能直接將其作為**類型提示**注入到我們的控制器中。

## 在自動組裝的類型中

為了知道我們可以使用哪種類型提示，我們還可以使用應用程式控制台並使用以下命令來詢問可用的類型，使用關鍵字 "mailer"：

```bash
php bin/console debug:autowiring Mailer
```

在顯示的清單中，我們找到以下元素：

```bash
Autowirable Types
=================

 The following classes & interfaces can be used as type-hints when autowiring:
 (only showing classes/interfaces matching Mailer)

 Interface for mailers able to send emails synchronous and/or asynchronous.
 Symfony\Component\Mailer\MailerInterface (mailer.mailer)
```

在傳送電子郵件時，`Symfony\Component\Mailer\MailerInterface` 類型被指定為用於 **type-hint** 的類型。 此外，我們看到相關的服務是 `mailer.mailer`，也就是我們先前辨識的服務。

## Type-hint

一旦確定了類型，我們就可以在控制器中使用它：

```php
public function register(
    Request $request,
    EntityManagerInterface $em,
    MailerInterface $mailer
): Response {
  //...
}
```

要建立並發送電子郵件，然後我們可以參考[郵件組件的文檔](https://symfony.com/doc/5.4/mailer.html#creating-sending-messages)。

若要建立具有更高級格式（例如 Twig 範本）的電子郵件，請查閱[文件中的這個部分](https://symfony.com/doc/5.4/mailer.html#html-content)。
