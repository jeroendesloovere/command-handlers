# Command + Command handlers

There are a lot of different vendors out there for this:
- `Proof` (we use this one in our project VVSG)
- `SimpleBus`
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
  app.handler.create_user:
    class: App\Domain\User\Command\CreateUser
    arguments:
      - "@app.repository.user"
    tags:
      - { name: command_handler, handles: App\Domain\User\Handler\CreateUserHandler }
```

**Note:** For newer Symfony versions this can automatically work without adding it to a yaml file.

### Create your Command

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

### Create your Handler

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
