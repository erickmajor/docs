<div class='article-menu'>
  <ul>
    <li>
      <a href="#overview">访问控制列表 (ACL)</a>
      <ul>
        <li>
          <a href="#setup">创建 ACL</a>
        </li>
        <li>
          <a href="#adding-roles">将角色添加到 ACL</a>
        </li>
        <li>
          <a href="#adding-resources">添加资源</a>
        </li>
        <li>
          <a href="#access-controls">定义访问控制</a>
        </li>
        <li>
          <a href="#querying">查询ACL</a>
        </li>
        <li>
          <a href="#function-based-access">Function based access</a>
        </li>
        <li>
          <a href="#objects">对象作为角色名称和资源名称</a>
        </li>
        <li>
          <a href="#roles-inheritance">角色继承</a>
        </li>
        <li>
          <a href="#serialization">序列化 ACL 列表</a>
        </li>
        <li>
          <a href="#events">事件</a>
        </li>
        <li>
          <a href="#custom-adapters">实现自己的适配器</a>
        </li>
      </ul>
    </li>
  </ul>
</div>

<a name='overview'></a>

# 访问控制列表 (ACL)

`Phalcon\Acl` 提供了简单、 轻量级的 Acl，以及附属于他们的权限管理。 [访问控制列表](http://en.wikipedia.org/wiki/Access_control_list)(ACL) 允许应用程序对其领域与基础对象的访问请求进行控制。 我们鼓励您阅读更多关于 ACL 的方法，以更熟悉它的概念。

总之，Acl 有角色和资源。 资源是通过ACLs定义的权限对象。 角色是通过ACL机制确定访问资源的请求是否被拒绝或允许的对象。

<a name='setup'></a>

## 创建 ACL

此组件最初被设计工作在内存中。 这提供了易用性和快速的访问列表中的每个方面。 `Phalcon\Acl` 构造函数作为适配器的第一个参数用于检索控制列表中信息。 下面是使用内存适配器示例︰

```php
<?php

use Phalcon\Acl\Adapter\Memory as AclList;

$acl = new AclList();
```

默认情况下 `Phalcon\Acl` 允许访问尚未定义的资源。 为了提高访问列表中的安全级别，我们可以定义 `deny` 级别，作为默认访问级别。

```php
<?php

use Phalcon\Acl;

// Default action is deny access
$acl->setDefaultAction(
    Acl::DENY
);
```

<a name='adding-roles'></a>

## 将角色添加到 ACL

角色是能或不能访问控制列表里面的某些资源的对象。 举例说明，我们将定义一个公司一组人员的的角色。 `Phalcon\Acl\Role` 类是可用于创建角色更结构化的方式。 让我们添加一些角色到我们最近创建的列表︰

```php
<?php

use Phalcon\Acl\Role;

// 创建一些角色.
// 第一个参数是名称，第二个参数是可选的说明。
$roleAdmins = new Role('Administrators', 'Super-User role');
$roleGuests = new Role('Guests');

// Add 'Guests' role to ACL
$acl->addRole($roleGuests);

// Add 'Designers' role to ACL without a Phalcon\Acl\Role
$acl->addRole('Designers');
```

正如你所看到的不使用实例的情况下直接定义角色。

<a name='adding-resources'></a>

## 添加资源

资源是的对象的访问控制。 通常在 MVC 应用程序中的资源引用到控制器。 虽然这不是强制性的 `Phalcon\Acl\Resource` 类可以用于定义资源。 它是重要的是将相关的操作添加到资源以便 ACL 可以理解它应控制。

```php
<?php

use Phalcon\Acl\Resource;

// Define the 'Customers' resource
$customersResource = new Resource('Customers');

// Add 'customers' resource with a couple of operations

$acl->addResource(
    $customersResource,
    'search'
);

$acl->addResource(
    $customersResource,
    [
        'create',
        'update',
    ]
);
```

<a name='access-controls'></a>

## 定义访问控制

Now that we have roles and resources, it's time to define the ACL (i.e. which roles can access which resources). This part is very important especially taking into consideration your default access level `allow` or `deny`.

```php
<?php

// Set access level for roles into resources

$acl->allow('Guests', 'Customers', 'search');

$acl->allow('Guests', 'Customers', 'create');

$acl->deny('Guests', 'Customers', 'update');
```

The `allow()` method designates that a particular role has granted access to a particular resource. The `deny()` method does the opposite.

<a name='querying'></a>

## 查询的 ACL

Once the list has been completely defined. We can query it to check if a role has a given permission or not.

```php
<?php

// Check whether role has access to the operations

// Returns 0
$acl->isAllowed('Guests', 'Customers', 'edit');

// Returns 1
$acl->isAllowed('Guests', 'Customers', 'search');

// Returns 1
$acl->isAllowed('Guests', 'Customers', 'create');
```

<a name='function-based-access'></a>

## Function based access

此外可以添加 4 参数作为您自定义的函数必须返回布尔值。 当您使用 `isAllowed()` 方法时，它将被调用。 You can pass parameters as associative array to `isAllowed()` method as 4th argument where key is parameter name in our defined function.

```php
<?php
// Set access level for role into resources with custom function
$acl->allow(
    'Guests',
    'Customers',
    'search',
    function ($a) {
        return $a % 2 === 0;
    }
);

// Check whether role has access to the operation with custom function

// Returns true
$acl->isAllowed(
    'Guests',
    'Customers',
    'search',
    [
        'a' => 4,
    ]
);

// Returns false
$acl->isAllowed(
    'Guests',
    'Customers',
    'search',
    [
        'a' => 3,
    ]
);
```

Also if you don't provide any parameters in `isAllowed()` method then default behaviour will be `Acl::ALLOW`. 您可以更改它的使用方法 `setNoArgumentsDefaultAction()`。

```php
<?php

use Phalcon\Acl;

// Set access level for role into resources with custom function
$acl->allow(
    'Guests',
    'Customers',
    'search',
    function ($a) {
        return $a % 2 === 0;
    }
);

// Check whether role has access to the operation with custom function

// Returns true
$acl->isAllowed(
    'Guests',
    'Customers',
    'search'
);

// Change no arguments default action
$acl->setNoArgumentsDefaultAction(
    Acl::DENY
);

// Returns false
$acl->isAllowed(
    'Guests',
    'Customers',
    'search'
);
```

<a name='objects'></a>

## 对象作为角色名称和资源名称

您可以将对象作为 `角色名` 和 `资源名称` 传递。 Your classes must implement `Phalcon\Acl\RoleAware` for `roleName` and `Phalcon\Acl\ResourceAware` for `resourceName`.

Our `UserRole` class

```php
<?php

use Phalcon\Acl\RoleAware;

// Create our class which will be used as roleName
class UserRole implements RoleAware
{
    protected $id;

    protected $roleName;

    public function __construct($id, $roleName)
    {
        $this->id       = $id;
        $this->roleName = $roleName;
    }

    public function getId()
    {
        return $this->id;
    }

    // Implemented function from RoleAware Interface
    public function getRoleName()
    {
        return $this->roleName;
    }
}
```

And our `ModelResource` class

```php
<?php

use Phalcon\Acl\ResourceAware;

// Create our class which will be used as resourceName
class ModelResource implements ResourceAware
{
    protected $id;

    protected $resourceName;

    protected $userId;

    public function __construct($id, $resourceName, $userId)
    {
        $this->id           = $id;
        $this->resourceName = $resourceName;
        $this->userId       = $userId;
    }

    public function getId()
    {
        return $this->id;
    }

    public function getUserId()
    {
        return $this->userId;
    }

    // Implemented function from ResourceAware Interface
    public function getResourceName()
    {
        return $this->resourceName;
    }
}
```

Then you can use them in `isAllowed()` method.

```php
<?php

use UserRole;
use ModelResource;

// Set access level for role into resources
$acl->allow('Guests', 'Customers', 'search');
$acl->allow('Guests', 'Customers', 'create');
$acl->deny('Guests', 'Customers', 'update');

// Create our objects providing roleName and resourceName

$customer = new ModelResource(
    1,
    'Customers',
    2
);

$designer = new UserRole(
    1,
    'Designers'
);

$guest = new UserRole(
    2,
    'Guests'
);

$anotherGuest = new UserRole(
    3,
    'Guests'
);

// Check whether our user objects have access to the operation on model object

// Returns false
$acl->isAllowed(
    $designer,
    $customer,
    'search'
);

// Returns true
$acl->isAllowed(
    $guest,
    $customer,
    'search'
);

// Returns true
$acl->isAllowed(
    $anotherGuest,
    $customer,
    'search'
);
```

Also you can access those objects in your custom function in `allow()` or `deny()`. They are automatically bind to parameters by type in function.

```php
<?php

use UserRole;
use ModelResource;

// Set access level for role into resources with custom function
$acl->allow(
    'Guests',
    'Customers',
    'search',
    function (UserRole $user, ModelResource $model) { // User and Model classes are necessary
        return $user->getId == $model->getUserId();
    }
);

$acl->allow(
    'Guests',
    'Customers',
    'create'
);

$acl->deny(
    'Guests',
    'Customers',
    'update'
);

// Create our objects providing roleName and resourceName

$customer = new ModelResource(
    1,
    'Customers',
    2
);

$designer = new UserRole(
    1,
    'Designers'
);

$guest = new UserRole(
    2,
    'Guests'
);

$anotherGuest = new UserRole(
    3,
    'Guests'
);

// Check whether our user objects have access to the operation on model object

// Returns false
$acl->isAllowed(
    $designer,
    $customer,
    'search'
);

// Returns true
$acl->isAllowed(
    $guest,
    $customer,
    'search'
);

// Returns false
$acl->isAllowed(
    $anotherGuest,
    $customer,
    'search'
);
```

You can still add any custom parameters to function and pass associative array in `isAllowed()` method. Also order doesn't matter.

<a name='roles-inheritance'></a>

## Roles Inheritance

You can build complex role structures using the inheritance that `Phalcon\Acl\Role` provides. Roles can inherit from other roles, thus allowing access to supersets or subsets of resources. There are two ways to use role inheritance:

1. You can pass the inherited role as the second parameter of the method call, when adding that role in the list.

```php
<?php

use Phalcon\Acl\Role;

// ...

// Create some roles

$roleAdmins = new Role('Administrators', 'Super-User role');

$roleGuests = new Role('Guests');

// Add 'Guests' role to ACL
$acl->addRole($roleGuests);

// Add 'Administrators' role inheriting from 'Guests' its accesses
$acl->addRole($roleAdmins, $roleGuests);
```

2. You can setup the relationships after roles are added

```php
<?php

use Phalcon\Acl\Role;

// Create some roles
$roleAdmins = new Role('Administrators', 'Super-User role');
$roleGuests = new Role('Guests');

// Add Roles to ACL
$acl->addRole($roleGuests);
$acl->addRole($roleAdmins);

// Have 'Administrators' role inherit from 'Guests' its accesses
$acl->addInherit($roleAdmins, $roleGuests);
```

<a name='serialization'></a>

## Serializing ACL lists

To improve performance `Phalcon\Acl` instances can be serialized and stored in APC, session, text files or a database table so that they can be loaded at will without having to redefine the whole list. You can do that as follows:

```php
<?php

use Phalcon\Acl\Adapter\Memory as AclList;

// ...

// Check whether ACL data already exist
if (!is_file('app/security/acl.data')) {
    $acl = new AclList();

    // ... Define roles, resources, access, etc

    // Store serialized list into plain file
    file_put_contents(
        'app/security/acl.data',
        serialize($acl)
    );
} else {
    // Restore ACL object from serialized file
    $acl = unserialize(
        file_get_contents('app/security/acl.data')
    );
}

// Use ACL list as needed
if ($acl->isAllowed('Guests', 'Customers', 'edit')) {
    echo 'Access granted!';
} else {
    echo 'Access denied :(';
}
```

It's recommended to use the Memory adapter during development and use one of the other adapters in production.

<a name='events'></a>

## Events

`Phalcon\Acl` is able to send events to an `EventsManager` if it's present. Events are triggered using the type 'acl'. Some events when returning boolean false could stop the active operation. The following events are supported:

| Event Name        | Triggered                                               | Can stop operation? |
| ----------------- | ------------------------------------------------------- |:-------------------:|
| beforeCheckAccess | Triggered before checking if a role/resource has access |         Yes         |
| afterCheckAccess  | Triggered after checking if a role/resource has access  |         No          |

The following example demonstrates how to attach listeners to this component:

```php
<?php

use Phalcon\Acl\Adapter\Memory as AclList;
use Phalcon\Events\Event;
use Phalcon\Events\Manager as EventsManager;

// ...

// Create an event manager
$eventsManager = new EventsManager();

// Attach a listener for type 'acl'
$eventsManager->attach(
    'acl:beforeCheckAccess',
    function (Event $event, $acl) {
        echo $acl->getActiveRole();

        echo $acl->getActiveResource();

        echo $acl->getActiveAccess();
    }
);

$acl = new AclList();

// Setup the $acl
// ...

// Bind the eventsManager to the ACL component
$acl->setEventsManager($eventsManager);
```

<a name='custom-adapters'></a>

## Implementing your own adapters

The `Phalcon\Acl\AdapterInterface` interface must be implemented in order to create your own ACL adapters or extend the existing ones.