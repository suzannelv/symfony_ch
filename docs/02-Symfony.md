# Symfony

如同[官網首頁](https://symfony.com/)所介紹的，Symfony 首先是**一套可重複使用的 PHP 元件**。

Symfony 框架本身將數十個這樣的元件整合在一個精確的結構（樹狀結構）中。 然後，我們在這個結構中建立應用程式。

## 版本

Symfony 也採用 `semver` 版本控制系統，每兩年發布新的主要版本。

在這兩年裡，將發布 5 個次版本：從 0 到 4。

第 4 個次版本將是主要版本的最後一個次版本（例如 3.4、4.4 等），並將與下一個主要版本同時發布。

因此，版本 3.4、4.4 等被稱為 **LTS** 版本或長期支援版本：對這些版本的錯誤修復提供 3 年支持，對安全漏洞提供 4 年支持。

## 安裝 Symfony 5.4 LTS

要安裝 Symfony 5.4 版本專案 ，即 LTS，我們將使用 [Symfony 二進位版本](https://symfony.com/download)（或 Symfony CLI）：

```bash
symfony new my_project_directory --version=5.4 --webapp
```

:::note 注意
Symfony 可以創建兩種類型的應用程式：一種用於創建微服務、API 或控制台應用程序，另一種是完整的 Web 應用程序，包括模板引擎（Twig）。 在後一種情況下，我們可以在建立應用程式的命令列指令中新增 `--webapp` 選項。 安裝的基本步驟保持不變（使用 Composer 套件 `symfony/skeleton`）。

:::
執行此命令時，我們將看到來自 packagist.org 的各種元件在我們的依賴項中被框架自動整合。

## 環境

在 Symfony 應用程式中，我們工作的預設環境是 `dev`。

環境將以一個名為 `APP_ENV` 的環境變量的形式呈現。

:::note 環境變量

環境變量存在於 `.env`, `.env.local` 等文件中。

:::

Symfony 預設提供 3 種環境：`dev`用於開發階段，`test` 用於單元和功能測試等，`prod` 用於生產部署、

這樣我們可以優化應用程式的配置，使其運行得更快。
