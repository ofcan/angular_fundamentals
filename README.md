# Angular Basics (and a small app)

Angular is very very modular if you take advantage of it.
Great for **SPA** apps

* [Modules](#modules)
* [Directives and Data-bindings](#directives-and-data-bindings)
    * [Basic behaviours](#basic-behaviours)
    * [Directive to Directive communication](#directive-to-directive-communication)
    * [The Dot](#the-dot)
    * [The link function](#the-link-function)
* [Isolate Scope](#isolate-scope)
    * [Isolate Scope @](#isolate-scope-at)
    * [Isolate Scope =](#isolate-scope-equals)
    * [Isolate Scope &](#isolate-scope-and)
* [Transclusion](#transclusion)
* [Angular Element](#angular-element)
    * [Compile](#compile)
* [Filters](#filters)
* [TemplateUrl](#templateUrl)
* [Views](#views)
* [Controllers and Scope](#controllers-and-scope)
    * [Creating Controllers](#creating-controllers)
    * [Sharing code between controllers](#sharing-code-between-controllers)
    * [Directives talking to controllers](#directives-talking-to-controllers)
    * [Scope vs scope](#scope-vs-scope)
* [Routes](#routes)
* [Factories, Services, Providers, Values](#factories-services-providers-values)
* [Testing](#testing)
* [Good Practices](#good-practices)
    * [Don't reference scope inside of the function](#dont-reference-scope-inside-of-the-function)
    * [Restrict custom directives to attributes](#restrict-custom-directives-to-attributes)
* [Examples](#examples)

![General overview](/images/general_overview.png)

### Usage

Just clone the thing and run index.html in your browser. If you're looking at
your app in chromium-browser, you might not be able to see it, since security
parameters prohibit browsing file:// URL.  To fix this set up a small server in
the folder of this app like so:

    python -m SimpleHTTPServer 8000

## Modules

![Module](/images/module.png)

**Module** is like an object container. This is how we create modules:

    var demoApp = angular.module('demoApp', ['helperModule']);

**'helperModule'** - module that demoApp depends on

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

### The link function

    function link(scope, iElement, iAttrs, controller) { ... }

The link function is responsible for registering DOM listeners as well as
updating the DOM. It is executed after the template has been cloned. **This is
where most of the directive logic will be put.**

* scope - Scope - The scope to be used by the directive for registering watches.

* iElement - instance element - The element where the directive is to be used.
It is safe to manipulate the children of the element only in postLink function
since the children have already been linked.

* iAttrs - instance attributes - Normalized list of attributes declared on this
element shared between all directive linking functions. See Attributes.

* controller - a controller instance - A controller instance if at least one
directive on the element defines a controller. The controller is shared among
all the directives, which allows the directives to use the controllers as a
communication channel.

## Isolate Scope

![Scopes](/images/scopes.png)

Immediately when you call a scope upon defining a directive you separate it from
parent scope.
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

    <div drink></div>

    var app = angular.module('drinkApp', []);

    app.controller('appController', function ($scope) {
    };

    app.directive('drink', function () {
      return {
        template: '<div>{{ flavor }}</div>'
        link: function (scope) {
          scope.flavor = 'cherry';
        }
      }
    });

Above just returns > cherry

    <div drink flavor='strawberry'></div>

    var app = angular.module('drinkApp', []);

    app.controller('appController', function ($scope) {
    };

    app.directive('drink', function () {
      return {
        template: '<div>{{ flavor }}</div>'
        link: function (scope, element, attrs) {
          scope.flavor = attrs.flavor
        }
      }
    });

We can replace the directive above like this and still get the same result:

    app.directive('drink', function () {
      return {
        scope:{
          flavor:'@'
        }
        template: '<div>{{ flavor }}</div>'
      }
    });

### Isolate scope equals

Set up a binding both ways, meaning anything that is updated in the directive
itself will also update everything in the controller scope.

    <div drink flavor=Flavor></div>

    var app = angular.module('drinkApp', []);

    app.controller('appController', function ($scope) {
      $scope.Flavor = 'blackberry'
    };

    app.directive('drink', function () {
      return {
        scope:{
          flavor:'='
        }
        template: '<div>{{ flavor }}</div>'
      }
    });

Now the thing doesn't expect the string but the object to bind to.

### Isolate scope and

Allows you to invoke, evaluate, call etc an expression on the parent scope of
whatever directive it is inside of.

    <div phone dial='callHome(message)'></div>

    var app = angular.module('drinkApp', []);

    app.controller('appContrller', function ($scope) {
      $scope.callHome = function (message) {
        alert(message)
      }
    });

    app.directive('phone', function () {
      return {
        scope:{
          dial:"&" // means if we invoke dial and pass in an object (like we do
                   // below) with message as the name, and value as the value,
                   // than we can communicate with controller from the directive
        },
        template:'<input type='text ng-model='value'>' +
                 '<div class='button' ng-click='dial({message:value})'>Call home!</div>'
      }
    }

## Transclusion

Transclusion basicly yanks 'What' from the example below into wherever you
define ng-trancslude. Without transclusion, 'What' text would have been
completely overwritten.

    <hero>
      <p>What</p>
    </hero>

    var app = angular.module('App', []);

    app.controller('appContrller', function ($scope) {
    });

    app.directive('hero', function () {
      return {
        restrict: 'E',
        transclude: true,
        template: '<div class='hero' ng-transclude>Hello</div>
      }
    });

## Angular Element

Let's examine how angular elements differ from elements in other frameworks
(like JQ for example)

    <dumb-password></dumb-password>

    var app = angular.module('app', []);

    app.directive('dumbPassword', function() {
      return {
        restrict: 'E',
        replace: true,
        template: '<div><input type='text' ng-model='model.input'><span></span></div>'
        link: function (scope) {
          scope.$watch('model.input', function (value) {
            if (value === 'password') {
              element.children(1).toggleClass('new-class')
            }
          })
        }
      }
    });

In the example above, dumb-password element is going to be totally replaced with
input field because of the 'replace: true' option.

Also, within the link function, we are watching for the input field the user is
typing in and observing if it matches the word 'password'. If it does, we add
the class 'new-class' to the 1st child element of that input field (wich is the
'span' element in our example).

Btw, this children(1) is a nasty way of looking for things. Typically people use
.find() or something similar. But .find() is not supported within angular so you
might use something like compile...

### Compile

The best way to understand the compile function is just to look at the example
below:

    <dumb-password></dumb-password>

    var app = angular.module('app', []);

    app.directive('dumbPassword', function() {

      var valid_element = angular.element('<span></span>');

      return {
        restrict: 'E',
        replace: true,
        template: '<div><input type='text' ng-model='model.input'></div>'
        compile: function(template_element) {
          template_element.append(valid_element);
          return function (scope, element) {
            scope.$watch('model.input', function (value) {
              if (value === 'password') {
                valid_element.toggleClass('new-class')
              }
            })
          }
        }
      }
    });


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

## TemplateUrl

Templates you create with directives and such usually are much much more
complicated for a single strings, so angular uses templateUrl!

    <my-directive></my-directive>

    var app = angular.module('myApp', []);

    app.directive('myDirective', function (scope) {
      restrict: 'E',
      return {
        templateUrl:'my_template.html',
      }
    });

In the example above, everything inside my_template.html is going to render
inside my-directive div.

On a more technical level, [the templateUrl is just looking up the template cache
with the corresponding key.](http://www.egghead.io/video/boBm3AU-uX4)

## Views

This is why Angular kick ass when it comes to SPA apps:

    <ng-view></ng-view>

    var app = angular.module('myApp', []);

    app.config(function($routeProvider) {
      $routeProvider
        .when('/',
          {
            templateUrl: app.html,
            controller: appController'
          })
        .when('pizza',
          {
            template: 'Yum!',
          })
        .otherwise({
          template: 'This doesnt exist'
        })
    });

    app.controller('appController, function ($scope) {
      $scope.model = {
        message: 'This is my app!'
      }
    });

And this is the content of app.html:

    <h1>{{ model.message }}</h1>

## Controllers and Scope

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

### Creating Controllers

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

Btw, in the same fashion you can create directives and the like (doesn't work on
filters though).

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

### Scope vs scope

Take a look at the example below. What is the difference between $scope and
scope?

    var app = angular.module('myApp', []);

    app.controller('myController', function ($scope) {
    });

    app.directive('myDirective', function (scope) {
      return {
        link: function (scope) {
        }
      }
    });

In the directive, when we define `(scope, element)` etc, **the order matters**.
You can name those things whatever you wish, they'll still mark scope,
element...

On the other hand, in the controller, when we define `($scope)`, that is the
angular variable - meaning you cannot name it whatever you want, but you can
order them in any way that suits you.

But in this case, they both mark the same thing / the same scope. :)

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

### Route params

    <h1>{{ model.message }}</h1>

    var demoApp = angular.module('demoApp', []);

    demoApp.config(function ($routeProvider) {
      $routeProvider
        .when('/map/:country/:city',
          {
            controller: 'SimpleController',
            templateUrl: 'app.html'
          })
    });

    demoApp.controller('simpleController', function ($scope, $routeParams) {
      $scope.model = {
        message: 'Address: ' + 
          $routeParams.country + ', ' +
          $routeParams.city
      }
    });

Visiting /map/croatia/zagreb url types out on the page 'Address: croatia,
zagreb'

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

## Examples

[Zippy with Angular](http://www.youtube.com/watch?v=JeuhhPlOZs0)
