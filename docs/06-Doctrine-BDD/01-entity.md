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
