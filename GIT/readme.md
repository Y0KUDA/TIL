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

## empty commit
第一コミットは空の方がいいよね(潔癖)  
`git commit --allow-empty -m "first commit"`
