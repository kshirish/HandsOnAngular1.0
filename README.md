# HandsOnAngular1.0

## Scope

- `$scope`: Scope is an object that refers to the application model. It is an execution context for expressions. Scopes provide context against which expressions are evaluated. For example `{{username}}` expression is meaningless, unless it is evaluated against a specific scope which defines the `username` property.
- `Child scope`: It (prototypically) inherits properties from its parent scope.
- `Isolate scope`: It does not.
- During the template linking phase the directives set up `$watch` expressions on the scope. The `$watch` allows the directives to be notified of property changes, which allows the directive to render the updated value to the DOM.
- When child scopes are no longer needed, it is the responsibility of the child scope creator to destroy them via `scope.$destroy()` API. This will stop propagation of `$digest` calls into the child scope and allow for memory used by the child scope models to be reclaimed by the garbage collector.

## Directive

- **Observing directives**: `ng-model` or `{{expression}}` registers listeners using the `$watch()` method. This type of directive needs to be notified whenever the expression changes so that it can update the view.
- **Listener directives**: `ng-click` registers a listener with the DOM. When the DOM listener fires, the directive executes the associated expression and updates the view using the `$apply()` method.

## Angular Event Loop

- Enter the Angular execution context by calling `scope.$apply(fn)`, where `fn` is the function you wish to execute in the Angular context.
- Angular executes the `fn()`, which typically modifies application state
- Angular enters the `$digest` loop. The loop is made up of two smaller loops which process `$evalAsync` queue and the `$watch` list. The `$digest` loop keeps iterating until the model stabilizes, which means that the `$evalAsync` queue is empty and the `$watch` list does not detect any changes.
- `$evalAsync` queue is used to schedule work which needs to occur outside of current stack frame, but before the browser's view render i.e. `setTimeout(0)`.
- `$watch` list is a set of expressions which may have changed since last iteration. If a change is detected then the `$watch` function is called which typically updates the DOM with the new value.
- Once `$digest` cycle ends, the browser re-renders the DOM to reflect any changes.

Here is the explanation of how the `Hello world example` achieves the data-binding effect when the user enters text into the text field.

- **During the compilation phase**: the `ng-model` and input directive set up a keydown listener on the `<input>`.
the interpolation sets up a `$watch` to be notified of name changes.
- **During the runtime phase**: Pressing an 'X' key causes the browser to emit a keydown event on the input control.
- The input directive captures the change to the input's value and calls `$apply("name = 'X';")` to update the application model inside the Angular execution context.
- Angular applies the name = 'X'; to the model.
- The `$digest` cycle begins.
- The `$watch` list detects a change on the name property and notifies the interpolation, which in turn updates the DOM.
- When the `$digest` cycle is over, the browser re-renders the view with update text.

## $scope.$apply()

- Each time you change some watched variable attached to the $scope object directly, Angular will know that the change has happened. This is because Angular already knew to monitor those changes. So if it happens in code managed by the framework, the digest cycle will carry on. However, sometimes you want to change some value outside of the Angular world and see the changes propagate normally. Consider this - you have a `$scope.myVar` value which will be modified within a jQuery's `$.ajax()` handler. This will happen at some point in future. Angular can't wait for this to happen, since it hasn't been instructed to wait on jQuery. To tackle this, `$apply` has been introduced. It lets you to start the digestion cycle explicitly. However, you should only use this to migrate some data to Angular (integration with other frameworks), but never use this method combined with regular Angular code, as Angular will throw an error then.
- There are two ways of declaring a $scope variable as being watched.
  - By using it in your template via expression: `<span>{{myVar}}</span>`.
  - By adding it manually via $watch service

## $evalAsync vs $timeout

- If you truly want to execute code at a later point in time, use `$timeout()`. However, if your only goal is tell AngularJS about a data change without throwing a `$digest already in progress` error, then go for `$scope.$evalAsync()`.

## Angular events

- Unsubscribing from angular events:
```js
var removeListener = $scope.$on("someEvent", function listener() {});

// while subscribing to `someEvent`, `$on` return a function which can be used to degister `listener`
removeListener();

```
- `$destroy()`
```js
  $scope.$on('$destroy', function() {
    /** When this scope will be destroyed, you are given a chance to do a final cleanup.
    * It also implies that the current scope is now eligible for garbage collection.
    */
  });
```
- Cancelling events
```js

$scope.$on('myCustomEvent', function (event, data) {
  event.stopPropagation();  // prevents it from bubbling further  
});

```

## Best practices

- Don’t clutter up the digest cycle by adding more variables( global ) to `$rootScope`, rather use Angular’s explicit event features like `emit` and `broadcast`.
- Instead of using multiple small watch functions i.e.
```js
$scope.$watch("mohan.firstName", function() {...});
$scope.$watch("mohan.lastName", function() {...});
$scope.$watch("mohan.age", function() {...});
```

Write a hash function i.e.

```js
Person.prototype.hash = function() {
    return this.firstName + this.lastName + this.Age;
}

$scope.$watch("mohan.hash", function() {
  /** firstname, lastname or age changed **/
});
```
- Prefer `$apply(fn)` instead of  `$apply()`, because the function call is wrapped inside a `try...catch` block, and any exceptions that occur will be passed to the `$exceptionHandler` service.

## Styleguide
 - **Bad:**
 ```js
var app = angular.module('app', []);
app.controller();
app.factory();
 ```
 
**Good:**
```js
angular
  .module('app', [])
  .controller()
  .factory();
```

 - **Bad:**
 ```js
var app = angular.module('app', []);
app.controller('MyCtrl', function ($scope) {
  
});
 ```
 
**Good:**
```js
app = angular.module('app', []);
  .controller('MyController', ['$scope', function MyController($scope) {
  
  }]); 
```

 - **Bad:**
 ```js
 function MainCtrl () {
  this.doSomething = function () {

  };
}
angular
  .module('app')
  .controller('MainCtrl', MainCtrl);
 ```
 
**Good:**
```js
function MainCtrl (SomeService) {
  this.doSomething = SomeService.doSomething;
}
angular
  .module('app')
  .controller('MainCtrl', MainCtrl);
```

 - **Bad:**
 ```js
 function AnotherService () {

  var someValue = '';

  var someMethod = function () {

  };
  
  return {
    someValue: someValue,
    someMethod: someMethod
  };

}
angular
  .module('app')
  .factory('AnotherService', AnotherService);
 ```
 
**Good:**
```js
 function AnotherService () {

  var AnotherService = {};
  
  AnotherService.someValue = '';

  AnotherService.someMethod = function () {

  };
  
  return AnotherService;
}
angular
  .module('app')
  .factory('AnotherService', AnotherService);
```

 
## References

- [Angular Docs](https://docs.angularjs.org/)
- [Understanding apply and digest](http://www.sitepoint.com/understanding-angulars-apply-digest/)
- [Styleguide](http://toddmotto.com/opinionated-angular-js-styleguide-for-teams/)
