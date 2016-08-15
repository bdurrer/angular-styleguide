# LGT SmartBanking Style Guide

## Purpose
*Opinionated Angular style guide for the SmartBanking team

## Usage
This list should remain short, so we can skim over it in no time.
Therefore we usually skip the explanation and link to the original style rules by [john_papa](https://github.com/johnpapa/angular-styleguide) instead.

Rules which are not based on the guide by john_papa should contain an explanation and a code sample


## Custom rules
  1. [Naming](#naming)
  1. [Files](#files)
  1. [AFP Components](#afp-components)
  1. [Webpack](#webpack)

## General rules
  1. [Single Responsibility](#single-responsibility)
  1. [Modules](#modules)
  1. [Controllers](#controllers)
  1. [Data Services](#data-services)
  1. [Directives](#directives)
  1. [Resolving Promises](#resolving-promises)
  1. [Exception Handling](#exception-handling)
  1. [Angular $ Wrapper Services](#angular--wrapper-services)
  1. [Constants](#constants)

  
  
# Custom Conventions
## Naming
  **input** discuss global naming rules


## Files
  - Put directives and utilities into a module
  - The entry point of a module is always named `index.js`
  - Put directives, controllers, components, services, .... in a single file. Do not combine multiple aspects in one file.

## AFP Components
  - Define the dependency as angular.module('app', ['afp-widgets'] ) without any `require('afp-widgets')`
  - Include the `global-loader.js` in your banklet
  
  *Why?* The AFP components have an very large dependency tree and are used everywhere. We placed the commonly-used modules in `global.js` and load it using the `global-loader.js`
  Since we cannot control the webpack build (yet), simply do not reference it with `require()`.

## Webpack
### Atomic files
  - Partial files of modules should be atomic, if possible
  - Angular partials should register themself and export the angular module
  
  *Why?* Define once, use everywhere. Avoids repeating controller definitions. Makes analyzing the code easier

  ```javascript
  /* avoid */
  // controller.js
  function MyCtrl(){
  }
  
  module.exports = MyCtrl;
  
  // index.js
  angular.module('mymodule', []).controller('MyCtrl', require('./controller.js');
  ```
  
  ```javascript
  /* recommended */
  // controller.js
  function MyCtrl(){
  }
  
  // the module is defined in index.js  
  module.exports = angular.module('mymodule').controller('MyCtrl',MyCtrl);
  
  // index.js
  angular.module('mymodule',[]);
  require('./controller.js');
  ```

### UnSafe from Minification [Style [Y090](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y090)]

  We have an Angular-Injection loader in Webpack integrated. However, when you don't follow the rule about partials, it might not detect your controller/factory/service

  - Avoid using the shortcut syntax of declaring dependencies without using a minification-safe approach.
  - When in doubt, place the marker statement `"ngInject";` the first statement of your function


# General rules


## Single Responsibility

#### Rule of 1 [Style [Y001](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y001)]

  - Define 1 component per file, recommended to be less than 400 lines of code.

#### Small Functions [Style [Y002](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y002)]

  - Define small functions, no more than 75 LOC (less is better).


## Modules

#### Avoid Naming Collisions [Style [Y020](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y020)]
  - Use unique naming conventions with separators for sub-modules.
  
  For example `app` may be your root module while `app.dashboard` may be a sub-module or component.
  

#### Definitions [Style [Y021](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y021)],  [Style [Y022](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y022)] and [Style [Y023](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y023)]

  - Declare modules once (in index.js) without a variable using the setter syntax. Use the getter syntax in partials.
  - When using a module, avoid using a variable and instead use chaining with the getter syntax.

  ```javascript
  /* avoid */
  var app = angular.module('app', [ 'ngRoute' ]);
  ```
  
  ```javascript
  /* recommended */

  // to set a module in index.js
  angular.module('app', []);

  // to get a module in partial.js
  angular.module('app').controller(...);
  ```


#### Named vs Anonymous Functions [Style [Y024](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y024)]

  - Use named functions instead of passing an anonymous function in as a callback.

  ```javascript
  /* avoid */
  angular
      .module('app')
      .controller('DashboardController', function() { })
  ```

  ```javascript
  /* recommended */

  // dashboard.js
  angular
      .module('app')
      .controller('DashboardController', DashboardController);

  function DashboardController() { }
  ```

## Controllers

#### Keep Controllers Focused [Style [Y037](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y037)]
  - Define an controller for a view
  - Do not reuse the controller for other views. Move reusable logic to factories and keep the controller simple and focused on its view.

#### controllerAs *View* Syntax [Style [Y030](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y030)]

  - Use the [`controllerAs`](http://www.johnpapa.net/do-you-like-your-angular-controllers-with-or-without-sugar/) syntax over the `classic controller with $scope` syntax.

  ```html
  <!-- avoid -->
  <div ng-controller="CustomerController">
      {{ name }}
  </div>
  ```

  ```html
  <!-- recommended -->
  <div ng-controller="CustomerController as customer">
      {{ customer.name }}
  </div>
  ```  
 
#### controllerAs *router* Syntax [CUST01] and [Style [Y038](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y038)]
  - Use controllerAs on directives and ui.router states
  - Define controllers along with their routes.

  *Why?* When nesting controllers, the scope is not mixed. Results in cleaner and more readable code.
  
  ```javascript
  angular
      .module('app')
      .config(config);
      
      function config($stateProvider) {
          $stateProvider.state('contacts', {
            template: 'custom.html',
            controller: 'CustomerController',
            controllerAs: 'customCtrl'
          })
      }
  ```
  
  ```html
  <!-- custom.html -->
  <div>{{customCtrl.name}}</div>
  ``` 

  
#### controllerAs *Controller* Syntax [Style [Y031](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y031)]

  - Use the `controllerAs` syntax over the `classic controller with $scope` syntax.
  - The `controllerAs` syntax uses `this` inside controllers which gets bound to `$scope`
  - Consider using `$scope` in a controller only when needed. E.g. when using events (`$emit`,`$broadcast`,`$on`)
  
  ```javascript
  /* avoid */
  function CustomerController($scope) {
      $scope.name = {};
      $scope.sendMessage = function() { };
  }
  ```

  ```javascript
  /* recommended - but see next section */
  function CustomerController() {
      this.name = {};
      this.sendMessage = function() { };
  }
  ```

#### controllerAs with vm [Style [Y032](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y032)]
  **input** naming? suggestion: use Ctrl
  - Use a capture variable for `this` when using the `controllerAs` syntax. Choose a consistent variable name such as `vm`, which stands for ViewModel.

  *Why?*: The `this` keyword is contextual and when used within a function inside a controller may change its context. Capturing the context of `this` avoids encountering this problem.

  ```javascript
  /* recommended */
  function CustomerController() {
      var vm = this;
      vm.name = {};
      vm.sendMessage = function() { vm.name = 'x' };
  }
  ```

  Note: When creating watches in a controller using `controller as`, you can watch the `vm.*` member using the following syntax. (Create watches with caution as they add more load to the digest cycle.)

  ```javascript
  function SomeController($scope, $log) {
      var vm = this;
      vm.title = 'Some Title';

      $scope.$watch('vm.title', function(current, original) { ... });
  }
  ```

  Note: When working with larger codebases, using a more descriptive name can help ease cognitive overhead & searchability. Avoid overly verbose names that are cumbersome to type.

  ```html
  <!-- avoid -->
  <input ng-model="customerProductItemVm.title">
  ```

  ```html
  <!-- recommended -->
  <input ng-model="productVm.title">
  ```

#### Bindable Members Up Top [Style [Y033](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y033)]
  **input** discuss usefulness. Bonus: It will be very easy to migrate to ES6
  - Place bindable members at the top of the controller, alphabetized, and not spread through the controller code.

  ```javascript
  /* avoid */
  function SessionsController() {
      var vm = this;

      vm.refresh = function() {
        /* ... */
      };
      vm.search = function() {
        /* ... */
      };
      vm.sessions = [];
      vm.title = 'Sessions';
  }
  ```

  ```javascript
  /* recommended */
  function SessionsController() {
      var vm = this;

      vm.refresh = refresh;
      vm.search = search;
      vm.sessions = [];
      vm.title = 'Sessions';

      ////////////

      function refresh() {
        /* */
      }

      function search() {
        /* */
      }
  }
  ```


#### Function Declarations to Hide Implementation Details [Style [Y034](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y034)]

  - Use function declarations to hide implementation details. 
  - Keep your bindable members up top. 
  - When you need to bind a function in a controller, point it to a function declaration that appears later in the file. This is tied directly to the section [Bindable Members Up Top](#bindable-members-up-top).

    *Why?*: Placing bindable members at the top makes it easy to read and helps you instantly identify which members of the controller can be bound and used in the View. (Same as above.)

    *Why?*: Placing the implementation details of a function later in the file moves that complexity out of view so you can see the important stuff up top.

    *Why?*: Function declarations are hoisted so there are no concerns over using a function before it is defined (as there would be with function expressions).

    *Why?*: You never have to worry with function declarations that moving `var a` before `var b` will break your code because `a` depends on `b`.

    *Why?*: Order is critical with function expressions

  ```javascript
  /**
   * avoid
   * Using function expressions.
   */
  function AvengersController(avengersService, logger) {
      var vm = this;
      vm.avengers = [];
      vm.title = 'Avengers';

      var activate = function() {
          return getAvengers().then(function() {
              logger.info('Activated Avengers View');
          });
      }

      var getAvengers = function() {
          return avengersService.getAvengers().then(function(data) {
              vm.avengers = data;
              return vm.avengers;
          });
      }

      vm.getAvengers = getAvengers;

      activate();
  }
  ```

  ```javascript
  /*
   * recommend
   * Using function declarations
   * and bindable members up top.
   */
  function AvengersController(avengersService, logger) {
      var vm = this;
      vm.avengers = [];
      vm.getAvengers = getAvengers;
      vm.title = 'Avengers';

      activate();

      function activate() {
          return getAvengers().then(function() {
              logger.info('Activated Avengers View');
          });
      }

      function getAvengers() {
          return avengersService.getAvengers().then(function(data) {
              vm.avengers = data;
              return vm.avengers;
          });
      }
  }
  ```

#### Controller Activation Promises [Style [Y080](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y080)]

  - Resolve start-up logic for a controller in an `activate` function.

  ```javascript
  /* recommended */
  function AvengersController(dataservice) {
      var vm = this;
      vm.title = 'Avengers';

      activate();

      ////////////

      function activate() {
      }
  }
  ```  

#### Defer Controller Logic to Services [Style [Y035](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y035)]

  - Defer logic in a controller by delegating to services and factories.

  ```javascript
  /* recommended */
  function OrderController(creditService) {
      var vm = this;
      vm.checkCredit = checkCredit;
      vm.isCreditOk;
      vm.total = 0;

      function checkCredit() {
         return creditService.isOrderTotalOk(vm.total)
            .then(function(isOk) { vm.isCreditOk = isOk; })
            .catch(showError);
      };
  }
  ```

## Data Services


#### Separate Data Calls [Style [Y060](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y060)]

  - Refactor logic for making data operations and interacting with data to a factory. Make data services responsible for XHR calls, local storage, stashing in memory, or any other data operations.
  - Place caching functionality into the service instead of repeating it in every controller


#### Return a Promise from Data Calls [Style [Y061](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y061)]

  - When calling a data service that returns a promise such as `$http`, return a promise in your calling function too.



## Directives
#### Provide a Unique Directive Prefix [Style [Y073](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y073)]
  **input** define sb prefix!
  
  - Provide a short, unique and descriptive directive prefix such as `acmeSalesCustomerInfo` which would be declared in HTML as `acme-sales-customer-info`.

#### Restrict to Elements and Attributes [Style [Y074](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y074)]
  - Never use `C` (CSS class)
  - When creating a directive that makes sense as a stand-alone element, allow restrict `E` (custom element) and optionally restrict `A` (custom attribute). Generally, if it could be its own control, `E` is appropriate. General guideline is allow `EA` but lean towards implementing as an element when it's stand-alone and as an attribute when it enhances its existing DOM element.


#### Directives and ControllerAs [Style [Y075](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y075)]
  - Use `controller as` syntax with a directive to be consistent with using `controller as` with view and controller pairings.
  - Bonus: It's easy to see when you accidentally access an variable which does not belong to the directive's scope
  - Using an controller splits the bootstrapping and the interactive features (event handlers) in two parts (controller and linkFunc). You might even be able to skip the linkFunc entirely.

```javascript
  angular
      .module('app')
      .directive('myDirective', myDirective);

  function myDirective() {
      var directive = {
          restrict: 'EA',
          templateUrl: 'directive.html',
          scope: {
              max: '='
          },
          link: linkFunc,
          controller: ExampleController,
          // note: This would be 'ExampleController' (the exported controller name, as string)
          // if referring to a defined controller in its separate file.
          controllerAs: 'vm',
          bindToController: true // because the scope is isolated
      };

      return directive;

      function linkFunc(scope, el, attr, ctrl) {
          console.log('LINK: scope.vm.min = %s', scope.vm.min);
          console.log('LINK: scope.vm.max = %s', scope.vm.max);
      }
  }

  function ExampleController($scope) {
      "ngInject";
      // Injecting $scope just for comparison
      var vm = this;
      vm.min = 3;
  }
  ```

  ```html
  <!-- directive.html -->
  <div>max={{vm.max}}</div>  <div>min={{vm.min}}</div>
  ```

#### [Style [Y076](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y076)]

  - Use `bindToController = true` when using `controller as` syntax with a directive when you want to bind the outer scope to the directive's controller's scope.


## Resolving Promises

#### Route Resolve Promises [Style [Y081](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y081)]

  - When a controller depends on a promise to be resolved before the controller is activated, resolve those dependencies in the `$routeProvider` before the controller logic is executed. If you need to conditionally cancel a route before the controller is activated, use a route resolver.
  - Use a route resolve when you want to decide to cancel the route before ever transitioning to the View.
  - If the code of the route resolve is slow and the view can already show some of the data, an controller activation function might be more suitable


#### Handling Exceptions with Promises [Style [Y082](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y082)]

  - The `catch` block of a promise must return a rejected promise to maintain the exception in the promise chain.
  - Always handle exceptions in services/factories.
  - Make use of global exception handler modules, if possible (or start one!)
  - When in doubt, define the dependencies manually. See [Style [Y092](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y092)], [Style [Y082](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y082)]




## Exception Handling

#### Decorators [Style [Y110](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y110)]
 **input** we should do this and make a rule to include some SB-core module. It's the same technique as the http-logger interceptor.
 **input** discuss exception and error handling in general
 
- Use a [decorator](https://docs.angularjs.org/api/auto/service/$provide#decorator), at config time using the [`$provide`](https://docs.angularjs.org/api/auto/service/$provide) service, on the [`$exceptionHandler`](https://docs.angularjs.org/api/ng/service/$exceptionHandler) service to perform custom actions when exceptions occur.


#### Exception Catchers [Style [Y111](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y111)]
**input** discuss exception and error handling in general
  - Create a factory that exposes an interface to catch and gracefully handle exceptions.


#### Route Errors [Style [Y112](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y112)]
**input** discuss exception and error handling in general
  - Handle and log all routing errors using [`$routeChangeError`](https://docs.angularjs.org/api/ngRoute/service/$route#$routeChangeError).

  


## Angular $ Wrapper Services

####  $document and $window [Style [Y180](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y180)]

  - Use [`$document`](https://docs.angularjs.org/api/ng/service/$document) and [`$window`](https://docs.angularjs.org/api/ng/service/$window) instead of `document` and `window`.

#### $timeout and $interval [Style [Y181](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y181)]

  - Use [`$timeout`](https://docs.angularjs.org/api/ng/service/$timeout) and [`$interval`](https://docs.angularjs.org/api/ng/service/$interval) instead of `setTimeout` and `setInterval` .

#### $log [CUSTOM]
 - Use `$log` instead of `console.log`

*WHY* Older browsers (eg IE8) do not support console.log (or not always) and it can break your code
*WHY* the $log interface is cleaner (you can even differentiate between warn,debug,error!)


## Constants

#### Constant Values [Style [Y241](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y241)]
  - Use constants for values that do not change and do not come from another service. When constants are used only for a module that may be reused in multiple applications, place constants in a file per module named after the module. Until this is required, keep constants in the main module in a `constants.js` file.
