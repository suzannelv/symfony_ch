# 事件訂閱

另一種監聽事件的方式是建立一個**事件訂閱**。 在這種情況下，我們可以像**註冊**一樣監聽事件。

讓我們以訂閱的形式建立另一個監聽器。 它將監聽 Doctrine 的 `prePersist`事件，我們將委託給它在持久化期間對使用者密碼進行雜湊處理。

## 訂閱的特點

事件監聽器和訂閱者之間的區別在於，**訂閱者始終知道它們監聽的是哪個（些）事件**：

> src/EventSubscriber/HashUserPasswordSubscriber.php

```php
<?php

namespace App\EventSubscriber;

use App\Entity\User;
use Doctrine\ORM\Event\PrePersistEventArgs;
use Doctrine\ORM\Events;
use Doctrine\Bundle\DoctrineBundle\EventSubscriber\EventSubscriberInterface;
use Symfony\Component\PasswordHasher\Hasher\UserPasswordHasherInterface;

class HashUserPasswordSubscriber implements EventSubscriberInterface
{
  public function __construct(private UserPasswordHasherInterface $hasher)
  {
  }

  public function getSubscribedEvents(): array
  {
    return [
      Events::prePersist,
    ];
  }

  public function prePersist(PrePersistEventArgs $args): void
  {
    $entity = $args->getObject();

    if (!$entity instanceof User) {
      return;
    }

    $entity->setPassword($this->hasher->hashPassword($entity, $entity->getPassword()));
  }
}
```

## 自動配置

在前面的部分中，我們看到了**標籤**的概念，它允許 Symfony 根據應用的需要以特定的方式註冊服務。

在 `HashUserPasswordSubscriber` 類別中，我們聲明實作了 Doctrine 提供的 `EventSubscriberInterface` 介面。

實際上，在我們的應用程式中啟用了一種**自動配置**系統：當讀取到這個介面時，它會**自動為我們的服務添加一個標籤**，使其被視為 Doctrine 的事件訂閱者。

然後，由於該介面定義了一個 `getSubscribedEvents` 方法的簽名，我們實作了這個方法，並聲明我們希望監聽哪個事件。

我們將方法命名為事件的名稱，在 `prePersist` 方法中，我們只有在實體類型為 `User` 時才執行我們的操作。 實際上，訂閱者將對持久化的任何實體進行呼叫。

讓我們從控制台驗證一下我們的服務自動新增的標籤。 對應的標籤是`doctrine.event_subscriber`：

```bash
php bin/console debug:container --tag=doctrine.event_subscriber

```

通常，我們可以在帶有該標籤的服務清單中找到我們的 `HashUserPasswordSubscriber` 類別。

## 添加訂閱者的後果

就像使用監聽器一樣，我們無需擔心可能需要在應用程式的多個地方執行的程式碼邏輯。

相反，我們可以監聽事件並在需要時執行這些程式碼片段。

然而，與監聽器不同的是，我們無需配置我們的訂閱以添加標籤：透過實現接口，Symfony 的**自動配置**機制會接管並自動為我們的訂閱添加標籤。
