# 資產

在安裝 Symfony 專案時，[**symfony/asset**](https://packagist.org/packages/symfony/asset) 套件將會自動安裝。 透過這個套件以及 Twig 範本中提供的 `asset` 函數，我們可以管理在頁面中包含靜態檔案（CSS、JS、images 等）。

如果我們想要建立一個 CSS 樣式表，我們可以在專案根目錄下的 `public` 資料夾中建立一個 `css` 資料夾。

:::note `public/`文件夾

`public/` 資料夾位於專案的根目錄，是我們伺服器啟動時公開的**根目錄**。

例如，如果輸入 **http://localhost:8000/css/styles.css**，我會讀取位於 `public/css/styles.css` 的檔案。
:::

在 `public/css` 資料夾中，我們將建立一個 `index.css` 文件，為測試良好的功能, 我們輸入測試用的 CSS:

```css
body {
    background-color: #ddd;
}
```

最後，為了使用 Symfony 的`資產`組件，我們將在應用程式的基礎範本中使用 Twig 擴充功能來整合我們的樣式表。 在 `base.html.twig` 檔案的 `stylesheets` 表塊 ：

```css
{% block stylesheets %}
  {# En-dessous de la balise qui intègre Bootstrap... #}
  <link rel="stylesheet" href="{{ asset('css/index.css') }}">
{% endblock %}
```

現在可以使用 `symfony serve --no-tls` 重新啟動伺服器，並檢查主頁背景是否為淺灰色。 資產元件會根據 `public/` 目錄自動包含檔案。

## 分組資產

我們可以在一個**package**中將資產組合在一起，並為它們定義一個基本目錄，例如 ：

> config/packages/framework.yaml

```yaml
framework:
    assets:
        packages:
            css:
                base_path: "/css"
            images:
                base_path: "/images"
```

接下來，可以使用 Twig 範本來指明想為哪個軟體包添加資產：

```html
<link rel="stylesheet" href="{{ asset('styles.css', 'css') }}" />
```

## 資產版本化

可以在 `config/packages/framework.yaml` 檔案中設定版本控制策略，以及指定版本應該以何種格式整合到資產（asset）的 URL 中：

```yaml
framework:
    assets:
        version: "v1"
        version_format: "%%s?v=%%s"
```

因此，當我們更新時，可以更改 URL 中包含的版本，例如，強制我們重新載入樣式表。

-   有關資產組件的更多信息，請點擊[此處](https://symfony.com/doc/5.4/components/asset.html)
-   有關配置參數的更多信息，請點擊[此處](https://symfony.com/doc/current/reference/configuration/framework.html#assets)

:::info WEBPACK ENCORE
為了管理 Symfony 應用程式的前端，Symfony 的貢獻者開發了一個完整的 [Webpack Encore](https://symfony.com/doc/5.4/frontend.html) 軟體包。 該軟體包使用 Javascript 工具，因此不是本模組的重點。

此外，如果我們使用 JS 函式庫或框架來管理前端，那麼我們只需要 Symfony 來管理 API，而不是介面。這也是我們不深入研究這方面的原因。

如果您有興趣，不妨花點時間查看！
:::
