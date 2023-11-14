# Composer - 回顧

## 依賴關係

Composer 是一個依賴管理器，允許我們聲明希望在應用程式中使用的外部軟體包。

例如我們可以使用 Composer 來建立 Symfony 應用程式。

我們也可以使用 Composer 匯入單獨的依賴項（例如僅導入 Symfony 元件）。

## 自動載入

Composer 還可以幫助我們管理如何載入應用程式的類別。

例如，如果我們希望使用 `PSR-4`，可以在 `composer.json` 設定檔中指定。

> composer.json

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

在這裡，我們指定了"App"命名空間對應於我們應用程式的"src/"資料夾。 因此，它將充當一個“根”命名空間，並遵循我們應用程式的目錄結構。

## 版本控制

我們採用 PHP 套件的 [`semver`](https://devhints.io/semver) 版本控制策略。

`Semver `版本控制由左至右分為三個部分：

-   主版本號（引入與先前版本相比較顯著的更改，可能刪除一些先前的功能）
-   次版本號（可在次版本中引入新功能，確保所有已存在的內容仍然正常運作）
-   補丁版本號（修復錯誤和安全性問題）

Composer 套件的大多數都可以在 [packagist.org](https://packagist.org/) 上找到。
