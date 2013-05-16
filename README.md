# Basic Angular tutorial app

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

You can find a whole bunch of directives at the [official docs](http://docs.angularjs.org/api/)

## Filters

    <li ng-repeat="person in people | filter:name | orderBy:'city'">{{ person }}</li>

Upper line filters people based on what you type in the input.

**You can write your own custom filters and directives.**

## Views, Controllers and Scope

VIEW <--- $scope ---> CONTROLLER
