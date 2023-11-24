# 配置參數

Symfony 的服務容器包含服務，也可以包含[**應用程式參數**](https://github.com/symfony/dependency-injection/blob/5.4/ContainerInterface.php#L70)。

## config/services.yaml 文件

在包含服務配置的 `config/services.yaml` 檔案中，還有一個預設為空的設定 `parameters` (參數)部分：

```yaml
parameters:

services:
    # default configuration for services in *this* file
    _defaults:
        autowire: true
```

在我們剛剛撰寫的電子郵件發送服務中，我們指明了一個發送地址（"admin@hb-corp.com"）。

問題是：在應用程式中, 我們如何在所有需要使用管理地址自動發送電子郵件的地方避免重複使用這個電子郵件地址？

我們可以將該變量作為**參數**聲明到 `config/services.yaml` 檔案中：

```yaml
parameters:
    app.admin_email: "admin@hb-corp.com"

services:
    # ...
```

:::info 環境變量？

這類數據與環境變量有何不同？ 很簡單，因為從一個環境到另一個環境，用於發送自動郵件的管理電子郵件不會改變。 這是應用程式的數據，而不是執行環境的數據。

它會隨應用程式的環境而改變嗎？ 那就將其設為環境變數 😄 。

:::

之後，我們希望能在服務中使用這個變量。 我們可以將其**註入**到建構函數中：

```php
class NewsletterSubscribed
{
  private $mailer;
  private $adminEmail;

  public function __construct(MailerInterface $mailer, string $adminEmail)
  {
    $this->mailer = $mailer;
    $this->adminEmail = $adminEmail;
  }
}
```

在此階段，我們只需在建構函式中指出服務在建置時需要 `string $adminEmail`，從而定義服務。 如上文所述，自動組裝適用於服務，但不適用於參數。

## 手動服務配置

在 `config/services.yaml` 檔案中，我們可以決定手動設定服務，以便在建置服務時自動提供 `app.admin_email` 參數。 我們可以在文件底部添加 ：

```yaml
App\Mail\NewsletterSubscribed:
    arguments:
        $adminEmail: "%app.admin_email%"
```

因此，在建置服務時，應用程式參數的值將自動提供給建構函式中的 `$adminEmail` 參數。

## 自動配置所有服務

手動配置服很有趣，但將此變量作為應用程式參數傳遞的最初目的是為了能在所有需要它的服務（現在或將來）中使用它。

這樣，我們就能盡量避免每次建立要傳送電子郵件的服務時，都必須手動設定該服務才能使用 app.admin_email 變數的情況...

您也可以配置容器，使其自動將服務中的建構函式參數綁定到某個值：
