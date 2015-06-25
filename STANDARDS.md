STANDARDS FOR THE FRONTEND

# Introduction

These are the styles agreed upon by the Front End Team for the Home Depot RenoWalk app.

# Table of content
* [General](#general)
    * [Directory structure](#directory-structure)
    * [Build](#build)
    * [Others](#others)
* [Modules](#modules)
* [Controllers](#controllers)
* [Directives](#directives)
* [Filters](#filters)
* [Services](#services)
* [Templates](#templates)
* [Routing](#routing)

# General

## Directory structure

* Create high-level divisions by functionality and lower-level divisions by component types.

```
.
├── app
│   ├── app.js
│   ├── common
│   │   ├── controllers
│   │   ├── directives
│   │   ├── filters
│   │   └── services
│   ├── home
│   │   ├── controllers
│   │   │   ├── FirstCtrl.js
│   │   │   └── SecondCtrl.js
│   │   ├── directives
│   │   │   └── directive1.js
│   │   ├── filters
│   │   │   ├── filter1.js
│   │   │   └── filter2.js
│   │   └── services
│   │       ├── service1.js
│   │       └── service2.js
│   └── about
│       ├── controllers
│       │   └── ThirdCtrl.js
│       ├── directives
│       │   ├── directive2.js
│       │   └── directive3.js
│       ├── filters
│       │   └── filter3.js
│       └── services
│           └── service3.js
├── partials
├── lib
└── test
```

* In case the directory name contains multiple words, use lisp-case syntax:

```
app
 ├── app.js
 └── my-complex-module
     ├── controllers
     ├── directives
     ├── filters
     └── services
```

* Put all the files associated with the given directive (i.e. templates, CSS/SASS files, JavaScript) in a single folder. If you choose to use this style be consistent and use it everywhere along your project.

```
app
└── directives
    ├── directive1
    │   ├── directive1.html
    │   ├── directive1.js
    │   └── directive1.sass
    └── directive2
        ├── directive2.html
        ├── directive2.js
        └── directive2.sass
```

## Build

* Automate build workflow using:
    * [Grunt](http://gruntjs.com)
    * [Bower](http://bower.io)

## Others

* Use angular objects:
    * `$timeout` instead of `setTimeout`
    * `$interval` instead of `setInterval`
    * `$window` instead of `window`
    * `$document` instead of `document`
    * `$http` instead of `$.ajax`

* Don't use global variables. Resolve all dependencies using Dependency Injection.
* When resolving dependency injections, the built-in AngularJS dependencies should be first, followed by your custom ones:

```javascript
module.factory('Service', function ($rootScope, $timeout, MyCustomDependency1, MyCustomDependency2) {
  return {
    //Something
  };
});
```

# Modules

* Modules should be named with lowerCamelCase. 
* For indicating that module `login` is submodule of module `renoApp` you can nest them by using namespacing like: `renoApp.login`.

```javascript
angular.module('app.login', []);
```

# Controllers

* Controllers should be named with UpperCamelCase (`loginController`, `userController`, etc.).
* The naming of the controller is done using the controller's functionality (for example login, user, group) and the substring `Ctrl` or `Controller` in the end.
* Do not manipulate DOM in your controllers, use directives instead.
* We need to define a syntax for defining controllers and place it below:

  ```JavaScript
  ```

* Make the controllers as lean as possible. Abstract commonly used functions into a service.
* Avoid writing business logic inside controllers. Delegate business logic to a `model`, using a service.
  For example:

  ```Javascript
  //This is a common behaviour (bad example) of using business logic inside a controller.
  angular.module('Store', [])
  .controller('OrderCtrl', function ($scope) {

    $scope.items = [];

    $scope.addToOrder = function (item) {
      $scope.items.push(item);//-->Business logic inside controller
    };

    $scope.removeFromOrder = function (item) {
      $scope.items.splice($scope.items.indexOf(item), 1);//-->Business logic inside controller
    };

    $scope.totalPrice = function () {
      return $scope.items.reduce(function (memo, item) {
        return memo + (item.qty * item.price);//-->Business logic inside controller
      }, 0);
    };
  });
  ```

  When delegating business logic into a 'model' service, controller will look like this:

  ```Javascript
  //Order is used as a 'model'
  angular.module('Store', [])
  .controller('OrderCtrl', function (Order) {

    $scope.items = Order.items;

    $scope.addToOrder = function (item) {
      Order.addToOrder(item);
    };

    $scope.removeFromOrder = function (item) {
      Order.removeFromOrder(item);
    };

    $scope.totalPrice = function () {
      return Order.total();
    };
  });
  ```

  Why business logic / app state inside controllers is bad?
  * Controllers instantiated for each view and dies when the view unloads
  * Controllers are not reusable - they are coupled with the view
  * Controllers are not meant to be injected
  

* When you need to format data encapsulate the formatting logic into a [filter](#filters) and declare it as dependency:

   ```JavaScript
   function myFormat() {
     return function () {
       // ...
     };
   }
   module.filter('myFormat', myFormat);

   function MyCtrl($scope, myFormatFilter) {
     // ...
   }

   module.controller('MyCtrl', MyCtrl);
   ```


# Directives

* Name your directives with lowerCamelCase.
* Use `scope` instead of `$scope` in your link function? This style is also used in AngularJS's source code.
* Use custom prefixes for your directives to prevent name collisions with third-party libraries.
* Do not use `ng` or `ui` prefixes since they are reserved for AngularJS and AngularJS UI usage.
* DOM manipulations must be done only through directives.
* Create an isolated scope when you develop reusable components.
* Use directives as attributes or elements instead of comments or classes, this will make your code more readable.
* Use `scope.$on('$destroy', fn)` for cleaning up. This is especially useful when you're wrapping third-party plugins as directives.
* Do not forget to use `$sce` when you should deal with untrusted content.

# Filters

* Name your filters with lowerCamelCase.
* Make your filters as light as possible. They are called often during the `$digest` loop so creating a slow filter will slow down your app.
* Do a single thing in your filters, keep them coherent. More complex manipulations can be achieved by piping existing filters.

# Services

This section includes information about the service component in AngularJS. It is not dependent of the way of definition (i.e. as provider, `.factory`, `.service`), except if explicitly mentioned.

* Use camelCase to name your services.
  * UpperCamelCase (PascalCase) for naming your services, used as constructor functions i.e.:

    ```JavaScript
    function MainCtrl($scope, User) {
      $scope.user = new User('foo', 42);
    }

    module.controller('MainCtrl', MainCtrl);

    function User(name, age) {
      this.name = name;
      this.age = age;
    }

    module.factory('User', function () {
      return User;
    });
    ```

  * lowerCamelCase for all other services.

* Encapsulate all the business logic in services. Prefer using it as your `model`. For example:
  ```Javascript
  //Order is the 'model'
  angular.module('Store')
  .factory('Order', function () {
      var add = function (item) {
        this.items.push (item);
      };

      var remove = function (item) {
        if (this.items.indexOf(item) > -1) {
          this.items.splice(this.items.indexOf(item), 1);
        }
      };

      var total = function () {
        return this.items.reduce(function (memo, item) {
          return memo + (item.qty * item.price);
        }, 0);
      };

      return {
        items: [],
        addToOrder: add,
        removeFromOrder: remove,
        totalPrice: total
      };
  });
  ```

* Services representing the domain preferably a `service` instead of a `factory`. In this way we can take advantage of the "klassical" inheritance easier:

	```JavaScript
	function Human() {
	  //body
	}
	Human.prototype.talk = function () {
	  return "I'm talking";
	};

	function Developer() {
	  //body
	}
	Developer.prototype = Object.create(Human.prototype);
	Developer.prototype.code = function () {
	  return "I'm coding";
	};

	myModule.service('Human', Human);
	myModule.service('Developer', Developer);

	```


# Templates

* Use `ng-bind` or `ng-cloak` instead of simple `{{ }}` to prevent flashing content.
* Avoid writing complex expressions in the templates.
* When you need to set the `src` of an image dynamically use `ng-src` instead of `src` with `{{ }}` template.
* When you need to set the `href` of an anchor tag dynamically use `ng-href` instead of `href` with `{{ }}` template.
* Instead of using scope variable as string and using it with `style` attribute with `{{ }}`, use the directive `ng-style` with object-like parameters and scope variables as values:

```HTML
<script>
...
$scope.divStyle = {
  width: 200,
  position: 'relative'
};
...
</script>

<div ng-style="divStyle">my beautifully styled div which will work in IE</div>;
```

# Routing

* Use UI-Router and specifiy states of the apps to handle routing.
