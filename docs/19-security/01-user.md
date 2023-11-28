# 用戶（使用者）

許多應用程式都需要對用戶進行身份驗證。 Symfony 提供了一個 `Security` 元件來管理身分驗證和授權（或存取控制）。

要在應用程式中引入使用者實體，可以使用 Maker 的 `php bin/console make:entity` 指令。 不過，Symfony 提供了 `php bin/console make:user` 指令來建立與 Symfony 安全元件相容的使用者實體。

這條命令會提出一系列關於我們如何將使用者整合到應用程式中的問題。 建立後的使用者實體的確與傳統建立的實體不同：它實作了 2 個介面：`UserInterface` 和 `PasswordAuthenticatedUserInterface`。

這些介面定義了一個實作契約，該契約由創建安全元件的開發人員設定。

它們強制使用者實體實作特定的方法，然後安全元件將使用這些方法。

例如，`getRoles()` 或 `getPassword()` 等方法。

:::note 角色
Symfony 以陣列 `array` 形式管理使用者角色。 在資料庫中，這將以 `json` 欄位的形式表示。
:::

如果您想要在使用者中新增額外的字段，完全可以在執行 `make:user` 指令後重新使用 `make:entity` 指令。 `make:user` 指令只是建立一個基本的用戶，與安全元件相容。 但是這個使用者其實是一個實體。 因此，我們可以隨心所欲地向為我們的應用程式添加附加屬性。

最後，就像我們在建立實體時所做的一樣，我們需要建立一個遷移並執行它。

就像我們建立常規實體時一樣，建立一個使用者實體, 在我們的例子中命名為 `User` , 會自動觸發 `UserRepository` 類別的建立。

此外，在安全性元件的設定檔中（位於 `config/packages/security.yaml`），`make:user` 指令自動新增了下列行：

```yaml
security:
    # ...

    providers:
        app_user_provider:
            entity:
                class: App\Entity\User
                property: email # <-- si nous avons choisi "email" comme nom de champ d'identification lors de l'exécution de la commande "make:user"
```

這就是使用 `make:user` 指令的另一個優點。 它直接在安全組件中註冊了一個用戶提供者。

這個提供者將能夠找到用戶，一旦經過身份驗證，還可以從會話中獲取用戶並進行刷新。

最後，在安全元件的設定中，也會註冊一個 `password_hasher`：`

```yaml
security:
    # ...
    password_hashers:
        Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface: "auto"
```

:::info 密碼哈希
值 "auto" 允許自動選擇目前最佳演算法。 例如，在 Symfony 5.4 中，它將自動選擇 bcrypt 演算法。
:::
