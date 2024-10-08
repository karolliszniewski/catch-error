```php
use Magento\Framework\Exception\NoSuchEntityException;
use Magento\Framework\Exception\LocalizedException;

class YourClass
{
    protected $productRepository;
    protected $logger;

    public function __construct(
        \Magento\Catalog\Api\ProductRepositoryInterface $productRepository,
        \Psr\Log\LoggerInterface $logger
    ) {
        $this->productRepository = $productRepository;
        $this->logger = $logger;
    }

    public function getProducts($productIds)
    {
        $successProducts = [];
        $failedProducts = [];

        foreach ($productIds as $productId) {
            try {
                $product = $this->productRepository->getById($productId);
                $successProducts[] = $product;
            } catch (NoSuchEntityException $e) {
                // Product not found
                $failedProducts[$productId] = "Product not found: " . $e->getMessage();
                $this->logger->error("Product not found: " . $e->getMessage(), ['productId' => $productId]);
            } catch (LocalizedException $e) {
                // Handle Magento specific exceptions
                $failedProducts[$productId] = "Error loading product: " . $e->getMessage();
                $this->logger->error("Error loading product: " . $e->getMessage(), ['productId' => $productId]);
            } catch (\Exception $e) {
                // Handle any other unexpected exceptions
                $failedProducts[$productId] = "Unexpected error: " . $e->getMessage();
                $this->logger->critical("Unexpected error loading product: " . $e->getMessage(), [
                    'productId' => $productId,
                    'trace' => $e->getTraceAsString()
                ]);
            }
        }

        return [
            'success' => $successProducts,
            'failed' => $failedProducts
        ];
    }

    // Example usage
    public function executeExample()
    {
        $productIds = [18305, 18306, 18307]; // Add your product IDs here
        $result = $this->getProducts($productIds);

        // Process successful products
        foreach ($result['success'] as $product) {
            // Do something with successfully loaded products
            echo "Successfully loaded product: " . $product->getSku() . "\n";
        }

        // Process failed products
        foreach ($result['failed'] as $productId => $error) {
            echo "Failed to load product {$productId}: {$error}\n";
        }
    }
}
```
