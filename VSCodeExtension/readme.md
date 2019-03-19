
# VSCode Extension
## ドキュメントを開く
```
workspace.openTextDocument(uri).then(doc=>window.showTextDocument(doc))
```

## オフラインで画像をWebViewで表示
WebPackでまとめたjsをhtmlにインライン展開して、それをTypeScriptからimportできるようにすればいい。

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

## VSCE Webviewに画像を表示
Webpackとかbase64埋め込みで可能。
Webpackだとurl-loaderを使う。  
limitで大きいサイズを指定しておくと制限に引っかからない。
全体のサイズ制限もこのコードで突破可能
```webpack.config.js
performance: {
    maxEntrypointSize: 10000000,
    maxAssetSize: 10000000
}
```
良い。

# 日記
## 2019.3.12
とうとうvsceをpublishした。  
https://marketplace.visualstudio.com/items?itemName=Y0KUDA.tabkeeper
