# 控制器

如果在安裝應用程式後立即啟動伺服器，我們會發現我們沒有任何首頁。 相反，Symfony 提供了一個特殊的首頁，顯示已安裝的 Symfony 版本，並提供一條訊息，告訴我們之所以看到這個頁面，是因為我們沒有配置主頁。

## 建立控制器

:::tip 控制器
控制器用於接收和處理應用程式中的請求。
:::

透過使用控制台和 `MakerBundle`，我們將創建應用程式的第一個控制器，並配置它在首頁上運行。

```bash
php bin/console make:controller
```

我們將命名 `IndexController` 類，然後查看 maker 建立的 `src/Controller/IndexController.php` 檔案的內容。

:::note 快速方式(RACCOURCI)
我們也可以使用 `php bin/console IndexController` 指令，這樣控制台會直接建立文件，而不會詢問我們控制器類別的名稱
:::

## 控制器類

在由 Maker 建立的類別中，我們找到一個名為 `index` 的函數，該函數將傳回一個 `Respons`e（回應）。 這是 Symfony 的基本原理：**請求/回應模型**。 控制器負責對給定的路由（即 URL）執行操作。 因此，它將接收一個請求，然後為發起請求的客戶端產生一個回應。

:::note 控制器類

關於控制器類別的一點說明：該類別擴充了一個抽象類別 `AbstractController`，而正是這個抽象類別為它提供了控制器的功能。 此外，請記住，在 PSR-4 自動載入時，控制器的命名空間是 `App\Controller` ，因為它位於 `src/Controlle`r 資料夾中。

:::

### HttpFoundation

請求/回應模型完全由 Symfony 軟體包管理：[`symfony/http-foundation`](https://packagist.org/packages/symfony/http-foundation)。

簡而言之，該套件提供了一個物件導向的接口，用於表示 HTTP 請求和回應。 例如，我們不使用超全域變數 `$_GET` 和 `$_POST`，而是使用 `SymfonyComponent\HttpFoundation\Request` 類別的查詢和請求屬性。
