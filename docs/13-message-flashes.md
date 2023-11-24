# Flash 訊息

在任何繼承自 `AbstractController` 類別的控制器中，我們都有一些實用方法，例如，可以將使用者重新導向到新頁面 (`redirectToRoute`)、渲染範本 (`render`)、渲染包含表單的範本 (`renderForm`)，等等。

在建立表格時，以訊息或通知的形式向使用者提供回饋是非常有用的。

由 Symfony 提供並廣泛用於表單的一個系統是 **Flash 訊息**。

## 原理

Flash 訊息的工作原理非常簡單：

-   當你從控制器中加入 Flash 訊息時，它會在會話中加入。
-   如果想要在 Twig 範本中顯示 Flash 訊息，可以使用 Symfony 提供的 `app` 全域變里量。
-   資訊一旦顯示，就會**自動**刪除。 因此，這非常適合用於顯示只需消耗一次的通知。

我們可以新增任何類型的消息，將其歸類為一個類別或其他類別。

## 註冊

摘自我們的通訊註冊表的完整範例：

> `src/Controller/NewsletterController.php`

```php
public function register(
    Request $request,
    EntityManagerInterface $em
): Response {
    $newsletter = new Newsletter();
    $form = $this->createForm(NewsletterType::class, $newsletter);

    $form->handleRequest($request);
    if ($form->isSubmitted() && $form->isValid()) {
        $newsletter->setCreated(new DateTime());
        $em->persist($newsletter);
        $em->flush();
        $this->addFlash('success', 'Votre email a été enregistré, merci');
        return $this->redirectToRoute('app_index');
    }

    return $this->renderForm('newsletter/register.html.twig', [
        'form' => $form
    ]);
}
```

首先，我們會註冊一個閃存訊息。 因此，它將被保存在會話中。

## 顯示

其次（見下文），我們將在模板中使用 app 變量，更具體地說是 `app.flashes` 來存取 Flash 訊息：

> `templates/base.html.twig`

```php
{# FLASH MESSAGES #}
{% for type, messages in app.flashes(['success', 'info', 'warning', 'danger']) %}
  {% for message in messages %}
    <div class="alert alert-{{ type }}" role="alert">
      {{ message }}
    </div>
  {% endfor %}
{% endfor %}
```

:::note TWIG 的 FOR 循環
此處使用的語法允許將類型提取為**鍵**，將關聯的訊息提取為**值**。 此處使用的類型是根據 Bootstrap 中的警報類型製定的。
[Twig 文檔](https://twig.symfony.com/doc/2.x/tags/for.html#iterating-over-keys-and-values)
:::
有關 flash 資訊的更多詳情，請點擊[此處](https://symfony.com/doc/5.4/controller.html#managing-the-session)。

:::note `app` 變量

Twig 範本中使用的 `app` 變量實際上是由 Symfony 新增的，目的是方便我們存取 Symfony 內部元素。 在不使用 Symfony 的情況下使用 Twig 時，它並不存在。

有關 `app` 變量的更多詳情，請點擊[此處](https://symfony.com/doc/5.4/templates.html#the-app-global-variable)。

:::
