# LGT SmartBanking Style Guide

## Purpose
*Opinionated Angular style guide for the SmartBanking team

## Usage
This list should remain short, so we can skim over it in no time.
Therefore we usually skip the explanation and link to the original style rules by [john_papa](https://github.com/johnpapa/angular-styleguide) instead.

Rules which are not based on the guide by john_papa should contain an explanation and a code sample


## Table of Contents
  1. [Custom Conventions](#custom-conventions)
  1. [Single Responsibility](#single-responsibility)
  1. [IIFE](#iife)
  1. [Modules](#modules)
  1. [Controllers](#controllers)
  1. [Services](#services)
  1. [Factories](#factories)
  1. [Data Services](#data-services)
  1. [Directives](#directives)
  1. [Resolving Promises](#resolving-promises)
  1. [Manual Annotating for Dependency Injection](#manual-annotating-for-dependency-injection)
  1. [Minification and Annotation](#minification-and-annotation)
  1. [Exception Handling](#exception-handling)
  1. [Naming](#naming)
  1. [Application Structure LIFT Principle](#application-structure-lift-principle)
  1. [Application Structure](#application-structure)
  1. [Modularity](#modularity)
  1. [Startup Logic](#startup-logic)
  1. [Angular $ Wrapper Services](#angular--wrapper-services)
  1. [Testing](#testing)
  1. [Animations](#animations)
  1. [Comments](#comments)
  1. [JSHint](#js-hint)
  1. [JSCS](#jscs)
  1. [Constants](#constants)
  1. [File Templates and Snippets](#file-templates-and-snippets)
  1. [Yeoman Generator](#yeoman-generator)
  1. [Routing](#routing)
  1. [Task Automation](#task-automation)
  1. [Filters](#filters)
  1. [Angular Docs](#angular-docs)
  
  
## Custom Conventions

### Files
  - Put directives and utilities into a module
  - The entry point of a module is always named index.js
  - Put directives, controllers, components, services, .... in a single file

### AFP Components
  - Define the dependency as angular.module('app', ['afp-widgets'] ) without any `require('afp-widgets')`
  - Include the `global-loader.js` in your banklet
  
  *Why?* The AFP components have an very large dependency tree and are used everywhere. We placed the commonly-used modules in `global.js` and load it using the `global-loader.js`
  Since we cannot control the webpack build (yet), simply do not reference it with `require()`.

### Webpack
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

## Single Responsibility

###### Rule of 1 [Style [Y001](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y001)]

  - Define 1 component per file, recommended to be less than 400 lines of code.

###### Small Functions [Style [Y002](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y002)]

  - Define small functions, no more than 75 LOC (less is better).


## Modules

###### Avoid Naming Collisions [Style [Y020](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y020)]
  - Use unique naming conventions with separators for sub-modules.
  
  For example `app` may be your root module while `app.dashboard` may be a sub-module or component.
  

###### Definitions [Style [Y021](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y021)],  [Style [Y022](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y022)] and [Style [Y023](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y023)]

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


###### Named vs Anonymous Functions [Style [Y024](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y024)]

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

###### Keep Controllers Focused [Style [Y037](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y037)]
  - Define an controller for a view
  - Do not reuse the controller for other views. Move reusable logic to factories and keep the controller simple and focused on its view.

###### controllerAs *View* Syntax [Style [Y030](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y030)]

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
 
###### controllerAs *router* Syntax [CUST01] and [Style [Y038](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y038)]
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

  
###### controllerAs *Controller* Syntax [Style [Y031](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y031)]

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

###### controllerAs with vm [Style [Y032](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y032)]
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

  Note: You can avoid any [jshint](http://jshint.com/) warnings by placing the comment above the line of code. However it is not needed when the function is named using UpperCasing, as this convention means it is a constructor function, which is what a controller is in Angular.

  ```javascript
  /* jshint validthis: true */
  var vm = this;
  ```

  Note: When creating watches in a controller using `controller as`, you can watch the `vm.*` member using the following syntax. (Create watches with caution as they add more load to the digest cycle.)

  ```html
  <input ng-model="vm.title"/>
  ```

  ```javascript
  function SomeController($scope, $log) {
      var vm = this;
      vm.title = 'Some Title';

      $scope.$watch('vm.title', function(current, original) {
          $log.info('vm.title was %s', original);
          $log.info('vm.title is now %s', current);
      });
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

###### Bindable Members Up Top [Style [Y033](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y033)]
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


###### Function Declarations to Hide Implementation Details [Style [Y034](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y034)]

  - Use function declarations to hide implementation details. Keep your bindable members up top. When you need to bind a function in a controller, point it to a function declaration that appears later in the file. This is tied directly to the section Bindable Members Up Top. For more details see [this post](http://www.johnpapa.net/angular-function-declarations-function-expressions-and-readable-code/).

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

  Notice that the important stuff is scattered in the preceding example. In the example below, notice that the important stuff is up top. For example, the members bound to the controller such as `vm.avengers` and `vm.title`. The implementation details are down below. This is just easier to read.

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

###### Defer Controller Logic to Services [Style [Y035](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y035)]

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

  
  
  
# Semi-Important stuff


## Services

### Singletons [Style [Y040](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y040)]
  **input** discuss

  - Services are instantiated with the `new` keyword, use `this` for public methods and variables. Since these are so similar to factories, use a factory instead for consistency.

    Note: [All Angular services are singletons](https://docs.angularjs.org/guide/services). This means that there is only one instance of a given service per injector.

  ```javascript
  // service
  angular
      .module('app')
      .service('logger', logger);

  function logger() {
    this.logError = function(msg) {
      /* */
    };
  }
  ```

  ```javascript
  // factory
  angular
      .module('app')
      .factory('logger', logger);

  function logger() {
      return {
          logError: function(msg) {
            /* */
          }
     };
  }
  ```


## Data Services

### Separate Data Calls
###### [Style [Y060](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y060)]

  - Refactor logic for making data operations and interacting with data to a factory. Make data services responsible for XHR calls, local storage, stashing in memory, or any other data operations.

    *Why?*: The controller's responsibility is for the presentation and gathering of information for the view. It should not care how it gets the data, just that it knows who to ask for it. Separating the data services moves the logic on how to get it to the data service, and lets the controller be simpler and more focused on the view.

    *Why?*: This makes it easier to test (mock or real) the data calls when testing a controller that uses a data service.

    *Why?*: Data service implementation may have very specific code to handle the data repository. This may include headers, how to talk to the data, or other services such as `$http`. Separating the logic into a data service encapsulates this logic in a single place hiding the implementation from the outside consumers (perhaps a controller), also making it easier to change the implementation.

  ```javascript
  /* recommended */

  // dataservice factory
  angular
      .module('app.core')
      .factory('dataservice', dataservice);

  dataservice.$inject = ['$http', 'logger'];

  function dataservice($http, logger) {
      return {
          getAvengers: getAvengers
      };

      function getAvengers() {
          return $http.get('/api/maa')
              .then(getAvengersComplete)
              .catch(getAvengersFailed);

          function getAvengersComplete(response) {
              return response.data.results;
          }

          function getAvengersFailed(error) {
              logger.error('XHR Failed for getAvengers.' + error.data);
          }
      }
  }
  ```

    Note: The data service is called from consumers, such as a controller, hiding the implementation from the consumers, as shown below.

  ```javascript
  /* recommended */

  // controller calling the dataservice factory
  angular
      .module('app.avengers')
      .controller('AvengersController', AvengersController);

  AvengersController.$inject = ['dataservice', 'logger'];

  function AvengersController(dataservice, logger) {
      var vm = this;
      vm.avengers = [];

      activate();

      function activate() {
          return getAvengers().then(function() {
              logger.info('Activated Avengers View');
          });
      }

      function getAvengers() {
          return dataservice.getAvengers()
              .then(function(data) {
                  vm.avengers = data;
                  return vm.avengers;
              });
      }
  }
  ```

### Return a Promise from Data Calls
###### [Style [Y061](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y061)]

  - When calling a data service that returns a promise such as `$http`, return a promise in your calling function too.

    *Why?*: You can chain the promises together and take further action after the data call completes and resolves or rejects the promise.

  ```javascript
  /* recommended */

  activate();

  function activate() {
      /**
       * Step 1
       * Ask the getAvengers function for the
       * avenger data and wait for the promise
       */
      return getAvengers().then(function() {
          /**
           * Step 4
           * Perform an action on resolve of final promise
           */
          logger.info('Activated Avengers View');
      });
  }

  function getAvengers() {
        /**
         * Step 2
         * Ask the data service for the data and wait
         * for the promise
         */
        return dataservice.getAvengers()
            .then(function(data) {
                /**
                 * Step 3
                 * set the data and resolve the promise
                 */
                vm.avengers = data;
                return vm.avengers;
        });
  }
  ```


## Directives
###### Provide a Unique Directive Prefix [Style [Y073](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y073)]
  **input** define sb prefix!
  
  - Provide a short, unique and descriptive directive prefix such as `acmeSalesCustomerInfo` which would be declared in HTML as `acme-sales-customer-info`.

###### Restrict to Elements and Attributes [Style [Y074](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y074)]
  - Never use `C` (CSS class)
  - When creating a directive that makes sense as a stand-alone element, allow restrict `E` (custom element) and optionally restrict `A` (custom attribute). Generally, if it could be its own control, `E` is appropriate. General guideline is allow `EA` but lean towards implementing as an element when it's stand-alone and as an attribute when it enhances its existing DOM element.


###### Directives and ControllerAs [Style [Y075](//github.com/johnpapa/angular-styleguide/tree/master/a1#style-y075)]
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
