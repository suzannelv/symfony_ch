# Doctrine - 數據模型

## ORM 原則

Doctrine 是一個**ORM（Object-Relational Mapper 物件關係映射器）**工具，讓我們可以定義表示我們**模型**的**類**別：資料庫(數據
庫)。

隨後，Doctrine 將這些類別轉換為資料庫中配置的關係（或表格）。 因此，我們可以專注於物件導向的邏輯，並在與資料庫的通訊中提供協助。

為此，我們將在 Symfony 應用程式中配置對資料庫的存取權限，然後使用控制台 console 建立**實體**：也就是我們資料庫的表。 在實體中，我們將描述類別的不同屬性，這些屬性將被解讀為表格的**欄位**。 我們也可以定義實體之間的關聯（或關係），這些關聯將由 Doctrine 自動管理和解讀。

與這些實體同時創建的還有**倉庫**（repositories）。 倉庫是由 Symfony 註冊的類，我們可以用它來與資料庫通訊（尋找元素，檢索元素列表等）。 因此，我們將每個表都有一個倉庫。

:::caution symfony 與 Doctrine
Doctrine ORM 可以在 Symfony 應用程式之外使用。 我們將在控制台中使用的命令可以幫助我們在 Symfony 框架內執行某些與 Doctrine 相關的任務

:::

## 配置

我們將使用 `DATABASE_URL` 變數在環境變數檔案中輸入資料庫存取詳情

:::caution .env / .env.local 文件
在更新資料庫之前，必須在 .env.local 檔案中輸入資料庫存取 URL，該檔案不包含在版本管理器（[文檔](https://symfony.com/doc/5.4/doctrine.html#configuring-the-database)）中。

:::
在 `.env` 檔案的註解中可以找到 MySQL 的預設範例：

```json
# Format : driver://user:pass@host:port/dbname?serverVersion=X&charset=utf8mb4
DATABASE_URL="mysql://app:!ChangeMe!@127.0.0.1:3306/app?serverVersion=8&charset=utf8mb4"

```

> 與使用 PDO 建立 DSN 時使用的連線資訊大致相同

接下來，我們可以使用以下命令讓 Doctrine 為我們建立資料庫：

```json
php bin/console doctrine:database:create
# Ou, plus court
php bin/console d:d:c

```
