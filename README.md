# Command + Command handlers

**Note**: Do not compare this with "console commands", that is something different.

There are a lot of different vendors out there for this:
- [`Prooph`](http://getprooph.org/)
- `SimpleBus` (The following example uses this one)
- ...

## Setting up

This example is minimal and uses SimpleBus:
```bash
composer require "simple-bus/doctrine-orm-bridge"
composer require "simple-bus/symfony-bridge"
```

### Update your services.yml

```yaml
services:
  # Create
  app.handler.create_user:
    class: App\Domain\User\Command\CreateUserHandler
    arguments:
      - "@app.repository.user"
    tags:
      - { name: command_handler, handles: App\Domain\User\Handler\CreateUser }

  # Update
  app.handler.update_user:
    class: App\Domain\User\Command\UpdateUserHandler
    arguments:
      - "@app.repository.user"
    tags:
      - { name: command_handler, handles: App\Domain\User\Handler\UpdateUser }

  # Delete
  app.handler.delete_user:
    class: App\Domain\User\Command\DeleteUserHandler
    arguments:
      - "@app.repository.user"
    tags:
      - { name: command_handler, handles: App\Domain\User\Handler\DeleteUser }
```

**Note:** For newer Symfony versions this can automatically work without adding it to a yaml file.

### Create your Commands

```php
<?php

namespace App\Domain\User\Command;

class CreateUser
{
    /** @var string */
    public $firstName;

    /** @var string */
    public $lastName;
}
```
> Note the public variables.

### Create your Handlers

```php
<?php

namespace App\Domain\User\Handler;

use App\Domain\User\Command\CreateUser;
use App\Infrastructure\Repository\UserRepository;

class CreateUserHandler
{
    /** @var UserRepository */
    private $repository;

    public function __construct(UserRepository $repository)
    {
        $this->repository = $repository;
    }

    public function handle(CreateUser $command)
    {
        $user = new User(
            $command->firstName,
            $command->lastName
        );

        // Save the new user to the database
        $this->repository->persist($user);
    }
}
```

```php
<?php

namespace App\Domain\User\Handler;

use App\Domain\User\Command\CreateUser;
use App\Infrastructure\Repository\UserRepository;

class DeleteUserHandler
{
    /** @var UserRepository */
    private $repository;

    public function __construct(UserRepository $repository)
    {
        $this->repository = $repository;
    }

    public function handle(CreateUser $command)
    {
        $user = $this->repository->getById($command->id);
        $this->repository->remove($user);
    }
}
```


### Example 01

We have a User class.

```php
<?php

namespace App\Domain\User;

class User
{
    private $firstName;
    private $lastName;

    public function __construct(string $firstName, string $lastName)
    {
        $this->firstName = $firstName;
        $this->lastName = $lastName;
    }
}
```

#### Create user

```
$createUser = new CreateUser();
$createUser->firstName = 'Jeroen';
$createUser->lastName = 'Desloovere';

$this->commandBus->handle($createUser);
```

#### Update user

```
$updateUser = new UpdateUser();
$updateUser->firstName = 'John';
$updateUser->lastName = 'Doe';

$this->commandBus->handle($updateUser);
```

#### Delete user

```
$userId = 1;
$this->commandBus->handle(new DeleteUser($userId));
```

### Extending with DataTransferObject

Using a DataTransferObject.
F.e.: `UserDataTransferObject`

```php
<?php

namespace App\Domain\User;

class UserDataTransferObject
{
    /** @var string */
    public $firstName;

    /** @var string */
    public $lastName;
}
```

```php
<?php

namespace App\Domain\User\Command;

class CreateCommand extends UserDataTransferObject
{

}
```

```php
<?php

namespace App\Domain\User\Command;

class UpdateCommand extends UserDataTransferObject
{

}
```