資產

在安裝 Symfony 專案時，symfony/asset 套件將會自動安裝。 透過這個套件以及 Twig 範本中提供的 asset 函數，我們可以管理在頁面中包含靜態檔案（CSS、JS、映像等）。

如果我們想要建立一個 CSS 樣式表，我們可以在專案根目錄下的 public 資料夾中建立一個 css 資料夾。

public/文件夾

public/資料夾位於專案的根目錄，是我們伺服器啟動時公開的根目錄。

例如，如果我鍵入 http://localhost:8000/css/styles.css，我將存取位於 public/css/styles.css 的檔案。

在 public/css 資料夾中，我們將建立一個 index.css 文件，並為測試良好的功能輸入測試用的 CSS
