# 操作

由於 API Platform 遵循 REST 原則，因此我們可以在給定資源上啟用/停用我們所選的**操作**。

預設情況下，當我們為一個類別新增 **ApiResource** 屬性時，API Platform 會啟動該資源上的[所有 REST 操作](https://api-platform.com/docs/v2.7/core/operations/)：集合操作（GET、POST）和與單一資源相關的操作（GET、PUT、PATCH、DELETE）。

## 僅啟動少數操作

若要只啟動部分操作，只需在 `ApiResource` 屬性中明確指定即可。 指定要啟動的操作後，只有這些操作會在資源上啟動。 其他操作將停用。

例如，如果我想對我的專案進行除 DELETE ：`

```php
#[ApiResource(
  operations: [
    new Post(),
    new GetCollection(),
    new Get(),
    new Put(),
    new Patch()
  ],
  normalizationContext: ['groups' => ['articles:read']]
)]
#[ApiFilter(BooleanFilter::class, properties: ['visible'])]
#[ApiFilter(SearchFilter::class, properties: ['title' => 'ipartial'])]
class Article
{
}
```

:::note HTTP 動詞
當然，我們當然可以找到通常與 REST API 一起使用的 HTTP 動詞。
:::

## PUT 和 PATCH

似乎 PUT 操作（完全修改，取代資源）和 PATCH 操作（部分修改資源）的行為仍然相同：我們可以發送資源的部分錶示，只有其中的欄位才會被更新。

唯一的區別是 PATCH 情況下應該使用的 `Content-Type` 標頭，它必須設定為 `application/merge-patch+json`。
