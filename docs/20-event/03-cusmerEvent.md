# 自定義事件

在我們的程式碼中，還可以**自定義事件**並使用 Symfony 的事件派發器進行傳播。

## 解耦程式碼

讓我們以訂閱 newsletter 為例。

在控制器中，我們在同一位置執行與電子郵件註冊**相關**的所有附加操作。

因此，如果表單已提交且有效，我們將註冊電子郵件，然後向使用者發送電子郵件。

相反，我們可以將電子郵件註冊邏輯保留在控制器中，然後在應用程式內傳播一個事件，指示已註冊電子郵件。

因此，控制器不再需要處理電子郵件註冊產生的副作用。 它只需要執行主要操作，即資料持久化，然後通知應用程式的其他部分已註冊電子郵件。

這種方法更加**靈活**：我們可以在應用程式的其他位置定義程式碼片段（類別），它們將對事件作出**回應**並執行自己的邏輯。

我們還可以想像，事件可以從應用程式的其他地方啟動：例如，如果我們透過 API 路由註冊電子郵件，除了表單資料接收控制器之外，我們還可以從另一個地方傳播相同的事件。 然後同樣的**監聽器**或**事件訂閱**將會被觸發。

:::info 事件

事件的使用更加靈活，因為它允許**解耦代碼**。 在需要時，可以停用或重新啟用某些監聽器，而不是在一個地方進行註解/取消註解程式碼。

然而，它可能會在應用程式內引入一些不確定性。 實際上，當我們看到某個事件在某個地方被觸發時，通常需要瀏覽程式碼以找到監聽此事件的監聽器/訂閱者。
:::

## 定義自訂事件

首先，需要在應用程式內定義自訂事件。 為此，我們將定義一個新類別。

如果我們想要通知應用程式的其他部分，有人剛剛訂閱了 newsletter，我們可以想像一個名為 `NewsletterSubscribedEvent` 的類別：

```php
<?php

namespace App\Event;

use App\Entity\Newsletter;

class NewsletterRegisteredEvent
{
  public const NAME = 'newsletter.registered';

  public function __construct(private Newsletter $newsletter)
  {
  }

  public function getNewsletter(): Newsletter
  {
    return $this->newsletter;
  }
}
```

在這個類別中，我們將定義幾個內容：事件的名稱和與事件相關的資料。

### 事件的名稱

可能已經在 Symfony 的 Kernel 中註意到了，通常事件的命名方式如下：`topic.event`。

我將事件按主題（`topic`）分類，一個事件通常是一個動詞（可以使用過去分詞）：`registered`。

事件的名稱(這裡模仿事件類別的名稱)，應該代表事件觸發時發生的情況：`newsletter.registered`。

:::note 事件的名稱
就像變量、類別、方法等一樣，我們在事件的命名上是自由的。

我們也可以將上述事件命名為 `newsletter.subscribed`，`newsletter.created` 等，只要清晰即可。
:::

### 相關的數據

我們是建立事件類別的人：因此我們定義了事件需要攜帶的有用數據，以便被監聽器或訂閱使用。

在這裡，如果我們的事件涉及向 newsletter 註冊新電子郵件，那麼我們可以在此事件中攜帶的資料就是與此相關的實體。 因此，我們新增一個私有屬性，類型為 `Newsletter`。

當派發程式呼叫監聽器或訂閱時，它將接收到這個資料。 對於我們的例子，我們將建立一個訂閱者，該訂閱者將向剛剛註冊的使用者發送電子郵件。 因此，我們需要剛剛註冊的 `Newsletter` 實例，以便能夠取得相關的電子郵件，然後觸發發送電子郵件的操作。

## 派發事件

一旦我們創建了事件類，我們就可以回到我們的控制器。

在控制器中，我們希望在適當的時候派發事件，也就是在實體被註冊後。

我們可以在控制器中使用 `Symfony\Component\EventDispatcher\EventDispatcherInterface` 介面進行類型提示，以取得事件派發器：

```php
use Symfony\Component\EventDispatcher\EventDispatcherInterface;

//...

#[Route('/newsletter/register', name: 'newsletter_register')]
public function register(
  Request $request,
  NewsletterRepository $newsletterRepository,
  EventDispatcherInterface $dispatcher
): Response {
  //...
}
```

然後，在 Newsletter 實例被註冊（持久化）的時候，我們可以派發事件：

```php
if ($form->isSubmitted() && $form->isValid()) {
  $newsletterRepository->save($newsletter, true);
  // 我創建了一個事件實例，並向其傳遞相關的實體
  // 因此，事件將攜帶與之相關的數據，可供監聽器/訂閱者使用
  $event = new NewsletterRegisteredEvent($newsletter);
  // 透過指定事件的名稱來派發事件
  $dispatcher->dispatch($event, NewsletterRegisteredEvent::NAME);
}
```

:::note 注意：
我們可以直接在 `dispatch` 方法的呼叫中實例化事件
:::

## 監聽和回應事件

目前，我們已經觸發了事件，但沒有人在監聽。

我們之前看到，事件幫助我們解耦邏輯。 因此，我們將建立一個訂閱者來監聽 `newsletter.register` 事件，並在該事件被派發時執行操作。 這個訂閱者將負責向使用者發送電子郵件。

### 創建一個訂閱者

要建立一個訂閱者，我們可以使用 Symfony 的控制台和 maker：

```bash
php bin/console make:subscriber
```

对于类的名称，我们将指定 `NewsletterRegisteredSubscriber`：这是 `newsletter.registered` 事件的订阅者。虽然我们可以按照自己的意愿进行命名，但由于我们正在解耦逻辑，因此我们会尽量保持一致的名称，使它们彼此接近。

然后，Maker 要求我们订阅哪个事件。它提供了一些已经存在于应用程序中的事件的建议，但我们想要的是 `newsletter.registered`。

Maker 随后为我们创建了一个类 `App\EventSubscriber\NewsletterRegisteredSubscriber`：

```php
<?php

namespace App\EventSubscriber;

use Symfony\Component\EventDispatcher\EventSubscriberInterface;

class NewsletterRegisteredSubscriber implements EventSubscriberInterface
{
  public function onNewsletterRegistered($event): void
  {
    //...
  }

  public static function getSubscribedEvents(): array
  {
    return [
      'newsletter.registered' => 'onNewsletterRegistered',
    ];
  }
}
```

就像我們之前為創建用戶實體時對密碼進行哈希的訂閱者一樣，我們可以看到這個訂閱者實作了 EventSubscriberInterface 介面。

:::caution DOCTRINE & SYMFONY EVENTSUBSCRIBERINTERFACE
注意，這兩個訂閱者不屬於相同的命名空間，一個是 Doctrine 的，另一個是 Symfony 的。

因此，我們在這裡看到了相同的邏輯，例如 `getSubscribedEvents()` 方法，但在這裡，這個方法是靜態的。

因此，這兩個訂閱者不會以相同的方式被標記。
:::

### 訂閱者的實現

我們可以透過引用事件類別中定義的常量 `NAME` 來更改事件的名稱：

```php
public static function getSubscribedEvents(): array
{
  return [
    NewsletterRegisteredEvent::NAME => 'onNewsletterRegistered',
  ];
}
```

我們可以看到將事件與 `onNewsletterRegistered` 方法關聯起來。 現在我們只需要將發送電子郵件的邏輯移到這個方法中，當然需要注入我們需要的服務或參數：

```php
public function __construct(
  private MailerInterface $mailer,
  private string $adminEmail
) {
}

public function onNewsletterRegistered(NewsletterRegisteredEvent $event): void
{
  $newsletter = $event->getNewsletter();

  $email = (new Email())
    ->from($this->adminEmail)
    ->to($newsletter->getEmail())
    ->subject("Inscription à la newsletter")
    ->text("Votre email " . $newsletter->getEmail() . " a bien été enregistré, merci");

  $this->mailer->send($email);
}
```

如果我們測試 newsletter 註冊表單，我們應該像之前一樣收到電子郵件。

在行為方面，沒有特別的改變。 我們是在設計層面**解耦**了我們的程式碼。
