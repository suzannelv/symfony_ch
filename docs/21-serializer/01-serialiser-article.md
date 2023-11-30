# 序列化我們的文章

在以 `--webapp` 模式建立的應用程式中，我們**已經**有了 Symfony 的序列化器。

因此，在以 API 模式建立新的應用程式之前，我們可以使用序列化器測試文章。

這種情況非常現實：除了新聞網站之外，你還可以決定為外部客戶端提供一個端點來檢索文章，但不是透過網頁，而是以 JSON 格式的原始輸出。

## 新控制器

因此，我們使用 maker 命令 `make:controller` 在應用程式中建立一個新的控制器類別。

對於類別名，如果在 `src/Controller` 資料夾中已經有一個 `ArticleController`，則可以指定 `Api\ArticleController`。 有了 `PSR-4`，maker 就會知道在哪裡建立新的控制器類，即在 `src/Controller/Api/ArticleController.php` 中。

## 以 JSON 格式發送回應

在控制器中，可以使用 `AbstractController::json` 繼承方法以 JSON 格式傳送結果：

```php
#[Route('/api/articles', name: 'app_api_articles')]
public function index(ArticleRepository $articleRepository): Response
{
  return $this->json($articleRepository->findAll());
}
```

問題是，我們立即收到一個錯誤：`A circular reference has been detected when serializing the object of class...`。

如果我們再深入研究一下，就會發現序列化器不僅要序列化我們的文章，還要序列化文章中的所有內容。 在文章內部，我們有 Category 等其他內容。

但是，當它探索 `Category` 類別物件並試圖序列化它時，它試圖序列化其中的所有內容，包括該類別中的文章。 這是個問題，因為這樣會拋出一個循環引用錯誤，並立即停止進程。

如果我們沒有任何系統來阻止這種情況，那麼序列化器就會在一篇文章、它的類別、這個類別中的文章、這些文章的類別之間無休止地循環，等等..... .

我們需要一種控制序列化的方法。

## 用組控制序列化

要控制序列化，我們需要考慮要傳回的資料格式。

在 JSON 中，我希望傳回一個文章數組，包括每個文章的類別，以避免第二次要求資訊。 例如 ：

```json
#[Route('/api/articles', name: 'app_api_articles')]
public function index(ArticleRepository $articleRepository): Response
{
  return $this->json($articleRepository->findAll());
}
```

有了 Symfony 的序列化器，我可以宣告**序列化群組**，其中包含我需要的屬性。 要聲明某個屬性屬於某個群組，只需使用序列化元件的 PHP8 Groups 屬性即可：

> src/Entity/Article.php

```php
#[ORM\Id]
#[ORM\GeneratedValue]
#[ORM\Column]
#[Groups('articles:read')] // PHP8 新屬性
private ?int $id = null;

#[ORM\Column(length: 255)]
#[Groups('articles:read')] // PHP8 新屬性
private ?string $title = null;

//...

#[ORM\ManyToOne(inversedBy: 'articles')]
#[Groups('articles:read')] // Également sur la catégorie
private ?Category $category = null;
```

:::note 組名稱

群組的名稱完全由使用者自行決定，但通常我們會先標示實體名稱（或 API 中的資源名稱），然後標示相關操作，中間用冒號 `:` 分隔。

例如，可以使用 `articles:read`、`article:read` 或 `articles:item:read`。
:::

接下來，我們將在 `Category` 實體中指明請求 `articles:read` 群組時必須包含的屬性：

> src/Entity/Category.php

```php
#[ORM\Id]
#[ORM\GeneratedValue]
#[ORM\Column]
#[Groups('articles:read')] // Nouvel attribut PHP8
private ?int $id = null;

#[ORM\Column(length: 255)]
#[Groups('articles:read')] // Nouvel attribut PHP8
private ?string $name = null;

#[ORM\OneToMany(mappedBy: 'category', targetEntity: Article::class)]
// 沒有必要說明你想序列化文章
// 否則我們會觸發另一個循環引用錯誤
// 因此，我們可以保留我們的屬性：沒有必要將其包含在群組中。
private Collection $articles;
```

我們要做的就是不將文章包含在我們的操作所涉及的群組中。

## 指定序列化的上下文

最後，我們可以返回控制器，向 `json` 方法指明我們希望在其中進行序列化的**上下文**。

上下文中有許多可用選項。

其中之一就是我們要使用的**組**。 因此，當我們呼叫 json 方法時，我們會告訴序列化器使用 `articles:read` 群組：

```php
#[Route('/api/articles', name: 'app_api_articles')]
public function index(ArticleRepository $articleRepository): Response
{
  return $this->json(
    $articleRepository->findAll(),
    context: ['groups' => 'articles:read']
  );
}
```

:::info `groups` (群組)
實際上，我們可以看到該選項的名稱是 `groups`：我們可以使用陣列向其指明多個群組
:::

如果我們使用 **Postman** 這樣的客戶端來測試我們的端點，結果會是這樣的：

```json
[
    {
        "id": 630,
        "title": "White Rabbit, who said in a helpless sort of a.",
        "date_created": "2021-08-06T00:00:00+00:00",
        "content": "Duchess asked, with another dig of her head down to her lips. 'I know SOMETHING interesting is sure to make out which were the cook, to see anything; then she remembered trying to find it out, we should all have our heads cut off, you know. But do cats eat bats? Do cats eat bats, I wonder?' As she said this, she looked at it again: but he would deny it too: but the cook and the pair of gloves and a great crowd assembled about them--all sorts of things, and she, oh! she knows such a thing.",
        "category": {
            "id": 70,
            "name": "sequi"
        }
    },
    {
        "id": 631,
        "title": "Queen. First came ten soldiers carrying clubs.",
        "date_created": "2022-04-26T00:00:00+00:00",
        "content": "March Hare and the fan, and skurried away into the garden. Then she went out, but it puzzled her very earnestly, 'Now, Dinah, tell me your history, she do.' 'I'll tell it her,' said the March Hare went 'Sh! sh!' and the little door about fifteen inches high: she tried her best to climb up one of them can explain it,' said the Mouse, who was passing at the Lizard in head downwards, and the procession came opposite to Alice, flinging the baby at her feet, they seemed to Alice an excellent.",
        "category": {
            "id": 68,
            "name": "doloremque"
        }
    },
    {
        "id": 632,
        "title": "CHAPTER VI. Pig and Pepper For a minute or two.",
        "date_created": "2021-07-03T00:00:00+00:00",
        "content": "I beg your pardon,' said Alice very meekly: 'I'm growing.' 'You've no right to grow here,' said the Mock Turtle sighed deeply, and began, in a shrill, loud voice, and see that the mouse doesn't get out.\" Only I don't know what it was: she was ever to get through was more and more puzzled, but she did not quite know what they're about!' 'Read them,' said the Pigeon; 'but if you've seen them so often, of course was, how to begin.' He looked anxiously at the bottom of a large caterpillar, that.",
        "category": {
            "id": 70,
            "name": "sequi"
        }
    }
    //...
]
```

## 日期格式

日期的預設格式並不合適。 我們更喜歡 `dd/mm/yyyy` 這樣的格式。

### 當序列化時

我們可以告訴它使用哪種日期格式。 為此，我們將使用 `datetime_format` 選項，該選項在 `DateTimeNormalizer::FORMAT_KEY` 類別中作為公共常量可用：

```php
#[Route('/api/articles', name: 'app_api_articles')]
public function index(ArticleRepository $articleRepository): Response
{
  return $this->json(
    $articleRepository->findAll(),
    context: [
      'groups' => 'articles:read',
      DateTimeNormalizer::FORMAT_KEY => 'd/m/Y'
    ]
  );
}
```

现在的输出格式如下：`"date_created": "06/08/2021"`。

### 直接在屬性上

如果我們想要避免重複，我們也可以一次定義我們希望在序列化器以 JSON 發送的文章中看到的格式。 。

為此，讓我們從來源開始：在 `date_created` 屬性上，我們可以使用 PHP8 的上下文（`Contex`t）屬性來指定參數：

```php
#[ORM\Column(type: Types::DATE_MUTABLE)]
#[Groups('articles:read')]
#[Context(options: [DateTimeNormalizer::FORMAT_KEY => 'd/m/Y'])] // Ajout du format
private ?\DateTimeInterface $date_created = null;
```

結果與之前相同。 差別僅僅在於，我們在呼叫序列化器時不再需要使用這個上下文參數。
