## How to Test Laravel's `Storage::temporaryUrl()`

Laravel provides a powerful and flexible `Storage` facade for file storage and manipulation. One notable feature is `temporaryUrl()`, which generates temporary URLs for files stored on services like Amazon S3 or DigitalOcean Spaces. However, Laravel's documentation does not cover how to test this method effectively. Testing it can pose challenges, particularly when using `Storage::fake`, as the fake storage driver does not support `temporaryUrl()` and throws the error:

> This driver does not support creating temporary URLs.

In this article, we’ll demonstrate two approaches to testing `Storage::temporaryUrl()` using a practical example. These approaches involve mocking the filesystem and using fake storage. Both methods ensure your tests remain isolated and reliable.


### Example Setup

We’ll use a `PriceExport` model, a corresponding controller, and test cases to illustrate the testing process. Here’s the setup:

#### Model
```php
final class PriceExport extends Model
{
    protected $fillable = [
        'user_id',
        'supplier_id',
        'path',
        'is_auto',
        'is_ready',
        'is_send',
    ];

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    public function supplier(): BelongsTo
    {
        return $this->belongsTo(Supplier::class);
    }
}
```

#### Controller
The controller generates a temporary URL for the file using the `temporaryUrl` method:

```php
final class PriceExportController extends Controller
{
    /**
     * @throws ItemNotFoundException
     */
    public function download(PriceExport $priceExport): DownloadFileResource
    {
        if (!$priceExport->is_ready || empty($priceExport->path)) {
            throw new ItemNotFoundException('price export');
        }

        $fileName = basename($priceExport->path);
        $diskS3 = Storage::disk(StorageDiskName::DO_S3->value);

        $url = $diskS3->temporaryUrl($priceExport->path, Carbon::now()->addHour());

        $downloadFileDTO = new DownloadFileDTO($url, $fileName);

        return DownloadFileResource::make($downloadFileDTO);
    }
}
```


### Testing `temporaryUrl()`

#### Test Case 1: Using `Storage::fake`

While `Storage::fake` doesn’t natively support `temporaryUrl`, we can mock the fake storage to simulate the method’s behavior. This approach ensures you can test without needing a real storage service.

```php
final class PriceExportTest extends TestCase
{
    public function test_price_export_download_fake(): void
    {
        // Arrange
        $user = $this->getDefaultUser();
        $this->actingAsFrontendUser($user);

        $supplier = SupplierFactory::new()->create();
        $priceExport = PriceExportFactory::new()->for($user)->for($supplier)->create([
            'path' => 'price-export/price-2025.xlsx',
        ]);

        $expectedUrl = 'https://temporary-url.com/supplier-price-export-2025.xlsx';
        $expectedFileName = basename($priceExport->path);

        $fakeFilesystem = Storage::fake(StorageDiskName::DO_S3->value);

        // Mock the fake filesystem
        $proxyMockedFakeFilesystem = Mockery::mock($fakeFilesystem);
        $proxyMockedFakeFilesystem->shouldReceive('temporaryUrl')->andReturn($expectedUrl);
        Storage::set(StorageDiskName::DO_S3->value, $proxyMockedFakeFilesystem);

        // Act
        $response = $this->postJson(route('api-v2:price-export.price-exports.download', $priceExport));

        // Assert
        $response->assertOk()->assertJson([
            'data' => [
                'name' => $expectedFileName,
                'url' => $expectedUrl,
            ]
        ]);
    }
}
```

#### Test Case 2: Using `Storage::shouldReceive`

This method leverages Laravel’s built-in mocking capabilities to mock the `temporaryUrl` behavior directly.

```php
final class PriceExportTest extends TestCase
{
    public function test_price_export_download_mock(): void
    {
        // Arrange
        $user = $this->getDefaultUser();
        $this->actingAsFrontendUser($user);

        $supplier = SupplierFactory::new()->create();
        $priceExport = PriceExportFactory::new()->for($user)->for($supplier)->create([
            'path' => 'price-export/price-2025.xlsx',
        ]);

        $expectedUrl = 'https://temporary-url.com/supplier-price-export-2025.xlsx';
        $expectedFileName = basename($priceExport->path);

        // Mock the storage behavior
        Storage::shouldReceive('disk')->with(StorageDiskName::DO_S3->value)->andReturnSelf();
        Storage::shouldReceive('temporaryUrl')->andReturn($expectedUrl);

        // Act
        $response = $this->postJson(route('api-v2:price-export.price-exports.download', $priceExport));

        // Assert
        $response->assertOk()->assertJson([
            'data' => [
                'name' => $expectedFileName,
                'url' => $expectedUrl,
            ]
        ]);
    }
}
```

---

### Key Takeaways

1. **Storage::fake Limitation**: The fake storage driver does not support `temporaryUrl`. Use a mocked version of the fake storage to work around this.
2. **Mocking Storage**: Laravel’s `Storage::shouldReceive` simplifies mocking methods like `temporaryUrl` when testing controllers.
3. **Isolation**: Both approaches ensure your tests don’t depend on external services, maintaining fast and reliable tests.

By combining these techniques, you can effectively test `Storage::temporaryUrl()` and ensure your application’s functionality is well-verified.

