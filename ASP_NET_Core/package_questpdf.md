# [套件]QuestPDF

.Net Core 產生 PDF 檔案的套件，可參考  
- 官網的範例
- 此處寫的心得 [QuestPDF - 支援中文、免費好用的 C# PDF 產生器](https://fullstackladder.dev/blog/2023/02/05/introduction-quest-pdf/)  

可即時看產出結果  

在 IDE 的 Terminal 輸入
```bash
dotnet watch run --project .\GeneratePdf_QuestPdf\GeneratePdf_QuestPdf.csproj
```

Visual Studio Code 輸入
```bash
questpdf-previewer
```

在 linux 環境需要額外裝的 nuget 套件
- HarfBuzzSharp.NativeAssets.Linux
- SkiaSharp.NativeAssets.Linux.NoDependencies