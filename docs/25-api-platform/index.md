# 應用程式介面平台 - 簡介

我們可以建立全端式的 Symfony 項目，包括預設的 Doctrine、Mailer、Notifier、Serializer 等元件，也可以建立以微服務、API 或控制台的項目。

在後一種情況下，我們需要自行安裝所需的依賴項。 例如，如果我們想使用資料庫，那麼就需要安裝 Doctrine。

當我們使用 JS 函式庫或框架來建立應用程式時，所選的 JS 技術將負責整個介面和與使用者的互動。 因此，我們的 Symfony 項目實際上變成了一個**後端**項目，它不會處理介面。 因此，它只能在 API 模式下工作，而不是全端模式。

:::note 工作原理
在工作原理方面，我們仍然會看到很多相似之處。 例如，我們可以建立控制器，而問題仍然是將請求轉換為回應。 簡而言之，在 API 模式下，我們將擁有一個更輕量級的應用程序，其中預設安裝的組件更少。
:::

## 建立 API 項目

要建立 API 模式的項目，我們將使用 Symfony CLI 工具：

```bash
symfony new my-project --version=5.4
```

由工具創建的項目更加輕量。 我們只能在其中找到 Console 元件、Dotenv 以及與框架相關的一些其他元件。 但是，我們不再像在全端模式下安裝的那樣擁有完整的套件。

如果列出可用的命令，我們甚至沒有 maker。

因此，我們將首先安裝所需的組件。

然後，我們可以像往常一樣使用指令 `symfony serve --no-tls` 啟動項目，但是在首頁看不到 Profiler。

## 組件安裝

為了在本地工作，我們將安裝一些常用元件，例如 Profiler、Doctrine、maker 等...

安裝這些元件後，我們就可以安裝 API 平台。

:::note SYMFONY FLEX
Symfony Flex 已經整合到專案中。 因此，我們將在專案中看到自動執行的 Flex 配方。
:::

## Doctrine

要安裝 Doctrine Bundle，我們可以使用其 Flex 別名：

```bash
composer require orm
```

執行的配方會自動建立 Doctrine 設定檔、遷移、新增 `DATABASE_URL` 環境變數等......這樣我們就可以在 `.env.local` 檔案中設定對資料庫的存取。

## Profiler

可使用 `symfony/profiler-package` 安裝 Profiler。 我們可以使用 `profiler` 別名，並確保將其新增為開發依賴：

```bash
composer require --dev profiler
```

如果重新啟動應用程序，就會在頁面底部發現網頁偵錯工具列。

## maker 工具

可以使用 `maker` 別名安裝 maker：`

```bash
composer require --dev maker
```

## Fixtures

與全端模式一樣

```bash
composer require --dev orm-fixtures
```

如果需要，也可以安裝 Faker。

## Security

若要建立使用者類別並進行驗證和存取控制，可以新增 Symfony 的 security-pack 安全性套件：

```bash
composer require security
```

## 啟動專案

安裝好主要元件後，我們就可以建立實體並啟動專案了。

唯一不同的是，我們不會用 Twig 創建介面：我們將專注於**後端**功能、資料管理等...

:::note Twig
如果專案中安裝了 Twig，那隻是因為安裝了 profiler。
:::

因此，我們要在這個應用程式中建立一個文章、分類、作者等系統。
