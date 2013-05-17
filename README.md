# Basic Angular tutorial app

![General overview](/images/general_overview.png)

Angular is very very modular if you take advantage of it.
Great for **SPA** apps

### Usage

Just clone the thing and run index.html in your browser :)

## Directives (ng-) and {{ Data Bindings }}

They teach HTML new tricks

    <html ng-app>
    <input type='text' ng-model='search_query' />
    {{ search_query }}

* **ng-app** - directive defining a default angular module
* **ng-model** - another directive
* **'search_query'** - the name of the property
* **{{ search_query }}** - data binding expression

**You can't and shouldn't put conditional logic in your views!**

Another neat directive is **ng-repeat**:

    <body ng-init="people=[{name:'John', city:'NY'},
                           {name:'Mark', city:'ZG'},
                           {name:'Bob', city:'RI'}]">

    <ul>
      <li ng-repeat="person in people">{{ name }}</li>
    <ul>

**It is a good practice to make directives like so: data-ng-model=''** because of how data attributes work in html5.
You can find a whole bunch of directives at the [official docs](http://docs.angularjs.org/api/)

## Filters

    <li ng-repeat="person in people | filter:name | orderBy:'city'">{{ person }}</li>

Upper line filters people based on what you type in the input.

**You can write your own custom filters and directives.**

## Views, Controllers and Scope

VIEW <--- $scope ---> CONTROLLER

**scope** is the glue between view and controller
Controller doesn't know anything about the view; it 'communicates' with view via $scope.

    <div class='container' ng-controller='SimpleController'>
      Name: <input type='text' ng-model='query' />
      <ul>
        <li ng-repeat="cust in customers | filter:query | orderBy:'city'">{{ cust.name }} - {{ cust.city }}</li>
      <ul>
    </div>

    <script>
      function SimpleController($scope) {
        $scope.customers = [
          {name:'John', city:'NY'},
          {name:'Mark', city:'ZG'},
          {name:'Bob', city:'RI'}
        ];
      }
    </script>

**customers** - a scope property
Note that we can also acces the 'query' thingy inside SimpleController function (because of the $scope).

## Modules

![Module](/images/module.png)

**Module** is like an object container. This is how we create modules:

    var demoApp = angular.module('demoApp', ['helperModule']);

**'helperModule'** - module that demoApp depends on

#### Example A: using anonymous function

    var demoApp = angular.module('demoApp', []);

    demoApp.controller('SimpleController', function ($scope) {
      $scope.customers = [
        {name:'John', city:'NY'},
        {name:'Mark', city:'ZG'},
        {name:'Bob', city:'RI'}
      ];
    });

#### Example B: creating controller outside the .controller call

    var demoApp = angular.module('demoApp', []);

    function SimpleController($scope) {
      $scope.customers = [
        {name:'John', city:'NY'},
        {name:'Mark', city:'ZG'},
        {name:'Bob', city:'RI'}
      ];
    }

    demoApp.controller('SimpleController', SimpleController)

#### Example C: Using .controllers option (for prototyping)

    var demoApp = angular.module('demoApp', []);

    var controllers = {};

    controllers.SimpleController = function ($scope) {
      $scope.customers = [
        {name:'John', city:'NY'},
        {name:'Mark', city:'ZG'},
        {name:'Bob', city:'RI'}
      ];
    }

    demoApp.controller(controllers)

## Routes
