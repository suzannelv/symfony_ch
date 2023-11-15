# 實體 - 模型

要在我們的應用程式中建立實體，我們將使用 Maker 工具：`php bin/console make:entity`。

命令列助手清晰且易於使用。 為要建立的每個屬性選擇其類型、大小、是否可為空等...

一旦完成建立實體，Maker 將在 `src/Entity` 中為我們建立一個實體類，並在 `src/Repository` 中建立一個儲存庫類別。

## 實體類別

這個類別包含各種屬性，這些屬性將成為表格的列。

屬性是在 `Doctrine\ORM\Mapping` 命名空間中定義的，這裡自動將其別名為 `ORM`。 因此，當我們使用屬性時，我們只需指定 `#[ORM...]`。

Maker 工具已經為我們完成了所有工作：產生了類別的屬性，以及對 Doctrine 非常有用的屬性，使我們能夠隨後在資料庫中建立相應的結構元素。

透過使用的屬性，我們可以看到這一點：例如，Id 屬性用於定義哪個屬性將成為我們表的**主鍵**，而 `GeneratedValue` 將自動配置 ID 在表中的自增。

[點擊這裡](https://www.doctrine-project.org/projects/doctrine-orm/en/2.14/reference/attributes-reference.html)查看 Doctrine 屬性的完整清單。

:::info 屬性/註解

與控制器和 `Route` 屬性一樣，在 PHP8 之前，我們使用註解為屬性定義 Doctrine 元資料。

:::
最後，封裝預設包含在內，因為對於每個屬性，我們可以找到對應的 **getter** 和 **setter** 方法。

## 儲存庫

除了實體類別之外，Maker 還為我們在 `src/Repository` 目錄中產生了另一個類別：一個以實體名為前綴的儲存庫。

我們稍後會看到如何實際使用儲存庫。 目前，我們先了解其一般定義：一個儲存庫允許我們就實體與資料庫互動進行通訊。 因此，我們絕對不希望刪除這個檔案。

透過控制台的一條指令，我們產生了與我們的實體相關的兩個檔案。 但是，如果我們去查看資料庫，就什麼都沒有改變。 實際上，如果我們想用新實體更新資料庫，就必須再次使用控制台。

## 更新數據庫

我們將看到兩種更新數據庫的方法：遷移和即時更新。

在所有情況下，資料庫更新都分**兩個階段**進行：
**準備和審查**要執行的 SQL 程式碼，然後**執行**更新。

:::caution 配置
在更新資料庫之前，必須在 .env.local 檔案中輸入存取資料庫的 URL，該檔案不包含在版本管理員（[文檔](https://symfony.com/doc/current/doctrine.html#configuring-the-database)）中。
:::

## 遷移

遷移由一個檔案表示，該檔案描述了要對數據庫進行的變更（用 SQL 語句）。

因此，遷移次數越多，各種結構變化的痕跡就越多。 這有點像是數據庫結構的**版本管理**。

### 產生遷移

為了生成遷移，我們只需執行 Maker 的以下命令 :

```bash
php bin/console make:migration
```

:::info MAKER
我們開始發現，maker 指令的前綴都是 `make：`
:::

該命令將比較實體類別的內容和資料庫結構的內容，然後在 `migrations` 資料夾中產生一個遷移類，其中包含同步雙方（程式碼和資料庫）所需的 SQL 程式碼。

生成遷移後，**遷移尚未執行**。 因此，資料庫尚未受到影響。

我們最好在產生的檔案中檢查 SQL 程式碼是否與要進行的更新一致。

### 執行遷移

檢查完 SQL 程式碼後，就可以執行更新，也就是執行遷移：

```bash
php bin/console doctrine:migrations:migrate
# 或更短
php bin/console d:m:m
```

在這種情況下，Doctrine 的**遷移包**接管了這個過程：它會檢查已經執行過的遷移，以避免重複執行相同的遷移，並執行需要執行的遷移。

:::info 遷移包
遷移包是[另一個 Doctrine 專案](https://www.doctrine-project.org/projects/doctrine-migrations/en/3.6/index.html)。 在我們的專案中，它透過 `doctrine/doctrine-migrations-bundle` 套件自動整合到了 Symfony 中。
:::

要檢查哪些遷移已經完成，只需參考資料庫即可。 在第一次遷移之後，你會發現一個 `doctrine_migration_versions` 表。 表中包含了剛執行的遷移類別的 FQCN、執行日期和時間。

:::note 遷移

資料庫遷移是管理結構變化的建議方法。 它的主要優點是嚴謹，透過產生遷移類，可以精確、嚴格地進行有針對性的更新。 更重要的是，這些類別與版本控制整合在一起，可以更準確地記錄所做結構變更的歷史。

然而，務必確保不要在不同的結構更新中混淆，同時確保遷移工具也能正確處理。

:::

## 即時更新

其操作方式類似，但不產生任何遷移檔案。 所有操作都在控制台中進行。

檢視要執行的程式碼：`php bin/console doctrine:schema:update --dump-sql`.

即時執行必要的更新：`php bin/console doctrine:schema:update --force`.
