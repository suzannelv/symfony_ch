# 身份驗證和存取(訪問)控制

既然資料庫中已經有了用戶，我們就需要對他們進行身份驗證。

就像執行 `make:user` 指令一樣，我們可以執行另一個 `make:auth` 指令，選擇 `Form Login` 來自動產生登入表單和驗證器。

## 防火牆 (firewalls)

當我們執行這條指令時，Maker 會自動在名為 **main** 的**防火牆**後面註冊我們的身分驗證：

> config/packages/security.yaml

```yaml
security:
    firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false
        main:
            lazy: true
            provider: app_user_provider
            custom_authenticator: App\Security\AppAuthenticator
            logout:
                path: app_logout
```

防火牆(**firewalls**)只是過濾傳入的請求。

:::note "DEV"
開發環境下的防火牆（Firewall）僅用於允許顯示偵錯器（profiler）和 Web Debug Toolbar，我們可以在螢幕底部看到該工具列，其中包含有關當前請求/頁面的各種資訊。
:::

因此，在我們的主防火牆（firewall `main`）中，我們找到了身份驗證器（authenticator）和用於使用者的身分提供者（provider）。

:::info 什麼是 LAZY？
`lazy` 指令的作用是只有在實際需要時才使用 PHP 會話來載入使用者。 例如，如果需要驗證使用者的角色，這可能很有用。 但情況並非總是如此。
:::

## 驗證器

`make:auth` 指令在 `custom_authenticator` 中產生並註冊了一個 `App\Security\AppAuthenticato`r 類別。

如果我們存取該類，會發現它繼承自一個抽象類別 `AbstractLoginFormAuthenticator`。 該類別由 Symfony 提供，以加快開發速度。 它已經包含了一個預設實現，可以滿足我們的需求。`

我們的驗證器包含一個 `authenticate` 方法，它只負責驗證傳入的請求（`Request` 物件）。

### 護照（Passports）

此身份驗證機制將透過建立 `Passport` 來實現和驗證，然後 Passport 傳回給呼叫者。

一個 `Passport` 將包含要進行身份驗證的使用者以及一些訊息，例如要驗證的密碼，以及要驗證的 CSRF 令牌。

在這裡，我們的 `Passport` 將包含三個資訊：

```php
public function authenticate(Request $request): Passport
{
    $email = $request->request->get('email', '');

    $request->getSession()->set(Security::LAST_USERNAME, $email);

    return new Passport(
        new UserBadge($email),
        new PasswordCredentials($request->request->get('password', '')),
        [
            new CsrfTokenBadge('authenticate', $request->request->get('_csrf_token')),
        ]
    );
}
```

**`UserBadge`**

由使用者識別碼產生的 `UserBadge`, 將用於透過關聯的提供者**檢索**使用者。

在 security 元件的設定中（`config/packages/security.yaml`），我們可以看到在主防火牆(main firewall)中，也就是我們放置了 `AppAuthenticator` 的地方，設定的提供者（使用者提供者）是 `app_user_provider`。

在聲明的 `provider` 中，這是 maker 在執行 `make:user` 指令時所建立的提供者：正是這個提供者能夠透過電子郵件尋找使用者。

**`Credentials`**

`Credentials` 只是身分驗證資訊（這裡是密碼）。 為了驗證它們，我們可以想像在表單中輸入的密碼將與資料庫中的雜湊進行比較。

**`CsrfTokenBadge`**

然後，`Passport` 可能會包含額外的徽章，如我們在 `Authenticator` 中所見，它們添加了其他類型的資料（在本例中，整合 CSRF 令牌的管理）。

因此，`Passport` 的目標將是驗證它所提供的各種 `Badge` 或 `Credentials`。

例如，`UserBadge` 將檢索使用者。 如果找不到使用者（電子郵件不存在），則徽章將不會被解析，無法通過驗證。 如果找到了用戶，它將嘗試驗證 `Credentials`，然後是 CSRF 令牌等...如果所有徽章都通過驗證，那麼身份驗證就成功了。

## 存取控制

通過身份驗證後，可能還有另一個階段：存取控制。

例如，假設我們要為文章產生一個 CRUD。

`make:crud` 指令會產生用於管理文章的控制器類別和檢視。

但之後，我們想限制只有管理員使用者才能存取該 CRUD。

首先，在控制器類別中，我們可以使用類別層級的 `Route` 屬性來定義 URL 前綴：

```php
#[Route("/admin/article")]
class ArticleController extends AbstractController
{
}
```

因此，該類別中的所有控制器都會回應以 `/admin/article` 開頭的 URL。

### 管理員

接下來，我們將在安全元件設定檔中啟動對所有以 `/admin` 開頭的 URL 的存取控制：

> config/packages/security.yaml

```yaml
security:
    access_control:
        - { path: ^/admin, roles: ROLE_ADMIN }
```

現在，當我們要查看以 `/admin` 開頭的 URL 時，Symfony 將執行以下操作：

-   檢查請求對應的防火牆。 在這裡，與 `/admin` 相符的請求將與我們的主(`main`)防火牆相符。
-   在防火牆上，我們沒有指定任何 URL 的 “pattern”，與 `dev` 防火牆不同，後者使用 `pattern: ^/(_(profiler|wdt)|css|images|js)/`，表示與所有以"\_profiler"、 "\_wdt"、"css"等開頭的 URL 符合。 所以，預設情況下，我們不需要進行身份驗證。
-   但在存取控制方面，我們指定要將所有以 `/admin` 開頭的內容限制為 `ROLE_ADMIN` 角色才能存取。 。
-   這時，這意味著我們需要一個已經過身份驗證的用戶，以便檢查他或他們的角色。
-   於是，我們啟動了 `AppAuthenticator` 身份驗證器。
-   預設情況下，我們尚未登入。 系統會自動重新導向至與 `authenticator` 關聯的登入頁面（請參閱 `getLoginUrl` 方法）。
-   我們使用表單進行身份驗證。 然後，如果我們的角色正確（如果我具有"ROLE_ADMIN"角色），那麼我就可以訪問我的管理頁面了。

### 僅限登入使用者（無論什麼角色）

```yaml
access_control:
    - { path: ^/connected, roles: IS_AUTHENTICATED_FULLY }
```

### 任何用戶（公開訪問）

```yaml
access_control:
    - { path: ^/public, roles: PUBLIC_ACCESS }
```
