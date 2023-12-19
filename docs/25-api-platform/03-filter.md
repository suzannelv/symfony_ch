# 過濾器

自動建立 REST 端點固然很好，但我們可能也希望能對資料進行**過濾**。

API Platform 包含一個非常有用的**過濾**系統：我們可以使用 URL 中的 GET 參數直接過濾我們想要的集合。

當然，需要事先在相關類別中配置這些過濾器。

## 僅可見項目

API Platform 包含一個布林過濾器（`BooleanFilter`），可直接套用於資源聲明層級：

```php
#[ApiResource(
  normalizationContext: ['groups' => ['articles:read']]
)]
#[ApiFilter(BooleanFilter::class, properties: ['visible'])]
class Article
{
}
```

:::note 注意
我們可以根據需要在**properties**中指定多個屬性。
:::

然後，只需在客戶端指定我們想要的可見或不可見文章。 而我們只想要可見的文章：

```bash
http://localhost:8000/api/articles?visible=true
```

如果不使用過濾器，所有項目都會顯示。 如果使用篩選器，則可以用 `true` 或 `false` 選擇所需的文章。

## 按名稱或部分名稱

對於類似搜尋引擎的需求，我們也可以使用 `SearchFilter`：

```php
#[ApiResource(
  normalizationContext: ['groups' => ['articles:read']]
)]
#[ApiFilter(BooleanFilter::class, properties: ['visible'])]
#[ApiFilter(SearchFilter::class, properties: ['title' => 'ipartial'])]
class Article
{
}
```

:::info IPARTIAL

'i'表示不區分大小寫。
:::

因此，可以按文章標題進行搜尋：

```bash
http://localhost:8000/api/articles?title=last
```

如果只想取得可見的文章，還可以增加一個額外的 GET 參數：

```bash
http://localhost:8000/api/articles?visible=true&title=last
```

## 按類別名稱（或部分類別名稱）

甚至可以更進一步，透過在類別屬性上套用 `SearchFilter` 來定位其名稱：

```php
#[ORM\ManyToOne(inversedBy: 'articles')]
#[ORM\JoinColumn(nullable: false)]
#[Groups('articles:read')]
#[ApiFilter(SearchFilter::class, properties: ['category.name' => 'ipartial'])]
private ?Category $category = null;
```

然後，進行搜尋：

```bash
http://localhost:8000/api/articles?visible=true&category.name=pariat
```

## 自訂/擴展

API Platform 預設提供了許多過濾器。 與 Symfony 元件的想法相同，API Platform 被設計為**可擴展的**。 因此，如果需要，可以建立自己的過濾器。
