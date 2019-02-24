# GIT
## diff3
差分だけじゃなく、共通の先祖が含まれるようになる。  
早速`git config --global merge.conflictstyle diff3`で有効にしよう。  
はっきり言って有効にしない理由がない。  

## diff3向きコンフリクト解消ツール
* Kdiff3
* vimdiff
これ以外知らない・・・

## マージコミットを殺さないリベース
`-p`。`git rebase foo_branch -p`で歴史の構造が保たれたままリベースできる。

# Julia

# CatBoost

# Kaggle

# プレゼンツール

# React Static

# VSCode Extension
## ドキュメントを開く
```
workspace.openTextDocument(uri).then(doc=>window.showTextDocument(doc))
```

## 文字色、背景色
```
let decorator = vscode.window.createTextEditorDecorationType({
 light: {
    backgroundColor: "rgba(58, 70, 101, 0.3)"
 },
 dark: {
   backgroundColor: "rgba(117, 141, 203, 0.3)"
 }
});
let range = new vscode.Range(startPos, endPos);
editor.setDecorations(decorator, [range])
```

## Web view
```
const panel = vscode.window.createWebviewPanel(
   "catCoding",
   "Cat Coding",
   vscode.ViewColumn.One,
   {}
);
panel.webview.html = htmlString;
```



