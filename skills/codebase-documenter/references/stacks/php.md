# PHP Stack Reference

## Detection Files
- `composer.json` (primary)
- `composer.lock`
- `*.php` files

## Public vs Private API

### Visibility Modifiers
```php
class MyClass
{
    // Public: Accessible everywhere
    public string $publicProperty;
    public function publicMethod(): void { }

    // Protected: Accessible in class and subclasses
    protected string $protectedProperty;
    protected function protectedMethod(): void { }

    // Private: Accessible only in this class
    private string $privateProperty;
    private function privateMethod(): void { }
}
```

### Interface Exposure
```php
// Public API is typically defined by interfaces
interface UserServiceInterface
{
    public function find(string $id): ?User;
    public function create(array $data): User;
}
```

### Trait Visibility
```php
trait Timestamps
{
    public \DateTime $createdAt;
    public \DateTime $updatedAt;
}
```

## Documentation Comments

### PHPDoc Format
```php
/**
 * Creates a new user with the given details.
 *
 * This method validates the input, hashes the password,
 * and persists the user to the database.
 *
 * @param string $name The user's display name
 * @param string $email The user's email address
 * @param array{role?: string, active?: bool} $options Additional options
 *
 * @return User The created user instance
 *
 * @throws InvalidArgumentException When name or email is empty
 * @throws DuplicateEmailException When email already exists
 *
 * @example
 * $user = $service->createUser('John', 'john@example.com');
 * echo $user->getId();
 *
 * @see User
 * @see UserRepository::save()
 *
 * @since 1.0.0
 * @deprecated 2.0.0 Use create() instead
 */
public function createUser(string $name, string $email, array $options = []): User
```

### Common PHPDoc Tags
- `@param type $name Description` - Parameter
- `@return type Description` - Return value
- `@throws ExceptionClass Description` - Exception
- `@var type Description` - Property/variable
- `@see reference` - Related item
- `@since version` - Version introduced
- `@deprecated version Description` - Deprecation
- `@example` - Usage example
- `@method returnType name(params)` - Magic method
- `@property type $name` - Magic property

### Class Documentation
```php
/**
 * Manages user authentication and sessions.
 *
 * This service handles login, logout, password reset,
 * and session management.
 *
 * @package App\Auth
 * @author Team <team@example.com>
 */
class AuthService
```

## Type System (PHP 8+)

### Type Declarations
```php
// Scalar types
function process(string $name, int $count, float $rate, bool $active): void

// Nullable types
function find(string $id): ?User

// Union types
function handle(string|int $id): Response|null

// Intersection types
function process(Countable&Iterator $items): void

// Mixed type
function log(mixed $data): void

// Never type (PHP 8.1+)
function fail(): never { throw new Exception(); }
```

### Constructor Property Promotion
```php
class User
{
    public function __construct(
        public readonly string $id,
        public string $name,
        private string $email,
    ) { }
}
```

### Enums (PHP 8.1+)
```php
/**
 * User status values.
 */
enum UserStatus: string
{
    case Active = 'active';
    case Pending = 'pending';
    case Suspended = 'suspended';
}
```

## Error Handling Patterns

### Exception Hierarchy
```php
/**
 * Base exception for application errors.
 */
class AppException extends \Exception
{
    public function __construct(
        string $message,
        public readonly string $errorCode,
        ?\Throwable $previous = null
    ) {
        parent::__construct($message, 0, $previous);
    }
}

class ValidationException extends AppException { }
class NotFoundException extends AppException { }
```

### Error Handling
```php
// Try-catch with multiple types
try {
    $result = $service->process($data);
} catch (ValidationException $e) {
    // Handle validation error
} catch (NotFoundException $e) {
    // Handle not found
} catch (\Exception $e) {
    // Handle other errors
}
```

## Project Structure Conventions

### PSR-4 Autoloading
```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

### Directory Structure
```
src/
  Controller/
  Service/
  Repository/
  Entity/
  Exception/
  DTO/
config/
tests/
public/
  index.php
composer.json
```

### Namespace Conventions
```php
namespace App\Service;
namespace App\Entity;
namespace App\Exception;
```

## Common Patterns to Recognize

### Constructor Injection
```php
class UserService
{
    public function __construct(
        private UserRepository $repository,
        private EventDispatcher $events,
    ) { }
}
```

### Repository Pattern
```php
interface UserRepository
{
    public function find(string $id): ?User;
    public function findByEmail(string $email): ?User;
    public function save(User $user): void;
    public function delete(User $user): void;
}
```

### Factory Methods
```php
class User
{
    public static function create(string $name, string $email): self
    {
        return new self($name, $email);
    }

    public static function fromArray(array $data): self
    {
        return new self($data['name'], $data['email']);
    }
}
```

### Value Objects
```php
final readonly class Email
{
    public function __construct(
        public string $value
    ) {
        if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException('Invalid email');
        }
    }
}
```

### Attributes (PHP 8+)
```php
#[Route('/users', methods: ['GET'])]
public function list(): Response

#[Deprecated('Use newMethod() instead')]
public function oldMethod(): void
```

### Magic Methods
```php
/**
 * @method User findByEmail(string $email)
 * @method User[] findByStatus(string $status)
 * @property-read int $count
 */
class UserRepository
{
    public function __call(string $name, array $arguments): mixed
    {
        // Dynamic method handling
    }
}
```
Document magic methods with @method tags.
