# 模板的分離與包含

讓我們回到 Twig 模板。

與其把所有內容都寫在一個模板中，我們可以把一些模板分開出來，然後導入（inclure）到其他模板中。

## 定義模板

以類別列表為例。 我們可以將產生 Bootstrap 卡片的程式碼分開：

```php
{# templates/category/card.html.twig #}
<div class="card">
  <div class="card-body">
    <h5 class="card-title display-5">
      {{ category.name }}
    </h5>
    <p class="card-text">
      {{ category.articles|length }} articles
    </p>
    <a href="{{ path('category_item', {'id': category.id}) }}" class="btn btn-primary">Voir</a>
  </div>
</div>
```

## 包含

然後將其包含在清單頁面中要顯示的每個類別中。 為此，我們將使用 Twig 的 include 方法：

```php
{# templates/index/index.html.twig #}
{# ... #}
<div class="row row-cols-1 row-cols-md-2 row-cols-lg-3 g-4">
  {% for category in categories %}
  <div class="col">
    {% include 'category/card.html.twig' with {'category': category} %}
  </div>
  {% else %}
    <div>Aucune catégorie trouvée</div>
  {% endfor %}
</div>
{# ... #}
```

:::danger 範本路徑
請注意模板的路徑：它總是相對於在 Twig 軟體包配置（此處為 `templates/`）中定義的模板根目錄。
::

我們將 category 變量注入到包含的模板中，並在其中使用。

因此，我們可以用相同的方法建立導覽欄，例如：一個獨立的範本代表功能表欄，然後在主佈局（`base.html.twig` 檔案）中加入一個 `include` 指令，讓 Twig 將該範本包含在基礎檔案中 。
