# az204-UsingTheAzureBlobStorageClientLibraryFor.NETv11

## 專案概述
此專案展示如何使用 Microsoft Azure Blob Storage 用戶端程式庫（版本 11.2.3）來執行基本的 Blob 儲存操作，例如建立容器、上傳檔案、列出 Blob、下載檔案以及刪除資源。

## 程式碼結構
專案的主要程式碼位於 `Program.cs` 檔案中，並包含以下主要功能：

### 1. `Main` 方法
```csharp
public static void Main()
{
    Console.WriteLine("Azure Blob Storage Demo \n");
    // 執行非同步範例程式碼，並等待結果
    ProcessAsync().GetAwaiter().GetResult();
    Console.WriteLine("\n\n Press enter to exit the sample application.");
    Console.ReadLine();
}
```
- **功能**：作為應用程式的進入點，負責初始化並呼叫 `ProcessAsync` 方法來執行 Blob 儲存操作。
- **說明**：此方法會顯示歡迎訊息，並在操作完成後等待使用者按下 Enter 鍵以結束應用程式。

---

### 2. `ProcessAsync` 方法
`ProcessAsync` 是執行所有 Blob 儲存操作的核心方法，包含以下步驟：

#### a. 驗證連線字串
```csharp
string storageConnectionString = "DefaultEndpointsProtocol=https;AccountName=_______________;AccountKey=____________________==;EndpointSuffix=core.windows.net";
CloudStorageAccount storageAccount;
if (CloudStorageAccount.TryParse(storageConnectionString, out storageAccount))
{
    Console.WriteLine("Valid connection string.\r\n");
    CloudBlobClient cloudBlobClient = storageAccount.CreateCloudBlobClient();
    CloudBlobContainer cloudBlobContainer = cloudBlobClient.GetContainerReference("demoblobs" + Guid.NewGuid().ToString());
    await cloudBlobContainer.CreateAsync();
}
else
{
    Console.WriteLine("A valid connection string has not been defined in the storageConnectionString variable.");
    Console.WriteLine("\n\n Press enter to exit the application.");
    Console.ReadLine();
}
```
- **功能**：檢查 Azure 儲存連線字串的有效性，並建立 Blob 用戶端和容器。
- **說明**：如果連線字串無效，程式會提示使用者並結束應用程式。

---

#### b. 建立本地檔案並上傳至 Blob
```csharp
string localPath = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
string localFileName = "BlobDemo_" + Guid.NewGuid().ToString() + ".txt";
string sourceFile = Path.Combine(localPath, localFileName);
File.WriteAllText(sourceFile, "Hello, World!");
Console.WriteLine("\r\nTemp file = {0}", sourceFile);
Console.WriteLine("Uploading to Blob storage as blob '{0}'", localFileName);

CloudBlockBlob cloudBlockBlob = cloudBlobContainer.GetBlockBlobReference(localFileName);
await cloudBlockBlob.UploadFromFileAsync(sourceFile);
Console.WriteLine("\r\nVerify the creation of the blob and upload in the portal.");
Console.WriteLine("Press 'Enter' to continue.");
Console.ReadLine();
```
- **功能**：在本地建立一個臨時檔案，並將其上傳至 Azure Blob 儲存。
- **說明**：此步驟會顯示檔案的本地路徑，並提示使用者確認上傳結果。

---

#### c. 列出容器中的 Blob
```csharp
Console.WriteLine("List blobs in container.");
BlobContinuationToken blobContinuationToken = null;
do
{
    var results = await cloudBlobContainer.ListBlobsSegmentedAsync(null, blobContinuationToken);
    blobContinuationToken = results.ContinuationToken;
    foreach (IListBlobItem item in results.Results)
    {
        Console.WriteLine(item.Uri);
    }
} while (blobContinuationToken != null);
```
- **功能**：列出指定容器中的所有 Blob。
- **說明**：此步驟會逐一列出 Blob 的 URI，並提示使用者繼續操作。

---

#### d. 下載 Blob 至本地檔案
```csharp
string destinationFile = sourceFile.Replace(".txt", "_DOWNLOADED.txt");
Console.WriteLine("Downloading blob to {0}", destinationFile);
await cloudBlockBlob.DownloadToFileAsync(destinationFile, FileMode.Create);
Console.WriteLine("\r\nLocate the local file to verify it was downloaded.");
Console.WriteLine("Press 'Enter' to continue.");
Console.ReadLine();
```
- **功能**：將指定的 Blob 下載到本地檔案。
- **說明**：下載的檔案會附加 `_DOWNLOADED` 字串以區分於原始檔案。

---

#### e. 清理資源
```csharp
Console.WriteLine("Press the 'Enter' key to delete the example files and example container.");
Console.ReadLine();
Console.WriteLine("Deleting the container");
if (cloudBlobContainer != null)
{
    await cloudBlobContainer.DeleteIfExistsAsync();
}
Console.WriteLine("Deleting the source, and downloaded files\r\n");
File.Delete(sourceFile);
File.Delete(destinationFile);
```
- **功能**：刪除建立的容器和本地檔案。
- **說明**：此步驟確保應用程式執行後不會留下任何臨時資源。

---

## 專案依賴
- **NuGet 套件**：`Microsoft.Azure.Storage.Blob` (版本 11.2.3)
- **目標框架**：.NET 8

`st.csproj` 檔案內容：
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.Azure.Storage.blob" Version="11.2.3" />
  </ItemGroup>
</Project>
```

---

## 執行方式
1. 確保已安裝 .NET 8 SDK。
2. 在專案目錄中執行以下命令以還原依賴項：
   ```bash
   dotnet restore
   ```
3. 編譯並執行專案：
   ```bash
   dotnet run
   ```

---

## 注意事項
- 請將程式碼中的 `storageConnectionString` 替換為有效的 Azure 儲存連線字串。
- 確保 Azure 帳戶具有足夠的權限來執行 Blob 儲存操作。

---

此說明文件提供了專案的完整背景與程式碼解釋，適合用於學習或快速上手 Azure Blob 儲存操作。
