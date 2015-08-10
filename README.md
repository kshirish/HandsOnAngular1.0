# HandsOnAngular1.0

## Scope

- `$scope`: Scope is an object that refers to the application model. It is an execution context for expressions. Scopes provide context against which expressions are evaluated. For example `{{username}}` expression is meaningless, unless it is evaluated against a specific scope which defines the `username` property.
- `Child scope`: It (prototypically) inherits properties from its parent scope.
- `Isolate scope`: It does not.
- `$watch`: Scopes provide APIs to observe model mutations.
- `$apply`: Scopes provide APIs to propagate any model changes through the system into the view from outside of the "Angular realm" (`controllers`, `services`, Angular event handlers).
- During the template linking phase the directives set up `$watch` expressions on the scope. The `$watch` allows the directives to be notified of property changes, which allows the directive to render the updated value to the DOM.
- At the end of `$apply`, Angular performs a `$digest` cycle on the `root scope`, which then propagates throughout all child scopes. During the `$digest` cycle, all `$watch`ed expressions or functions are checked for model mutation and if a mutation is detected, the `$watch` listener is called.
- When child scopes are no longer needed, it is the responsibility of the child scope creator to destroy them via `scope.$destroy()` API. This will stop propagation of `$digest` calls into the child scope and allow for memory used by the child scope models to be reclaimed by the garbage collector.

## Directive

- **Observing directives**: `ng-model` or `{{expression}}` registers listeners using the `$watch()` method. This type of directive needs to be notified whenever the expression changes so that it can update the view.
- **Listener directives**: `ng-click` registers a listener with the DOM. When the DOM listener fires, the directive executes the associated expression and updates the view using the `$apply()` method.

## Angular Event Loop

- Enter the Angular execution context by calling `scope.$apply(stimulusFn)`, where `stimulusFn` is the work you wish to do in the Angular execution context.
- Angular executes the `stimulusFn()`, which typically modifies application state
- Angular enters the `$digest` loop. The loop is made up of two smaller loops which process `$evalAsync` queue and the `$watch` list. The `$digest` loop keeps iterating until the model stabilizes, which means that the `$evalAsync` queue is empty and the `$watch` list does not detect any changes.
- The `$evalAsync` queue is used to schedule work which needs to occur outside of current stack frame, but before the browser's view render i.e. `setTimeout(0)`.
- The `$watch` list is a set of expressions which may have changed since last iteration. If a change is detected then the `$watch` function is called which typically updates the DOM with the new value.
- Once the Angular `$digest` loop finishes the execution leaves the Angular and JavaScript context. This is followed by the browser re-rendering the DOM to reflect any changes.

Here is the explanation of how the `Hello world example` achieves the data-binding effect when the user enters text into the text field.

- **During the compilation phase**: the `ng-model` and input directive set up a keydown listener on the `<input>` control.
the interpolation sets up a `$watch` to be notified of name changes.
- **During the runtime phase**: Pressing an 'X' key causes the browser to emit a keydown event on the input control.
- The input directive captures the change to the input's value and calls `$apply("name = 'X';")` to update the application model inside the Angular execution context.
- Angular applies the name = 'X'; to the model.
- The `$digest` loop begins
- The `$watch` list detects a change on the name property and notifies the interpolation, which in turn updates the DOM.
- Angular exits the execution context, which in turn exits the keydown event and with it the JavaScript execution context.
- The browser re-renders the view with update text.


## References

- [Angular Docs](https://docs.angularjs.org/)
