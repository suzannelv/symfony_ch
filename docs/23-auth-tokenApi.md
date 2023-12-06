# 身份驗證與 API 令牌

身份驗證與 API 令牌

現在我們有了一個返回文章的端點，我們希望透過身份驗證來保護它，以便將其使用限制為某些用戶。

為此，我們將要求客戶端在其請求中附帶一個 `X-API-TOKEN` 標頭。 伺服器的作用將是提取此標頭，然後驗證令牌是否有效（在資料庫中註冊）。

## 運作方式

我們將要建立的 API 將整合到一個使用 `--webapp` 模式建立的全端應用程式中。 因此，我們已經有了一個透過登入表單進行身份驗證的機制和 `User` 安全類別。

對於我們的 API，我們希望有一個稍微不同的系統：我們想要保留登入表單和我們的普通用戶，以便他們可以登入並使用應用程式建立文章等等... 但我們希望外部用戶、其他服務或 應用程式可以存取我們的應用程式介面部分，以便讀取我們的 JSON 格式文章。

因此，我們不會將 API 金鑰整合到現有的 `User` 類別中，而是建立一個新的安全類別 `ApiToken`，並將其儲存在資料庫中。

這個類別將成為一個表，攜帶一個身份驗證令牌，用於授權或拒絕存取我們的 API。

因此，任何對我們的 API 發出請求的客戶端都必須在請求的頭部中附加其令牌。 透過帶有令牌的頭，我們才能辨識請求。

## 安全類

要建立一個安全性類，我們可以使用控制台的 `make:user` 指令。

因此，身份驗證資料將是一個我們將儲存在資料庫中的 `token` 列（因此我們希望將其作為 Doctrine 實體）。

然而，不需要任何密碼。 因此，我們不希望使用我們的 `ApiToken` 安全類別來雜湊或驗證密碼。 具體來說，這意味著什麼呢？ 這意味著 maker 建立的類別將只實作 `UserInterface` 介面而不是 `PasswordAuthenticatedUserInterface` 介面。

一旦我們正確回答了 maker 的問題以創建我們的新安全類，我們將產生一個遷移，並在審查後執行該遷移，以建立我們的新表。

:::note make:entity
在建立實體後，我們可以使用 `make:entity` 為實體新增其他欄位。

例如，我們將新增一個 `name` 字段，以便了解與請求相關聯的客戶端。 我們也可以新增一個請求計數器，以限制每天、每月等的請求次數。
:::

### 令牌提供者

Maker 會自動在設定檔 `config/packages/security.yaml` 中為我們的安全類別建立了一個提供者（**provider**）。

如果我們已經有一個現有的提供者（用於我們的 `User` 類別），它可能已經被覆寫了。 在這種情況下，讓我們重新配置以適應我們的操作：我們想要兩個提供者（providers），一個用於 `User`，一個用於 `ApiToken` ：

> `config/packages/security.yaml`

```yaml
security:
    enable_authenticator_manager: true
    # https://symfony.com/doc/current/security.html#registering-the-user-hashing-passwords
    password_hashers:
        Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface: "auto"
    # https://symfony.com/doc/current/security.html#loading-the-user-the-user-provider
    providers:
        # used to reload user from session & other features (e.g. switch_user)
        app_user_provider:
            entity:
                class: App\Entity\User
                property: email
        api_token_provider:
            entity:
                class: App\Entity\ApiToken
                property: token
```

## 防火牆（Firewall）

為了捕獲 API 請求並透過身份驗證對其進行保護，我們需要在配置中新增一個**防火牆**。

透過將其配置為特定 URL 格式，我們可以**隔離**API 請求並將其提交給我們的身份驗證機制。

在配置的 `firewalls` 部分，我們將新增一個名為 `api` 的防火牆：

```yaml
firewalls:
    dev:
        pattern: ^/(_(profiler|wdt)|css|images|js)/
        security: false
    api:
        pattern: /api/articles
        provider: api_token_provider
    main:
    #...
```

:::danger 縮排

注意 YAML 格式和縮排！
:::

我們定義的模式非常具體：因為我們只有一個 `/api/articles` 端點，所以暫時我們將為此 URL 定義一個防火牆。 將來，如果我們定義了更多的端點，例如以 `/api` 開頭等，我們可以擴展這個模式...

我們也指定了我們要使用的**提供者**：在本例中，我們的使用者提供者將是先前新增至提供者中的 `api_token_provider`。

## 認證器

安全類別和防火牆建立完成後，我們需要實施身份驗證機制，從請求頭（如果存在）中提取令牌，然後檢查資料庫中是否存在令牌，以便授權請求。

如我們所見，要建立認證器，我們可以使用 `make:auth` 命令。 但與前面建立登入表單的方式不同，我們將建立一個**空的**認證器（Empty Authenticator）。 實際上，maker 將產生一個預先填入的認證器類，然後由我們編寫各種方法的實作。

我們將這個認證器命名為 `ApiTokenAuthenticator`。

:::note 防火牆

maker 會偵測我們配置中存在的不同防火牆，並詢問我們要將認證器套用到哪個防火牆。 因此，我們將選擇 `api` 防火牆。
:::

建立的類別包含了從 `Symfony\Component\Security\Http\Authenticator\AuthenticatorInterface`介面繼承的各種方法：

-   `supports` 指示我們的認證器是否能夠根據傳入的請求對使用者進行認證。
-   `authenticate`允許從傳入的請求對使用者進行認證。 在這個方法中，我們將編寫認證的方式。 為此，我們將建立一個護照（`Passport`），然後將其傳回給安全元件。
-   `onAuthenticationSuccess` 指示如果認證成功，應該執行什麼操作。
-   `onAuthenticationFailure` 則指示相反的情況：如果認證失敗，應該執行什麼操作。

:::info 安全組件

所有這些方法都是由安全元件**自動**呼叫的，它會偵測並註冊應用程式中存在的認證器。

因此，我們正在提供一個**新的行為**，這將整合到安全組件執行的整個過程中。

### `supports`

這是安全元件呼叫的第一個方法。 它的目的是檢查我們的認證器是否可以使用。

在我們的例子中，我們將指示**只有**當請求包含 `X-API-TOKEN` 頭時，我們才能對其進行認證：

:::

```php
class ApiTokenAuthenticator extends AbstractAuthenticator
{
  // 令牌名称存储在一个常量中，因此其值可以在多个地方重复使用
  private const TOKEN_NAME = 'X-API-TOKEN';

  public function supports(Request $request): ?bool
  {
    return $request->headers->has(self::TOKEN_NAME);
  }

  //...
}
```
