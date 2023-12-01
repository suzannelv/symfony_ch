# 自訂存(自定義)取控制

在 `security.yaml` 設定檔中的存取控制可以允許我們將整個部分限制為特定的角色、HTTP 方法或其他各種選項。

但是，如果我們想要更精細的存取控制，例如在控制器級別，配置將不再足夠。

## 在控制器中

在控制器中，我們可以使用 `AbstractController` 提供的方法。

### `denyAccessUnlessGranted`

`denyAccessUnlessGranted($attribute, $subject = null, string $message = 'Access Denied.') ` 可以在使用者沒有權限存取資源（`subject`）執行操作（`attribute`）時自動產生 `403 Forbidden` 錯誤。

:::note 角色
我們也可以在 `$attribute` 中傳遞一個角色：

```php
$this->denyAccessUnlessGranted('ROLE_ADMIN');
```

:::

### `isGranted`

`isGranted($attribute, $subject = null)` 傳回布林值，表示使用者是否有權存取資源以執行操作。 它不會自動產生 403 錯誤，只會傳回一個布林值。

我們也可以將它與一個角色一起使用，以驗證使用者是否具有指定的角色。

:::danger getRoles()
建議不要直接在使用者上使用 `getRoles()` 方法來驗證其角色。

相反，應該使用 `denyAccessUnlessGranted` 或 `isGranted` 方法。
:::

## 取得用戶

### 從控制器中

如果我們想要從控制器中取得用戶，我們可以使用實用方法 `$this->getUser()`。 回傳類型當然是 `UserInterface|null`。

### 從服務中

如果我們不在控制器中，我們可以直接注入 `Security` 類型：
`

```php
class MyService
{
  public function __construct(
    private Security $security
  ) {
  }

  public function myMethod()
  {
    $user = $this->security->getUser();
  }
}
```

:::note SECURITY
`Security` 服務提供了 `isGranted` 方法，用於檢查使用者是否具有某個角色或能夠執行某個操作。 再次強調，**沒有必要**取得用戶，然後使用 `getRoles()` 檢查他的角色：我們可以直接在 `Security` 服務上使用 `isGranted`。
:::

## 使用投票者（`Voter`）

為了實現更詳細的存取控制，我們還可以編寫自訂的 `Voter`。

例如，假設只有當登入的使用者是文章的作者或管理員時，才能編輯或刪除文章。

在控制器中，我們可以檢查使用者是否有權修改文章：

```php
$this->denyAccessUnlessGranted('ARTICLE_EDIT', $article);
```

驗證邏輯可以寫在一個繼承自抽象類別 Voter 的類別中：

```php
class ArticleVoter extends Voter
{
  public const EDIT = 'ARTICLE_EDIT';
  public const DELETE = 'ARTICLE_DELETE';

  public function __construct(
    private Security $security
  ) {
  }

  protected function supports(string $attribute, mixed $subject): bool
  {
    return in_array($attribute, [self::EDIT, self::DELETE])
      && $subject instanceof Article;
  }

  protected function voteOnAttribute(string $attribute, mixed $subject, TokenInterface $token): bool
  {
    $user = $token->getUser();
    // if the user is anonymous, do not grant access
    if (!$user instanceof UserInterface || !$subject instanceof Article) {
      return false;
    }

    if ($this->security->isGranted('ROLE_ADMIN') || $subject->getAuthor() === $user) {
      return true;
    }

    return false;
  }
}
```

:::note MAKER
我們可以用 Maker 建立一個投票者：`php bin/console make:voter`
:::

只要我們的投票者繼承了 `Voter` 抽象類，ymfony 就會自動偵測到我們的服務，並自動將其註冊為存取控制驗證。

因此，當我們呼叫 `denyAccessUnlessGranted` 或 `isGranted` 時，該 voter 會自動被呼叫。

首先，Symfony 將呼叫 `supports` 方法來查詢每個已註冊的投票人。 此方法會傳回布林值，表示它是否能為屬性和主題提供決策。

如果 `supports` 回傳 `true`，那麼 Symfony 將自動呼叫投票人的 `voteOnAttribute` 方法。
