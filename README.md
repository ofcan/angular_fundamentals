# Angular Basics (and a small app)

### Overview

![General overview](/images/general_overview.png)

* [Directives and Data-bindings](#directives-and-data-bindings)
    * [Basic behaviours](#basic-behaviours)
    * [Directive to Directive communication](#directive-to-directive-communication)
    * [The Dot](#the-dot)
* [Isolate Scope](#isolate-scope)
    * [Isolate Scope @](#isolate-scope-at)
    * [Isolate Scope =](#isolate-scope-equals)
    * [Isolate Scope &](#isolate-scope-and)
* [Filters](#filters)
* [Views, Controllers and Scope](#views-controllers-and-scope)
    * [Sharing code between controllers](#sharing-code-between-controllers)
    * [Directives talking to controllers](#directives-talking-to-controllers)
* [Modules](#modules)
* [Routes](#routes)
* [Factories, Services, Providers, Values](#factories-services-providers-values)
* [Testing](#testing)
* [Good Practices](#good-practices)
    * [Don't reference scope inside of the function](#dont-reference-scope-inside-of-the-function)
    * [Restrict custom directives to attributes](#restrict-custom-directives-to-attributes)

Angular is very very modular if you take advantage of it.
Great for **SPA** apps

### Usage

Just clone the thing and run index.html in your browser. If you're looking at
your app in chromium-browser, you might not be able to see it, since security
parameters prohibit browsing file:// URL.  To fix this set up a small server in
the folder of this app like so:

    python -m SimpleHTTPServer 8000

## Directives and Data Bindings

They teach HTML new tricks

    <html ng-app>
    <input type='text' ng-model='search_query' />
    {{ search_query }}

* **ng-app** - directive defining a default angular module
* **ng-model** - another directive
* **'search_query'** - the name of the property
* **{{ search_query }}** - data binding expression

You can put data bindings anywhere in html, like for example:

    <p class='{{ search_query }}'>

**You shouldn't put any logic in your views!**

Another neat directive is **ng-repeat**:

    <body ng-init="people=[{name:'John', city:'NY'},
                           {name:'Mark', city:'ZG'},
                           {name:'Bob', city:'RI'}]">

    <ul>
      <li ng-repeat="person in people">{{ name }}</li>
    <ul>

**It is a good practice to make directives like so: data-ng-model=''** because
of how data attributes work in html5.  You can find a whole bunch of directives
at the [official docs](http://docs.angularjs.org/api/)

You can also create your very own custom directives:

    <div ng-app='app'>
      <superman></superman>
    </div>


    var app = angular.module("superhero", []);

    app.directive("superman", function () {
      return {
        restrict: "E",
        template: "<div>Here I am to save the day</div>"
      }
    });

Upper code just types 'here i am...' in the div on the page.

**E** - means directive is restricted to an 'element'. You can have a whole
bunch of these ( C-class, M-comment, A-attribute ). It is best practice to
restrict directives to attributes when it comes to adding behaviours. Let's add
so that when you click on a paragraph a flash message appears:

    <div ng-app='app'>
      <p superman></p>
    </div>

    var app = angular.module("superhero", []);

    app.directive("superman", function () {
      return {
        restrict: "A",
        link: function () {
          alert('Hiya')
        }
      }
    });

It you don't specify a restriction, it defaults to 'A'.

### Basic behaviours

Events on mouseenter and mouseleave:

    <div enter leave>Mouse over me and see!</div>

    var app = angular.module('behaviorApp', []);

    app.directive('enter', function() {
      return function (scope, element) {
        element.bind('mouseenter', function () {
          console.log('Eagle has landed');
        }
      }
    });

    app.directive('leave', function () {
      return function (scope, element) {
        element.bind('mouseenter', function () {
          console.log('Eagle flew away');
        }
      }
    });

Let's make something more useful than above and add classes on mouse events:

    <div enter='hero' leave>Mouse over me and see!</div>

    var app = angular.module('behaviorApp', []);

    app.directive('enter', function() {
      return function (scope, element, attrs) {
        element.bind('mouseenter', function () {
          element.addClass(attrs.enter);
        }
      }
    });

    app.directive('leave', function () {
      return function (scope, element, attrs) {
        element.bind('mouseenter', function () {
          element.removeClass(attrs.enter);
        }
      }
    });

### Directive to Directive communication

    <superhero strenght speed flight>The Superman</superhero>
    <superhero speed>The Flash</superhero>

    var app = angular.module('superApp', []);


    app.directive('superhero', function () {
      return {
        restrict: 'E',
        // We need to set up isolate scope
        // without it, moving your mouse over various superheroes would just
        // show the attributes from the 1st superhero you mouse over (it
        // wouldn't update the attributes )
        scope:{};

        // You can make custom controllers for directives
        // This is a sort of an API controller
        controller: function ($scope) {
          $scope.abilities = [];

          this.addStrength = function () {
            $scope.abilities.push("strength")
          }

          this.addSpeed = function () {
            $scope.abilities.push("speed")
          }

          this.addFlight = function () {
            $scope.abilities.push("flight")
          }

        },

        link: function (scope, element) {
          element.bind('mouseenter', function () {
            console.log(scope.abilities);
          })
        }
      }
    };

    app.directive('strength', function () {
      return {
        require: 'superhero',
        // line above gives us access to superheroCtrl below
        link: function (scope, element, attrs, superheroCtrl) {
          superheroCtrl.addStrength();
        }
      }
    });
    app.directive('speed', function () {
      return {
        require: 'superhero',
        link: function (scope, element, attrs, superheroCtrl) {
          superheroCtrl.addSpeed();
        }
      }
    });
    app.directive('flight', function () {
      return {
        require: 'superhero',
        link: function (scope, element, attrs, superheroCtrl) {
          superheroCtrl.addFlight();
        }
      }
    });

### The Dot

'The Dot' is important for scope inheritance.

    <div ng-app=''>
      <input type='text' ng-model='data.message' />
      <p>{{ data.message }}</p>
    </div>

**data** - is the object we created
**message** - is the property of that object

The Dot is extremly important when it comes to **inheritance**. In the case
below, whatever you type in 1st input field will be copyed across all
paragraphs.

    <div ng-app=''>
      <input type='text' ng-model='data.message' />
      <p>{{ data.message }}</p>
      <div ng-controller='firstController'>
        <input type='text' ng-model='data.message' />
        <p>{{ data.message }}</p>
      </div>
      <div ng-controller='secondController'>
        <input type='text' ng-model='data.message' />
        <p>{{ data.message }}</p>
      </div>
    </div>

However, if we omit the data.message and just create ng-model='message', the
paragraphs in different controllers loose the inheritance and each has a
separate message

    <div ng-app=''>
      <input type='text' ng-model='message' />
      <p>{{ message }}</p>
      <div ng-controller='firstController'>
        <input type='text' ng-model='message' />
        <p>{{ message }}</p>
      </div>
      <div ng-controller='secondController'>
        <input type='text' ng-model='message' />
        <p>{{ message }}</p>
      </div>
    </div>

## Isolate Scope

It is best to observe isolate scope on a simple example

    <div ng-app='choreApp'>
      <kid></kid>
      <kid></kid>
      <kid></kid>
    </div>

    var app = angular.module('choreApp', []);

    app.directive('kid', function () {
      return {
        restirct: 'E',
        template: '<input type='text' ng-model='chore' /> {{ chore }}
      }
    });

In the example above, when you write in one kid's input, you write in all of
them. That is solvable by using isolate scope like so:

    app.directive('kid', function () {
      return {
        restirct: 'E',
        scope:{},
        template: '<input type='text' ng-model='chore' /> {{ chore }}
      }
    });

Now when you write in one kids input, you edit just his input, not everybody
elses.
**But be careful** as this breaks the interaction with the controllers who wrap
around it.

    <div ng-app='choreApp'>
      <div ng-controller='choreController'>
        <kid done='logChore(chore)'></kid>
      </div>
    </div>

    var app = angular.module('choreApp', []);

    app.controller('choreController', function ($scope) {
      $scope.logChore = function (chore) {}
        alert(chore + ' is done!')
    });

    app.directive('kid', function () {
      return {
        restirct: 'E',
        scope:{
          done:"&" // & sign is for expression
        },
        template: '<input type='text' ng-model='chore' />' +
                  '<a class='btn' ng-click='done({chore:chore})'>Task completed</a>

                  // done() is mapped to whatever is passed into done:"&" above
                  // 1st chore in chore:chore is the property
                  // 2nd chore in chore:chore corresponds to ng-model='chore'
                  // chore:chore makes it possible to accept chore from the view
                  //    to chore in the controller
      }
    });

    // so now when you click on 'Task completed' it says
    // for example: 'Clean your room is done!'

### Isolate scope at
### Isolate scope equals
### Isolate scope and

## Filters

    <input type='text' ng-model='search' />
    <li ng-repeat="person in people | filter:search | orderBy:'city' | limitTo: 5 ">{{ person }}</li>

Upper line filters people's names and people's cities at the same time, based on
what you type in the input. If you wish to specify what to search for (example,
search only by the name), the syntax is this:

    <input type='text' ng-model='search.name' />
    <li ng-repeat="person in people | filter:search | orderBy:'city'">{{ person }}</li>

**You can write your own custom filters and directives.**

    myApp.filter('reverse', function () {
      return function (text) {
        return text.reverse();
      };
    });

    > This
    >> sihT

Ofcourse, you can inject data into your custom filters

    myApp.filter('reverse', function (Data) {
      return function (text) {
        return text.reverse() + Data.message;
      };
    });

    > This
    >sihTMessage

Angular provides A LOT of default filters:

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

### Sharing code between controllers

    var myApp = angular.module('myApp', []);

    myApp.factory('Data', function () {
      return { message: "I'm data from the factory" }
    });

    function firstController ($scope, Data) {
      $scope.data = Data;
    }

    function secondController ($scope, Data) {
      $scope.data = Data;
    }

We say that 'we have injected Data factory into the Controllers'.

### Directives talking to controllers

Here's a basic example:

    <div ng-app='twitterApp'>
      <div ng-controller='appController'>
        <div enter>Roll over to load more Tweets</div>
      </div>
    </div>

    var app = angular.module('twitterApp', []);

    app.controller('appController', function ($scope) {
      $scope.loadMoreTweets = function () {
        alert("Loading Tweets")
      }
    });

    app.directive('enter', function () {
      return function (scope, element, attrs) {
        element.bind('mouseenter', function () {
          scope.loadMoreTweets();
          // above is the same as:
          // scope.$apply(loadMoreTweets())
        });
      }
    });

And here's that example improved. It is much more modular and easier to
maintain.

    <div ng-app='twitterApp'>
      <div ng-controller='appController'>
        <div enter='loadMoreTweets()'>Roll over to load more Tweets</div>
      </div>
    </div>

    var app = angular.module('twitterApp', []);

    app.controller('appController', function ($scope) {
      $scope.loadMoreTweets = function () {
        alert("Loading Tweets")
      }
    });

    app.directive('enter', function () {
      return function (scope, element, attrs) {
        element.bind('mouseenter', function () {
          scope.$apply(attrs.enter);
        });
      }
    });

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

![Routes scheme](/images/views.png)

    var demoApp = angular.module('demoApp', []);

    demoApp.config(function ($routeProvider) {
      $routeProvider
        .when('/',
          {
            controller: 'SimpleController',
            templateUrl: 'view1.html'
          })
        .when('/partial2',
          {
            controller: 'SimpleController',
            templateUrl: 'view2.html'
          })
        .otherwise({ redirectTo: '/' });
    });

## Factories, Services, Providers, Values

These are just different ways to get data for your app.  Well just be looking at
**Factories**

    var demoApp = angular.module('demoApp', [])
      .factory('simpleFactory', function () {
        var factory = {};
        var customers = [ ... ];
        factory.getCustomers = function () {
          # in real life this calls a db; sends ajax call, whatever
          return customers;
        };
        return factory;
      })
      .controller('SimpleController, function ($scope, simpleFactory) {
          $scope.customers = simpleFactory.getCustomers();
    });

Above, we use a technique called 'chaining' to chain factory and controller
right after defining module.

## Testing

Most commonly used test suite for Angular is Karma + Jasmine.

Here's an example. The wording is very similar to rspec tests in Rails:

    describe('filter', function () {
      beforeEach(module('myApp'));

      describe('reverse', function () {
        it('should reverse a string', inject(function (reverseFilter) {
          expect(reverseFilter('ABCD')).toEqual('DCBA');
          expect(reverseFilter('John')).toEqual('nhoJ');
        }));
      });
    });

Upper tests are tests for the following filter:

    myApp.filter('reverse', function () {
      return function (text) {
        return text.reverse();
      };
    });

## Good practices

#### Don't reference scope inside of the function

Bad practice:

    {{ reversedMessage }}

    $scope.reversedMessage = function () {
      return $scope.message.reverse();
    };

Good practice:
This removes the dependency of our scope and makes it easier to test and all that stuff.

    {{ reversedMessage(message) }}

    $scope.reversedMessage = function (message) {
      return message.reverse();
    }

#### Restrict custom directives to attributes

It is best practice to restrict directives to attributes when it comes to adding
behaviours. Let's add so that when you click on a paragraph a flash message
appears:

    <div ng-app='app'>
      <p superman></p>
    </div>

    var app = angular.module("superhero", []);

    app.directive("superman", function () {
      return {
        restrict: "A",
        link: function () {
          alert('Hiya')
        }
      }
    });
;
