#Components

An Angular 2 application defines a set of components, for every UI element, screen, and route. Every application will always have a root component that contains all other components.

###How does a component look like in real life?
```js
// Let's take an example of MyComponent
@Component({
  selector: 'my-component', // that's how you know me
  properties: {},   // properties and 
  events: [],   // events attached to the component
  lifecycle: [onChange],
  appInjector: []   // DI here
})
@View({
  directives: [],   // directives that it will be using
  templateUrl: 'url-for-your-template.html' 
})
class MyComponent {
  component: MyComponent;   
  onChange(changes) {
    // triggered when any of the properties change
  }
  // constructor and a lot more        
}
```
** *Do not rely on code yet, it is changing frequently.*

- Property and event bindings are the public API of a component.
- Angular 2 embraces web platform standards, so it puts the view of the component in the Shadow DOM of the element where it has been created.

- A component interacts with its host DOM element in the following ways: 
    - Listens to its events. [*hostListener*]
    - Updates its properties. [*hostProperties*]
    - Invokes methods on it.
