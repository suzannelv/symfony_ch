# 建立表單

第一個範例涉及創建（newsletter）時事通訊。

我們的目標是建立一個簡單的表單，其中包含一個電子郵件字段，並將地址保存到時事通訊表中。

:::info 前提條件
首先，我們需要建立一個負責接收這些資料的實體，然後如上所述更新資料庫模式。
:::
要在專案中建立表單，我們可以使用 `MakerBundle` ：

```bash
php bin/console make:form
```

:::info 實體
我們可以將我們的表單與實體關聯或不關聯，Maker 會詢問我們是否希望這樣做。 在我們的例子中，是 `Newsletter` 實體。

:::

Maker 為我們在 `src/Form` 資料夾中產生了一個類別。 因此，我們可以看到我們的表格類別將與程式碼的其餘部分很好地分開，位於 `App\Form` 命名空間中。

:::note 注意
我們在表單類別的名稱後綴中加上 `Type` ，因為在 Symfony 中，所有東西都被稱為"form type"：表格、欄位等等...這樣更方便組合和整合表單。
:::

在這個類別中，我們找到一個關鍵的方法：`buildForm`。 在這個方法中，我們將聲明表單的不同欄位及其屬性：

```php
//...
use Symfony\Component\Form\Extension\Core\Type\EmailType;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;
//...

class NewsletterType extends AbstractType
{
  public function buildForm(FormBuilderInterface $builder, array $options)
  {
      $builder
          ->add('email', EmailType::class)
          ->add("S'inscrire", SubmitType::class)
      ;
  }
  // ...configureOptions, liaison du formulaire à la classe d'entité
}
```

在 `buildForm` 方法中可以注意到以下幾點：

-   我們可以使用 `FormBuilderInterface` 類型的流暢介面來呼叫 `add` 方法。
-   每個聲明的欄位本身都有一個 `Type` 類型：本例中為電子郵件類型（`EmailType`）和提交類型（`SubmitType`）。
-   `EmailType` 、`SubmitType` 類別都是 Symfony 的 Form 元件的一部分，確保使用（`use`）正確的命名空間引入這些類別。
-   我們可以看到，預設情況下，表單不會自動產生提交按鈕。 在建立表單時，我們需要新增一個 `SubmitType` 類型的欄位。

:::note 注意
應用於我們表單的各個欄位的 `***Type` 類別將確保在 Twig 渲染中正確顯示這些欄位！
:::
