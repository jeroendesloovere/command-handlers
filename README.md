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
    
    /**
     * @return string
     */
    public  function getFirstName() : string
    {
        return $this->firstName;
    }

    /**
     * @return string
     */
    public  function getLastName() : string
    {
        return $this->lastName;
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

### Writing cleaner code using DataTransferObject

Using a DataTransferObject.
F.e.: `UserDataTransferObject`

```php
<?php

namespace App\Domain\User;

class UserDataTransferObject
{
    /**
     * @var User
     */
    protected $userEntity;
    
    /**
     * @var string 
     */
    public $firstName;

    /**
     * @var string 
     */
    public $lastName;
    
    public function __construct(User $user = null)
    {
        $this->userEntity = $user;
        
        if (!$this->hasExistingUser()) {
            return;
        }

        $this->firstName = $this->userEntity->getFirstName();
        $this->lastName = $this->userEntity->getLastName();
    }

    public function getUserEntity() : User
    {
        return $this->userEntity;
    }

    public function hasExistingUser() : bool
    {
        return $this->userEntity instanceof User;
    }
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
    public function __construct(User $user)
    {
        parent::__construct($user);
    }
}
```

We can now change the User class to the following:

```php
<?php

namespace App\Domain\User;

use App\Domain\User\UserDataTransferObject;

class User
{
    private $firstName;
    private $lastName;

    private function __construct(string $firstName, string $lastName)
    {
        $this->firstName = $firstName;
        $this->lastName = $lastName;
    }

    private function fromDataTransferObject(UserDataTransferObject $dataTransferObject) : User
    {
        if ($dataTransferObject->hasExistingUser()) {
            $user = $dataTransferObject->getUserEntity();
            $user->firstName = $dataTransferObject->firstName;
            $user->lastName = $dataTransferObject->lastName;
            
            return $user;
        }

        return new self(
            $dataTransferObject->firstName,
            $dataTransferObject->lastName
        );
    }
    
    /**
     * @return string
     */
    public  function getFirstName() : string
    {
        return $this->firstName;
    }

    /**
     * @return string
     */
    public  function getLastName() : string
    {
        return $this->lastName;
    }
}
```