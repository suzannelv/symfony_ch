# 模板（檢視） - Twig - 簡介

在 webapp 版本中，為了建立一個完整的應用程序，Symfony 提供了 [Twig](https://twig.symfony.com/) 作為模板引擎。

## 文法(語法)

需要記住以下 3 個 Twig 語法元素：

-   控制結構或 Twig 語法：`{% %}`
-   評估表達式並在螢幕上顯示結果： `{{ }}`
-   在範本中輸入註解： `{# #}`

## 範例

Twig 檔案範例：

```html
{# base.html.twig #}
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <title>{% block title %}Mon super titre{% endblock %}</title>
        {% block stylesheets %}{% endblock %}
    </head>
    <body>
        {% block body %}{% endblock %} {% block javascripts %}{% endblock %}
    </body>
</html>
```

在任何模板檔案中，我們可以包含用於編譯模板的 Twig 指令。

例如，在第一個摘要中，我們建立了一個基本的 HTML 框架。

這樣做的目的是為我們應用程式中的所有頁面提供一個通用的基本範本。

## 區塊

因此，我們可以在基礎框架中定義不同的**區塊**，並將其提供給子模板，以便它們各自定義自己的內容：

```html
{# index/index.html.twig #} {% extends 'base.html.twig' %} {% block title %} {#
Il est possible de rappeler le contenu du bloc parent avec la fonction parent()
#} {{ parent() }} - Hello TestController! {% endblock %} {% block body %}

<div class="example-wrapper">
    {# On peut afficher la valeur de variables passées par le contrôleur à la
    vue #}
    <h1>Hello {{ controller_name }}! ✅</h1>
</div>
{% endblock %}
```

:::info 繼承
因此，父模板中定義的各種區塊將繼承子模板中定義的內容。 因此，我們可以分別定義每個頁面的內容，同時保持共同的顯示基礎（包含 Bootstrap CSS 和 JS 等）。
:::

## 函數

在[這裡](https://twig.symfony.com/doc/3.x/functions/index.html)可以找到可用函數的列表。 以下是其中幾個：

### block (區塊)

上面使用的 `block` 元素實際上是一個**函數**。 Twig 提供了許多函數，為我們提供了更多可能性，有點像 SPL：

### parent (父級)

在上述模板中也有這個功能：`parent` 函數呼叫父級區塊的內容，以避免子模板始終覆蓋繼承的區塊內容。

### include (包含)

類似於 SPL 中的 `include` 或 `require` ，我們可以使用 Twig 的 `include` 函數**將一個模板包含到另一個模板**。 如果涉及的模板使用變量，甚至可以透過第二個參數傳遞資料給它：

```html
{# Template seul #} {{ include('template.html') }} {# Template auquel on
transmet des données #} {{ include('template.html', { foo: 'bar' }) }}
```

預設情況下，**包含的模板可以存取在包含它的模板中定義的變量**。 透過第三個 boolean 參數 `with_context`，可以停用這種存取權限。

## 控制結構

在 Twig 中，可以使用 `if ... else` 結構以及 `for ... in` 循環結構。

總的來說，我們可以參考 Twig 文件的[這個頁面](https://twig.symfony.com/doc/3.x/templates.html)，了解 Twig 的使用和其功能。 在整個模組中，我們將使用和/或解釋其中的一些功能。

## Symfony 提供的 Twig 擴展

Twig 是一個[可擴展](https://twig.symfony.com/doc/3.x/advanced.html#functions)的模板引擎，這意味著除了引擎中包含的功能外，任何人都可以定義自己的功能。

因此，在包含 `TwigBundle` 的 Symfony 應用程式中，將包含 [`Symfony/twig-bridge`](https://packagist.org/packages/symfony/twig-bridge) 套件。 在這個包中定義了 Twig 的一些擴充。

### 產生帶有路徑(path)的鏈接

例如，`path` 函數用於產生到應用程式路由的超連結。

基本上，Twig 並不了解 Symfony。 是 Symfony 引入了[一系列的擴展](https://symfony.com/doc/5.4/reference/twig_reference.html)，其中包括與 Symfony 的 `Routing` 組件耦合的 `path` 函數。

傳統上，如果我們想在我們的應用程式中顯示一個鏈接，我們可以非常簡單地做到。 例如，到"關於"頁：

```html
<a href="/about">À propos</a>
```

但隨後可能出現的問題是鏈接的**有效性**：在更改相關 `Route` 屬性中控制器的 URL 時，我們是否記得將鏈接更改為 `/about`？

例如，在控制器中 ：

```php
- #[Route('/about', name: 'about')]
+ #[Route('/about-us', name: 'about')]
```

這樣，我們就需要考慮將應用程式中**所有指向 `/about` 的鏈接**更改為 `/about-us`。

為了避免這種情況，我們可以參考應用程式中定義的**路由**。 然後，我們可以讓 Symfony 的路由器（`Routing` 組件）為我們提供與名為 `about` 的路由相關聯的 URL。

這就是 `path` 路徑函數的作用：你可以透過給予路由的名稱來呼叫它，然後它將與路由器通訊（透過[`Routing`組件](https://github.com/symfony/twig-bridge/blob/5.4/Extension/RoutingExtension.php#L14)的 [URL 產生器](https://github.com/symfony/twig-bridge/blob/5.4/Extension/RoutingExtension.php#L48)），以取得對應的 URL。

因此，在 Twig 模板中，可以使用頁面名稱產生頁面鏈接：

```php
<!-- 更新後，href 屬性的值將改為"/about-us"，因此無需修改此標籤。 -->
<a href="{{ path('about') }}">À propos</a>
```

Symfony 定義了許多 Twig [擴充功能](https://symfony.com/doc/5.4/reference/twig_reference.html)來幫助我們寫模板。
