# 關係

使用 ORM，我們將以物件形式設計我們的資料庫。

這意味著在我們正在建立的範例中，即新聞網站，我們建立了一個 `Article` 實體，對應我們資料庫中的 `article` 表。

如果我們希望我們的文章有一個類別，那麼在資料庫中，我們知道這種需求對應於一個 1-N 的關聯，需要在 `article` 表中添加一個指向類別表的外鍵。

在 Doctrine 中，這種關聯以**關係**的形式表示。 每個可用的關係都代表著 MCD/MLD 中存在的一種關聯類型。

## 新增 OneToMany 關係

使用 Doctrine，我們將建立一個 `Category` 實體，但目標是以以下方式操作我們的類別實例，例如：如果我有一個 `Category` 實例，我想透過 `$category->getArticles()` 取得該類別的所有文章。

同樣，如果我查看一篇文章，即我有一個 `Article` 實例，我希望透過 `$article->getCategory()`來取得它的類別。

所有這些都可以由 maker 自動管理，並在正確的位置寫入相應的程式碼。 我們需要了解在兩個實體之間創建關係時添加了哪些內容。

因此，我們將在 `Category` 實體中建立一個 `articles` 屬性。 但在 maker 中，我們可以將此屬性定義為一個**關係**，更準確地說是一個與 `Article` 實體關聯的 **OneToMany** 關係。

助理也會詢問我們是否要在 `Article` 中建立一個新的屬性，以建立**雙向**關係，使我們能夠檢索類別的文章以及文章的類別。 我們確認在 `Article` 實體中建立一個 category 屬性。

然後創建`Category`實體，並且 `Article` 實體相應地被修改：

```php
class Article
{
  //...

  #[ORM\ManyToOne(inversedBy: 'articles')]
  private ?Category $category = null;

  //...

  public function getCategory(): ?Category
  {
    return $this->category;
  }

  public function setCategory(?Category $category): self
  {
    $this->category = $category;

    return $this;
  }
}
```

```php
class Category
{
  //...

  #[ORM\OneToMany(mappedBy: 'category', targetEntity: Article::class)]
  private Collection $articles;

  public function __construct()
  {
      $this->articles = new ArrayCollection();
  }

  //...

  /**
   * @return Collection<int, Article>
   */
  public function getArticles(): Collection
  {
      return $this->articles;
  }

  public function addArticle(Article $article): self
  {
      if (!$this->articles->contains($article)) {
          $this->articles->add($article);
          $article->setCategory($this);
      }

      return $this;
  }

  public function removeArticle(Article $article): self
  {
      if ($this->articles->removeElement($article)) {
          // set the owning side to null (unless already changed)
          if ($article->getCategory() === $this) {
              $article->setCategory(null);
          }
      }

      return $this;
  }
}

```

:::note 文章集合
articles 屬性被創建並從 Doctrine 的角度被視為 ArrayCollection，即物件的集合，可以添加/刪除元素。

:::

因此，我們注意到，從我們的角度來看，即 Symfony 應用程序，我們只專注於管理我們的物件。 在 Article 類別中沒有任何地方引用到指向 Category 表的外鍵。 相反，我們有一個類型為 Category 的屬性。 如果需要，Doctrine 將負責檢索相應的類別。

由於我們剛剛創建了一個新的實體，我們需要影響我們的資料庫以應用這些變更：我們將產生一個新的遷移，然後執行它以觸發新表的建立。

在執行遷移時，Doctrine 將自動修改 article 表以新增指向新表 category 的外鍵。 因此，外鍵確實存在於我們的資料庫中，但我們不需要透過程式碼手動管理它。 我們嘗試管理的是物件之間的關係。
