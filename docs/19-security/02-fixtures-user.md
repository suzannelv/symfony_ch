# 使用者 fixtures 配置

每次建立表格之後一樣，我們需要一些可用的測試數據，以便在安裝專案後立即進行工作。

因此，我們將新增 2 個使用者設定：一個普通使用者和一個管理員使用者。

## 密碼

為了對密碼進行雜湊處理，我們將需要一個服務。

如果不知道如何對密碼進行雜湊處理，可以找到可用的**自動配置**的服務，以便直接將其註入到固定配置的建構函數中：

```bash
php bin/console debug:autowiring Password
```

乍一看，我們可能會對 `UserPasswordHasherInterface` 和 `UserPasswordEncoderInterface` 這兩個介面感興趣。

稍作研究就會發現，`UserPasswordHasherInterface` 是在 Symfony 5.3 中的 `PasswordHasher` 元件中引入的，以便更清晰地命名元件，並避免密碼編碼而不是密碼雜湊引起的混淆。

因此，我們將使用 `PasswordHasher` 介面注入密碼雜湊服務：

```php
use Symfony\Component\PasswordHasher\Hasher\UserPasswordHasherInterface;

class AppFixtures extends Fixture
{
  public function __construct(private UserPasswordHasherInterface $hasher)
  {
  }
}
```

該介面有一個 `hash` 方法，我們可以在服務容器提供的實例上使用該方法。 然後，我們就可以建立兩個使用者並將其持久化到資料庫中：

```php
public function load(ObjectManager $manager): void
{
  $regularUser = new User();
  $regularUser
    ->setEmail('bobby@bob.com')
    ->setPassword($this->hasher->hashPassword($regularUser, 'test'));

  $manager->persist($regularUser);

  $adminUser = new User();
  $adminUser
    ->setEmail('admin@mycorp.com')
    ->setRoles(['ROLE_ADMIN'])
    ->setPassword($this->hasher->hashPassword($adminUser, 'test'));

  $manager->persist($adminUser);

  //...其它fixures...

  $manager->flush();
}
```

這樣，密碼已經以哈希形式儲存在資料庫中！
