# 使用 API (應用程式介面)

一旦我們的 API 就位，我們就想從 news(新聞)應用程式中使用它。

當用戶要註冊 newsletter 時，我們要聯絡 API 以驗證電子郵件註冊。

## HTTP 用戶端

要使用 Symfony 提出請求，我們可以使用 [`http-client`](https://symfony.com/doc/5.4/http_client.html) 元件。

:::info 客戶端/伺服器
雖然我們的新聞應用程式是一個 Symfony 應用程序，而且從瀏覽器的角度來看它是一個**伺服器**，但在這裡它卻成了我們 API 的**客戶端**：它希望使用外部服務。
:::

在控制器中，我們可以使用類型提示來聲明 `HttpClientInterface` 介面。 這樣，我們就有了一個可以呼叫 `request` 請求方法的實例：

```php
$response = $client->request(
  Request::METHOD_POST, // 我們使用 POST 方法
  "http://localhost:8001/api/check", // 我們要請求的 URL
  [
    // 資料將自動轉換為 JSON 格式，並整合到請求體中。
    'json' => ['email' => $newsletterEmail->getEmail()]
  ]
);
```

`$response` 是一個實作了 `ResponseInterface` 介面的類別的實例。

在這個實例上，我們可以呼叫 `toArray()` 方法，自動提取數組形式的回應內容：

```php
$data = $response->toArray();
$isSpam = $data['result'] === 'spam';
```

然後，我們可以根據情況決定如何處理：例如，如果電子郵件是垃圾郵件，我們可以在表單中新增一個特殊錯誤，使其無效：

```php
$form->addError(new FormError("Une erreur est survenue lors de la vérification de l'email"));
```

## ScopingHttpClient

到目前為止，還有一個大問題：我們的 API 的 URL 在程式碼中是**硬編碼**。

我們希望有更多的彈性，尤其是在環境方面。 在本地開發期間，我們的 API 可能在 `localhost:8001` 上運行，但是在生產環境中呢？ URL 肯定是不同的。

這牽涉到**環境**層面的差異。 因此，我們可以配置客戶端以依賴環境變量，而不是在應用程式中硬編碼 URL。

我們可以在框架層級進行配置，宣告一個 [`scoped_clients`](https://symfony.com/doc/5.4/http_client.html#scoping-client)：

```php
config/packages/framework.yaml

framework:
  # ...

  http_client:
    scoped_clients:
      spam.checker:
        base_uri: "%env(SPAM_CHECKER_URI)%"
```

:::danger 縮排
注意 YAML 檔案中的縮排
:::

在 `.env.local` 檔案中，我們輸入了對應的環境變量 ：

```php
.env.local
SPAM_CHECKER_URI=http://localhost:8001
```

在進行了這樣的配置後，Symfony 會自動建立一個**自動組裝**的服務，我們可以以以下方式進行類型提示：

```php
public function newsletterSubscribe(
  //...
  HttpClientInterface $spamChecker
): Response {
  //...
  $response = $spamChecker->request(
    Request::METHOD_POST,
    "/api/check",
    [
      'json' => ['email' => $newsletterEmail->getEmail()]
    ]
  );
}
```

:::note 類型和名稱
在這裡，類型提示也透過參數的**名稱**進行了補充：Symfony 自動建立了這個服務，並將其自動連接到我們的 scoped client。 就好像我們告訴 Symfony 我們**確切**地想要的是 HttpClient Spam Checker。 它知道這是哪個配置。
:::

現在，我們只需使用`/api/check`即可。 Symfony 會自動在 URI 前加上環境變數中指定的位址。
