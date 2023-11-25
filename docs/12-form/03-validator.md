# 数据验证

数据检索和映射完成后，需要确保数据有效。

这不是 Symfony 的 `Form` 组件的职责，而是 `Validator` 组件的职责。

我们可以简单地调用表单中的两个方法：`isSubmitted` 和 `isValid`，以确保用户已提交表单，且数据有效：

```php
//...
$form->handleRequest($request);

if ($form->isSubmitted() && $form->isValid()) {
  // Si tout va bien, alors on peut persister l'entité et valider les modifications en BDD
  $em->persist($newsletter);
  $em->flush();
}
```

但是，為了正確驗證數據，我們需要為不同欄位指定驗證格式。

例如，我如何告訴我的 `Validator` 元件，我的 `Newsletter` 實體中的 `email` 欄位必須接受電子郵件格式的字串？

我們可以直接在實體層級設定**驗證約束**：

```php
//...
use Symfony\Component\Validator\Constraints as Assert;
//...
class Newsletter
{
  #[ORM\Column(length: 255)]
  #[Assert\Email]
  private ?string $email = null;

  //...
}
```

:::info 驗證約束
驗證約束可以以 PHP8 屬性的形式直接套用於實體的屬性。

在此之前，我們可以使用註解：

```php
/**
 * @ORM\Column(type="string", length=255)
 * @Assert\Email
 */
private $email;
```

:::

若要尋找 Symfony 中可用的所有驗證約束，請查閱[本頁](https://symfony.com/doc/5.4/reference/constraints.html)。

:::note 版本
查閱文件時請注意選擇正確的 Symfony 版本。 這裡我們使用的是 5.4 版，但 Symfony 的最新主要版本是 6 版，這意味著在 5 版和 6 版之間，有些內容可能已經改變了。 預設情況下，文件以最新版本顯示
:::

透過在實體欄位中套用驗證約束， `Validator` 元件可以偵測並報告表單中的格式錯誤。
