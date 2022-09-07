#Azure Storage Account

## Blob


```cmd
dotnet add package Azure.Storage.Blobs
```

![1.png](./images/storage_account/1.png "")
![1.png](./images/storage_account/2.png "")


```C#
BlobServiceClient blobServiceClient = new BlobServiceClient(_connectionString);
BlobContainerClient containerClient = _blobServiceClient.GetBlobContainerClient(containerName);
BlobClient blobClient = containerClient.GetBlobClient(fileName);
await blobClient.UploadAsync(localFilePath, overwrite: true);
return blobClient.Uri;
```