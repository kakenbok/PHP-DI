# Configure

The configuration is **optional**. PHP-DI will work with default and automatic behavior without it.

Here is an example of a configuration (you can put it in a `di.php` configuration file if you want to):

```php
Container::addConfiguration(array(

	// Used to restrict the configuration to a specific namespace
	"namespace" => "",

	// Beans and value definitions
	"entries" => array(
		"email.from" => "me@example.org",
		"db.params" => array(
			"dbname"   => "foo",
			"user"     => "root",
			"password" => "",
		),
		"dbAdapter" => function(Container $c) {
			return new DbAdapter($c["db.params"]);
		},
	),

	// Type mapping for injection using abstract types
	"aliases" => array(
		"My\Interface" => "My\Implementation",
	),

));
```


## Namespace

When you call `Container::addConfiguration`, you can specify a namespace:

```php
Container::addConfiguration(array(
	"namespace" => "Application\Users",
	"entries" => array(
		"dbAdapter" => function(Container $c) {
			return new DbAdapter("user_database");
		},
	),
));
```

The whole configuration given in the array will then be effective only for classes of this namespace.

That way, you can for example make your modules use 2 different databases without impacting your code:

```php
Container::addConfiguration(array(
	"namespace" => "Application\Products",
	"entries" => array(
		"dbAdapter" => function(Container $c) {
			return new DbAdapter("product_database");
		},
	),
));
```

By default you can skip it and your configuration will be applied globally.


## Beans

You can define a bean that can be injected using the [@Inject("myBean") annotation](doc/inject).

```php
Container::addConfiguration(array(
	"entries" => array(
		"myBean" => new MyClass(),
	),
));
```

However, a more efficient way of configuring a bean is through a closure (or callback):

```php
Container::addConfiguration(array(
	"entries" => array(
		"myBean" => function(Container $c) {
			return new MyClass($c["foo"], $c["bar"]);
		},
	),
));
```

Using this way, the object is instantiated only when (and if) it is injected.


## Values

You can define a value that can be injected using the [@Inject annotation](doc/inject).

```php
Container::addConfiguration(array(
	"entries" => array(
		"name" => "value",
	),
));
```


## Aliases

If you are trying to inject an abstract type (interface or abstract class),
this configuration allows you to define which implementation to use.

```php
Container::addConfiguration(array(
	"aliases" => array(
		"My\Interface" => "My\Implementation",
	),
));
```

You can also use it to map any types, i.e. to override a type being injected
(or even configuration values).

If you really want to know, this is simply translated by PHP-DI to:

```php
Container::addConfiguration(array(
	"entries" => array(
		"My\Interface" => function(\DI\Container $c) {
			return $c->get("My\Implementation");
		},
	),
));
```