# 將邏輯移到一個應用服務中

我們將建立一個應用服務，負責從 `Newsletter` 的實例發送電子郵件。

因此，我們將在 `src/` 資料夾中建立一個類，這是我們應用程式邏輯的所在地。 我們甚至將創建一個名為 Mail 的命名空間，並在檔案 `src/Mail/NewsletterSubscribedConfirmation.php` 中建立服務。

## 創建服務

因此，這個服務將是一個類，與檔案同名，並包含一個名為 `send` 的方法：

```php
class NewsletterSubscribedConfirmation
{
  public function send(Newsletter $newsletter)
  {
    $email = (new Email())
      ->from("admin@hb-corp.com")
      ->to($newsletter->getEmail())
      ->subject("Inscription à la newsletter")
      ->text("Votre email " . $newsletter->getEmail() . " a bien été enregistré, merci");

    $mailer->send($email);
  }
}
```

:::note 注意
先前控制器中用於建立和發送電子郵件的邏輯已移至此處
:::

## 將郵件服務 Mailer 注入到服務中

我們已經確定，我們可以進行**類型提示**的類型是 `MailerInterface`。 也就是說，將依賴關係直接注入方法只在控制器中有效。

但是，如果我們在一個獨立的服務中，那麼就必須**透過建構函數**注入依賴關係。

原理很簡單：正如我們可以在控制器方法中鍵入類型，告訴服務容器我們需要什麼類型一樣，我們也可以在服務的建構函數中鍵入類型。 這樣，容器就會提供我們對應的服務實例，我們可以隨意使用。 這裡，我們的想法是聲明一個與該服務相對應的私有屬性，並在構造過程中將容器提供的服務置於該屬性中：

```php
class NewsletterSubscribedConfirmation
{
  public function __construct(private MailerInterface $mailer)
  {
  }

  //...
}
```

隨後，當我們想要呼叫 send 方法時，我們將能夠使用類別的屬性：

```php
public function send(Newsletter $newsletter)
{
  //...

-  $mailer->send($email);
+  $this->mailer->send($email);
}
```

## 在控制器中註入服務

一旦我們的服務準備好了，我們就需要在控制器中使用它，取代先前使用的 `MailerInterface`。 幸運的是，使用 Symfony 的預設配置，我們所有的應用服務都是可以自動注入的：

> config/services.yaml

```yaml
services:
    # default configuration for services in *this* file
    _defaults:
        autowire: true # Automatically injects dependencies in your services.
```

我們也可以使用 Symfony 控制台檢查我們的服務是否作為類型提示可用：

```bash
php bin/console debug:autowiring Newsletter --all
```

```php
Autowirable Types
=================

 The following classes & interfaces can be used as type-hints when autowiring:
 (only showing classes/interfaces matching Newsletter)

 App\Controller\NewsletterController

 App\Form\NewsletterType

 App\Mail\NewsletterSubscribedConfirmation

 App\Repository\NewsletterRepository
```

有了我們的服務後，我們可以在控制器中進行類型提示：

```php
public function register(
    Request $request,
    EntityManagerInterface $em,
    NewsletterSubscribedConfirmation $notificationService
): Response {
  //...

  if ($form->isSubmitted() && $form->isValid()) {
      //Enregistrer l'entité...

      // Utilisation du service qu'on a type-hinté
      $notificationService->send($newsletter);
      //...
  }

  return $this->renderForm('newsletter/register.html.twig', [
      'form' => $form
  ]);
}
```

:::note 依賴注入
透過建構函式進行依賴注入也適用於控制器類別。如果您在多個控制器（因此也包括多個方法）中使用相同的依賴關係，則透過將注入轉移到控制器類別的實例化中，這將非常有用
:::
