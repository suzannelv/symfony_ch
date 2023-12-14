# 控制器

若要建立控制器，請使用 maker 工具 ：

```bash
php bin/console make:controller Spam
```

然後就會在 `src/Controller` 資料夾中建立一個 `SpamController` 控制器。

例如，我們可以將 `index` 方法重新命名為 `check`，並調整其路由：

```php
class SpamController extends AbstractController
{
  #[Route('/api/check', name: 'app_spam_check', methods: "POST")]
  public function check(Request $request): JsonResponse
  {
  }
}
```

## 路由

路由將在 URI `/api/check` 後進行回應。 使用的 HTTP 方法是 `POST`，以便在請求體中接收要檢查的電子郵件。

然後，我們可以使用例如 Postman 這樣的工具，透過其介面手動建構請求來測試 API。

## 控制器

`check` 方法包含來自 `HttpFoundation` 元件的 `Request` 的類型提示。 然後，Symfony 會提供請求，我們可以從中找到要驗證的電子郵件（在請求體中）。

回傳類型比平常精確一些：我們接收一個請求並回傳一個回應，但在 API 的情況下，我們指定了 `JsonResponse` 類型。

## 實現

### 提取請求體

為了從請求中提取電子郵件，我們可以使用 `toArray()` 方法：該方法會提取請求主體的內容並將其轉換為陣列。 然後，我們可以提取收到的電子郵件：

```php
$data = $request->toArray();
$email = $data['email'];
```

### 驗證電子郵件格式

要驗證電子郵件格式，我們可以使用 SPL 的 `filter_var` 函數。

但 Symfony 的電子郵件 `Validator` 元件使用了一個軟體包：`egulias/email-validator`。 我們可以將其新增至專案的依賴關係：

```bash
composer require egulias/email-validator
```

:::note 驗證
要驗證電子郵件的格式是否正確，還可以探索其他可能性：例如安裝 [Symfony 的驗證器](https://symfony.com/doc/5.4/validation.html#using-the-validator-service)。
:::

### 電子郵件驗證

要檢查電子郵件是否為垃圾郵件，我們首先要定義一個靜態陣列。

然後，我們可以在控制器類別中將其宣告為一個私有常數：

```php
class SpamController extends AbstractController
{
  private const SPAM_DOMAINS = ["test.com", "mailinator.com", "youpi.net"];

  //...
}
```

最後，在控制器中，我們可以使用 SPL 的 `in_array` 方法檢查電子郵件網域是否在陣列中。

## 回饋響應

要傳回 JSON 格式的回應，`AbstractController` 類別提供了一個實用方法：`json`。

此方法首先檢查`serializer`(序列化)元件是否存在，該元件用於將資料從 PHP 傳輸到定義的格式（JSON、XML、CSV 等），反之亦然。 由於我們的應用程式是以 API 模式安裝的，因此沒有該元件。

因此，該方法需要使用 SPL `json_encode`。

在控制器中，我們不必擔心這個實作細節。 我們只需向 `json` 方法傳遞要以正確格式傳回的資料即可：

```php
if (in_array($domain, self::SPAM_DOMAINS)) {
  return $this->json(['result' => 'spam']);
}

return $this->json(['result' => 'ok']);
```
