# 獲取 Symfony 在线项目

讓我們總結一下迄今為止所做的工作：

-   建立一個專案（`symfony new..`.）
-   在這個專案中建立頁面（控制器）（`php bin/console make:controller` 建立一類控制器，或在一類控制器中建立帶有 `Route` 屬性的方法）
-   在 Twig 範本中編寫內容
-   配置存取資料庫的憑證（`.env.local`）
-   建立資料庫、建立實體、建立遷移，然後運行它
-   編寫 fixtures，然後執行 fixtures 將測試資料插入資料庫

但如果我們將 Symfony 專案發佈到 Github 倉庫上呢？

:::note 注意
以下步驟可能因項目而異。 例如，可能沒有 fixtures 或遷移......請根據相關項目調整步驟。
:::

## 克隆項目

最簡單的方法莫過於使用 `git clone` 指令在本機上取得項目。

## 安裝依賴項

請記住，由 Composer 管理的 `vendor` 檔案**沒有**版本控制。 因此，專案恢復後需要重新建立該資料夾。

若要使用與 Github 上發佈的軟體包版本**完全相同**的版本重新建立該資料夾，請使用 `Composer install` 命令：

```bash
composer install
```

:::tip COMPOSER.LOCK

`compose install` 指令會直接搜尋 `compose.lock` 文件，以獲得準確的軟體包版本。

:::

## 配置資料庫訪問

正如我們在 Doctrine 一章中所看到的，要配置對資料庫的訪問，我們需要在 `.env.local` 檔案中定義自己的 `DATABASE_URL` 作為環境變數。

## 建立資料庫

在環境變數中配置好存取權限後，我們需要建立用於承載我們的結構的資料庫 :

```bash
php bin/console d:d:c
```

:::note 連接

這一步驟也用於檢查環境變數中輸入的存取權限是否正確，以便應用程式能夠連接到 MySQL 伺服器。

:::

## 執行遷移

建立資料庫後，Doctrine 需要建立其**結構**：表格和不同表格之間的連結。

如果我們已經克隆了項目，那麼遷移可能已經存在於 `migrations` 資料夾中。 因此，我們將直接運行它們：

```bash
php bin/console d:m:m
```

## 載入測試數據（fixtures）

最後，建立資料庫並載入結構後，我們將導入測試資料：fixtures。

與遷移一樣，fixtures 可能已經存在於程式碼中，因此我們可以直接執行它們：

```bash
php bin/console d:f:l
```
