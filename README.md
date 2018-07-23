> **Note: Codebase taken from [ngReact](https://github.com/ngReact/ngReact)**

# ng-hyperapp

The [Hyperapp](https://github.com/hyperapp/hyperapp) library can be used as a view component in web applications. ng-hyperapp is an Angular module that allows hyperapp Components to be used in [AngularJS](https://angularjs.org/) applications.

Motivation for this could be any of the following:

- You need greater performance than Angular can offer (two way data binding, Object.observe, too many scope watchers on the page) and Hyperapp is typically more performant due to the Virtual DOM and other optimizations it can make

- Hyperapp offers an easier way to think about the state of your UI; instead of data flowing both ways between controller and view as in two way data binding, Hyperapp holds firm on the functional programming front when managing your state, but takes a pragmatic approach to allowing for side effects, asynchronous actions, and DOM manipulations.

- You're already deep into an Angular application and can't move away, but would like to experiment with Hyperapp

# Installation

```bash
npm install ng-hyperapp (still to be published)
```

# Usage

Then, include the 'hyperapp' Angular module as a dependency for your new app

```html
<script>
    angular.module('app', ['hyperapp']);
</script>
```

and you're good to go.

# Features

Specifically, ng-hyperapp is composed of:

- `hyperapp-component`, an Angular directive that delegates off to a hyperapp Component
- `hyperappDirective`, a service for converting hyperapp components into the `hyperapp-component` Angular directive

**ng-hyperapp** can be used in existing angular applications, to replace entire or partial views with hyperapp components.

## The hyperapp-component directive

The `hyperapp-component` directive is a generic wrapper for embedding your hyperapp components.

With an Angular app and controller declaration like this:

```javascript
angular.module('app', ['hyperapp'])
  .controller('helloController', function($scope) {
    $scope.person = { fname: 'Clark', lname: 'Kent' };
  });
```

And a hyperapp Component like this

```javascript
var HelloComponent = function (props) {
 return <span>Hello {props.fname} {props.lname}</span>; 
};
app.value('HelloComponent', HelloComponent);
```

The component can be used in an Angular view using the hyperapp-component directive like so:

```html
<body ng-app="app">
  <div ng-controller="helloController">
    <hyperapp-component name="HelloComponent" props="person" watch-depth="reference"/>
  </div>
</body>
```

Here:

- `name` attribute checks for an Angular injectable of that name and falls back to a globally exposed variable of the same name
- `props` attribute indicates what scope properties should be exposed to the hyperapp component
- `watch-depth` attribute indicates what watch strategy to use to detect changes on scope properties.  The possible values for hyperapp-component are `reference`, `collection` and `value` (default)

## The hyperappDirective service

The hyperappDirective factory, in contrast to the hyperappComponent directive, is meant to create specific directives corresponding to hyperapp components. In the background, this actually creates and sets up directives specifically bound to the specified hyperapp component.

If, for example, you wanted to use the same hyperapp component in multiple places, you'd have to specify `<hyperapp-component name="yourComponent" props="props"></hyperapp-component>` repeatedly, but if you used hyperappDirective factory, you could create a `<your-component></your-component>` directive and simply use that everywhere.

The service takes the hyperapp component as the argument.

```javascript
app.directive('helloComponent', function(hyperappDirective) {
  return hyperappDirective(HelloComponent);
});
```

Alternatively you can provide the name of the component

```javascript
app.directive('helloComponent', function(hyperappDirective) {
  return hyperappDirective('HelloComponent');
});
```

This creates a directive that can be used like this:

```html
<body ng-app="app">
  <div ng-controller="helloController">
    <hello-component fname="person.fname" lname="person.lname" watch-depth="reference"></hello-component>
  </div>
</body>
```

The `hyperappDirective` service will read the hyperapp component `propTypes` and watch attributes with these names. If your hyperapp component doesn't have `propTypes` defined you can pass in an array of attribute names to watch. If you don't pass any array of attribute names, fall back to use directive attributes as a last resort. By default, attributes will be watched by value however you can also choose to watch by reference or collection by supplying the watch-depth attribute.  Possible values are `reference`, `collection` and `value` (default).

```javascript
app.directive('hello', function(hyperappDirective) {
  return hyperappDirective(HelloComponent, ['fname', 'lname']);
});
```

You may also customize the watch depth per prop/attribute by wrapping the name and an options object in an array inside the props array:

```javascript
app.directive('hello', function(hyperappDirective) {
  return hyperappDirective(HelloComponent, [
    'person', // takes on the watch-depth of the entire directive
    ['place', {watchDepth: 'reference'}],
    ['things', {watchDepth: 'collection'}],
    ['ideas', {watchDepth: 'value'}]
  ]);
});
```

By default, ng-hyperapp will wrap any functions you pass as in `scope.$apply`. You may want to override this behavior, for instance, if you are passing a hyperapp component as a prop. You can achieve this by explicitly adding a `wrapApply: false` in the prop config:

```javascript
app.directive('hello', function(hyperappDirective) {
  return hyperappDirective(HelloComponent, [
    'person',
    ['place', {watchDepth: 'reference'}],
    ['func', {watchDepth: 'reference', wrapApply: false}]
  ]);
});
```


If you want to change the configuration of the directive created the `hyperappDirective` service, e.g. change `restrict: 'E'` to `restrict: 'C'`, you can do so by passing in an object literal with the desired configuration.

```javascript
app.directive('hello', function(hyperappDirective) {
  return hyperappDirective(HelloComponent, undefined, {restrict: 'C'});
});
```

### Minification
A lot of automatic annotation libraries including ng-annotate skip implicit annotations of directives. Because of that you might get the following error when using directive in minified code:
```
Unknown provider: eProvider <- e <- helloDirective
```
To fix it add explicit annotation of dependency
```javascript
var helloDirective = function(hyperappDirective) {
  return hyperappDirective('HelloComponent');
};
helloDirective.$inject = ['hyperappDirective'];
app.directive('hello', helloDirective);
```


## Reusing Angular Injectables

In an existing Angular application, you'll often have existing services or filters that you wish to access from your hyperapp component. These can be retrieved using Angular's dependency injection. The hyperapp component will still be render-able as aforementioned, using the hyperapp-component directive.

It's also possible to pass Angular injectables and other variables as fourth parameter straight to the hyperappDirective, which will then attach them to the props

```javascript
app.directive('helloComponent', function(hyperappDirective, $ngRedux) {
  return hyperappDirective(HelloComponent, undefined, {}, {store: $ngRedux});
});
```
