# Powershell
## Tips
### ファイル名をまとめて変更（拡張子を含まない）
```powershell
(ls)|%{
    mv $_.Name ($_.BaseName+"_.csv")
}
```