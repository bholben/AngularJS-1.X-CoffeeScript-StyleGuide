# AngularJS 1.X CoffeeScript Style Guide

*Opinionated AngularJS 1.X style guide for teams by [@rbholben](//twitter.com/rbholben)*

The purpose of this style guide is to propose structure and conventions for scalable team-built Angular 1.X CoffeeScript applications.  It can also serve as a syntactical "cheat sheet" for how to write various Angular code snippets.  This guide has been forked from and heavily influenced by Todd Motto's [AngularJS style guide](https://github.com/toddmotto/angularjs-styleguide) and customized to work with CoffeeScript.

The [`yo ng-poly`](https://github.com/dustinspecker/generator-ng-poly) project is the closest Yeoman generator I have found for scaffolding a project that fits these conventions.  This generator is very flexible, providing a plethora of options (HTML, CSS, JavaScript, Jade, Sass, Less, Stylus, CoffeeScript, et al).

***A note about style guides:***
The goal here is to set a starting point.  Any team can start with this document and customize it to meet their needs.  Keep in mind that there are many ways to accomplish the same end goal.  Techniques defined below are merely one way.  The important thing is that teams agree on their own style early in the process.  Since much more time is spent reading code rather than writing code, team members that don't fit in with a common convention can greatly reduce the productivity of the team as a whole.

***A note about Angular methods:***
Angular apps are built with a handful of primary methods provided by Angular.  The frequently used methods are `.controller`, `.factory`, `.service`, `.constant`, `.value`, `.provider`, `.directive`, and `.filter`.  When this guide refers to "Angular methods," we mean these Angular methods.


## Table of Contents

  1. [General CoffeeScript](#general-coffeescript)
  1. [Angular Naming Conventions](#angular-naming-conventions)
  1. [Code Structure](#code-structure)
  1. [Controllers](#controllers)
  1. [Services](#services)
  1. [Directives](#directives)
  1. [Filters](#filters)
  1. [Comment Standards](#comment-standards)
  1. [Minification & Annotation](#minification--annotation)
  1. [File & Folder Conventions](#file--folder-conventions)
  1. [Tips & Tricks](#tips--tricks)
    1. [Publish & Subscribe (Pub/Sub) Events](#publish--subscribe-pubsub-events)
    1. [Performance](#performance)
    1. [Angular Wrapper References](#angular-wrapper-references)


## General CoffeeScript

  - **Parentheses:** In CoffeeScript, parentheses are optional in many situations.  Favor the approach without parentheses.

    ```coffeescript
    # avoid
    angular.module('someApp')

    # recommended
    angular.module 'someApp'
    ```

  - `@` is CoffeeScript shorthand for `this`.  Favor `@`.

  - **IIFE scoping:** CoffeeScript automatically compiles to JavaScript wrapped inside an IIFE (immediately invoked function expression).  This ensures that the global namespace will not be polluted.  It is not necessary to add any additional IIFE.

    This CoffeeScript...

    ```coffeescript
    'use strict'

    someVar = 'My string!'
    ```

    compiles into this JavaScript...

    ```coffeescript
    (function() {
      'use strict'
      var someVar;

      someVar = 'My string!';

    }).call(this);
    ```

**[Back to top](#table-of-contents)**


## Angular Naming Conventions

  - **lowerCamelCase:** Use lowerCamelCase for all directives and filters.  Angular requires them to be this way in order to translate into a hyphenated html name, i.e. `someDirective` is translated into `some-directive`.

    ```coffeescript
    # required
    angular.module 'someApp'
    .directive 'someDirective', ...

    angular.module 'someApp'
    .filter 'someFilter', ...
    ```

  - **UpperCamelCase:** Use UpperCamelCase for all other Angular methods.

    ```coffeescript
    # recommended
    angular.module 'someApp'
    .controller 'SomeCtrl', ...

    angular.module 'someApp'
    .service 'SomeService', ...

    angular.module 'someApp'
    .constant 'SomeConstant', ...

    angular.module 'someApp'
    .value 'SomeValue', ...

    angular.module 'someApp'
    .provider 'SomeProvider', ...
    ```

**[Back to top](#table-of-contents)**


## Code Structure

  - **Module Setter & Getters:**

    `angular.module 'someApp', []` sets a module.

    `angular.module 'someApp'` gets the previously set module.

    A module must only be set once.  Do this in a standalone file.

      ```coffeescript
      # avoid
      angular.module 'someApp', [
        ...
        ...
      ]
      .config...
      .run...

      # recommended
      angular.module 'someApp', [
        ...
        ...
      ]
      ```

    For anything more than a simple app, consider breaking the app into multiple modules.  This does not mean that every file needs its own module, but it may be sensible to break it into logical chunks.

  - **One Angular Method per File:** Only put one Anguler method per file.  This consistent approach will keep your development project more navigable and predictable.  Build tools will ultimately concatenate into a single file for production.

  - **Angular Method Chaining:** Assigning the the returned module to a variable is not the preferred approach. Instead, chain your Angular method onto `angular.module 'someApp'` (the getter).  This makes each Angular file consistent and predictable.  It reduces the amount of cross-file searching to identify a variable.

    ```coffeescript
    # avoid
    someApp = angular.module 'someApp', []
    ```
    ```coffeescript
    # avoid
    someApp.controller ...
    ```


    ```coffeescript
    # recommended
    angular.module 'someApp', []
    ```
    ```coffeescript
    # recommended
    angular.module 'someApp'
    .controller 'SomeCtrl', ...
    ```

  - **Angular Method Indentation:** Don't bother indenting the Angular method.  This causes unnecessary whitespace for all that follows.  You may find this to be your editor's default behavior.

    ```coffeescript
    # avoid
    angular.module 'someApp'
      .controller 'SomeCtrl', ...

    # recommended
    angular.module 'someApp'
    .controller 'SomeCtrl', ...
    ```

  - **Using Classes:** CoffeeScript functions do not support JavaScript function declaration syntax, however CoffeeScript classes do.  This nuance allows us to use CoffeeScript classes to organize our code in a readable top-to-bottom way and take advantage of hoisting.

    ```coffeescript
    # avoid
    angular.module 'someApp'
    .controller 'SomeCtrl', -> ()
      ...

    # recommended
    angular.module 'someApp'
    .controller 'SomeCtrl', class SomeCtrl
      constructor: (SomeService) ->

        @publicVar = 'some string'
        privateVar = 'some string'

        @publicMethod = (someParam) ->
          @someVar1 = SomeService.someMethod1(someParam1)

        privateMethod = (someParam) ->
          @someVar2 = SomeService.someMethod2(someParam2)
    ```

    At first glance, this appears to only instantiate one controller instance, but don't let this trip you up.  Angular manages the controller instances, not this code.  The same goes for services (singletons).  Angular will manage instantiation.

    Use `@` to dictate which variables and methods are exposed.

  - **Match Your Names:** The string you use to define your Angular method should match the function (or class) that it calls.

    ```coffeescript
    # avoid
    angular.module 'someApp'
    .controller 'SomeCtrl', class SomeCoolCtrl

    # recommended
    angular.module 'someApp'
    .controller 'SomeCtrl', class SomeCtrl
    ```

  - **Strict Mode:** Write all files using [strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode).  The first line in each CoffeeScript (or JavaScript) file should be `'use strict'`.

  - **Angular code at top of each file:** Place the Angular module and provider lines at the top of each file.  This makes it immediately evident to the viewer what the code is doing.

  - **Putting it all Together:**

    Module setter file...

    ```coffeescript
    angular.module 'someApp', [
      ...
      ...
    ]
    ```

    Angular method file (same approach for [controller](#controllers), [service](#services), [directive](#directives), etc.)...

    ``` coffeescript
    'use strict'

    angular.module 'someApp'
    .controller 'SomeCtrl', class SomeCtrl
      constructor: $log, someService ->

        @someVar1 = someService.someSvcMethod1()
        @someVar2 = someService.someSvcMethod2()

    ```

**[Back to top](#table-of-contents)**


## Controllers

  - **Ctrl:** Most variables should be spelled out.  We make an exception for controllers since they are used everywhere and the word is excessively long.  Use "Ctrl".

    ```coffeescript
    angular.module 'someApp'
    .controller 'SomeCtrl', ...
    ```

  - **controllerAs syntax:** Controllers are classes, so use the `controllerAs` syntax at all times.  The `controllerAs` syntax uses `this` inside controllers, which gets bound to `$scope`.  `controllerAs` especially shines with nested controllers as it makes all template variables explicitly clear.  Using this approach will also make the eventual transition to Angular 2.0 smoother.

    ```html
    <!-- avoid -->
    <div ng-controller="SomeCtrl">
      {{ someObject }}
    </div>

    <!-- recommended -->
    <div ng-controller="SomeCtrl as sc">
      {{ sc.someObject }}
    </div>
    ```

  - **Fat Arrow:** Use CoffeeScript's "fat arrow" syntax to pass outer scope `this` context into a nested function.  Do not bind `vm` to `this` when using CoffeeScript.

    ```coffeescript
    @someVar = (query) => SomeService.someVar(query).then =>
      @arr = SomeService.someObject
    ```

    Use CoffeeScript's regular arrow if no nested functions.

    ```coffeescript
    @someVar = -> SomeService.someObject
    ```

  - **Avoid `$scope`:** Only use `$scope` when necessary; for example, publishing and subscribing events using `$emit`, `$broadcast`, `$on` or `$watch`. Try to limit the use of these, however, and treat `$scope` as a special use case.

  - **No Business Logic:** The only logic in a controller should be presentation logic.  Avoid business logic (this should only live in a [service](#services)).

    ```coffeescript
    # avoid
    angular.module 'someApp'
    .controller 'SomeCtrl', class SomeCtrl
      constructor: ->

        @retrieve = $http.get('/somepath').success (response) =>
          @data = response

        @delete = someObject, index => $http.delete('/someString/' + someObject.id).then (response) =>
          @data.splice index, 1

    # recommended
    angular.module 'someApp'
    .controller 'SomeCtrl', class SomeCtrl
      constructor: SomeService ->

        @retrieve = SomeService.get().then =>
          @data = SomeService.someObject

        @delete = someObject, index => SomeService.delete(someObject).then =>
          @data.splice index, 1
    ```

  - **No DOM manipulation:** Avoid DOM manipulation (this should only live in a [directive](#directives)).

  - **Quick Summary Example:**

    See [code structure](#code-structure) explanation for using classes, the constructor, and exposing variables.

    ``` coffeescript
    'use strict'

    angular.module 'someApp'
    .controller 'SomeCtrl', class SomeCtrl
      constructor: $log, someService ->

        @someVar1 = someService.someSvcMethod1()
        @someVar2 = someService.someSvcMethod2()

    ```



**[Back to top](#table-of-contents)**


## Services

  - **Use `.service` instead of `.factory`:** All Angular services are singletons.  Usage of `.service` or `.factory` is purely a preference and each provides a different way to create objects.  Favor services over factories since the syntax exactly matches the way that [controllers are defined above](#controllers).  Build muscle memory.

  - **Quick Summary Example:**

    See [code structure](#code-structure) explanation for using classes, the constructor, and exposing variables.

    ```coffeescript
    'use strict'

    angular.module 'someApp'
    .service 'SomeService', class SomeService
      constructor: ($http, $q, $log) ->

        @get = =>
          deferred = $q.defer()
          $http.get()
          .success (data) =>
            deferred.resolve(data)
            @someObject = data
          .error (data, status) ->
            $log.error 'Error: ', status, data
    ```

**[Back to top](#table-of-contents)**


## Directives

  - **KISS Principle:** Only include the high level information at the top of your directive.  Don't make directives more complicated than they need to be.  Put details in classes further down in the code.

    ```coffeescript
    angular.module 'directives'
    .directive 'someDirective', ->
      restrict: 'E'
      templateUrl: 'app/some-directive.html'
      controller: someDirectiveCtrl
      controllerAs: 'sd'
      link: someDirectiveLink

    class someDirectiveCtrl
      constructor: ->
        @someVar1 = 'some string'

    class someDirectiveLink
      constructor: (scope, elem, attr) ->
        @someVar2 = attr.someAttr
    ```

  - **Declaration restrictions:** Only create element or attribute directives (`restrict: 'EA'`).  Avoid old-style directives (class and comment).

  - **Choosing Directive Type:** Simple guideline... If your directive contains a template, use an element directive.  Otherwise, use an attribute directive.  The default value for `restrict` is `EA`; this is fine, but keep this guideline in mind when coding your html.

    ```html
    <!-- when the directive has a template or templateURL property -->
    <some-directive></some-directive>

    <!-- when the directive has no template -->
    <div some-directive></div>
    ```

  - **Templating:** Use `Array.join ''` for a cleaner, easier-to-read template.

    ```coffeescript
    # avoid
      template: '<div>' +
          '<h1>My directive</h1>' +
        '</div>'

    # recommended
      template: [
        '<div>'
          '<h1>My directive</h1>'
        '</div>'
      ].join ''
    ```

    For longer templates, use `templateUrl` and move to a separate file.  This further improves readability and opens up additional editor features.

  - **DOM Manipulation:** Should only take place inside a directives, never a controller or service.

    ```coffeescript
    # avoid
    angular.module 'someApp'
    .controller 'UploadCtrl', class UploadCtrl
      constructor:
        $('.dragzone').on 'dragend', function ->
          ...

    # recommended
    angular.module 'someApp'
    .directive 'dragUpload',  ->
      link: DragUploadLink

    class DragUploadLink
      constructor: (scope, element, attrs) ->
        element.on('dragend', ->
          ...
    ```

  - **Naming conventions:** Never prefix custom directives with `ng-`, they might conflict with future native directives.  It is recommended to prefix all custom directives with company or project-specific characters to reduce the liklihood of naming collisions with 3rd party directives.  Also see [general name convention comment](#angular-naming-conventions) about lowerCamelCasing directive names.

  - **controllerAs:** Use the `controllerAs` syntax inside Directives as well.

**[Back to top](#table-of-contents)**


## Filters

  - **Global filters:** Create global filters using `angular.filter()` only.  Never use local filters inside controllers or services so as to enhance testing and reusability.

    ```coffeescript
    # avoid
    angular.module 'someApp'
    .controller 'SomeCtrl', class SomeCtrl
      constructor: ->

        @startsWithLetterA = (items) ->
          items.filter (item) ->
            /^a/i.test item.name


    # recommended
    angular.module 'someApp'
    .filter 'startsWithLetterA', class startsWithLetterA
      constructor: ->

        (items) ->
          items.filter (item) ->
            /^a/i.test item.name
    ```

  - See [general name convention comment](#angular-naming-conventions) about lowerCamelCasing filter names.

**[Back to top](#table-of-contents)**


## Comment Standards

  **jsDoc:** Use jsDoc syntax to document function names, description, params and returns.

  ```coffeescript
  ###*
   # @name Http

   # @desc
   # Send http requests to server.
   #
   ###

  angular.module 'someApp'
  .service 'Http', class Http
    constructor: ($http, $q, $log) ->

      ###*
       # @name get
       # @desc Send http get request to API endpoint
       # @param {String} url - URI to direct request to
       # @param {...} ... - ...
       # @returns {Object} promise - ...
       ###

      @get = (url, ...) ->
        deferred = $q.defer()
        ...
        $http.get()
        .success (data) ->
          deferred.resolve(data)
  ```

**[Back to top](#table-of-contents)**


## Minification & Annotation

  - Do not use Angular array syntax or `.$inject` in your code.  Neither technique is DRY.  They both reduce readability, increase unnecessary complexity to your code, and increase the potential for mistakes.  Let your build system handle this for you.

  - Use [ng-annotate](//github.com/olov/ng-annotate) for Gulp or Grunt (`ng-min` is deprecated).  This will protect your code from minification routines which shorten variable names and break the app.  Function declarations will use Angular's `.$inject` notation and function expressions will use Angular bracket notation.  Adding `@ngInject` comments to your code will explicitly require `.$inject` notation which can yield faster performace.

**[Back to top](#table-of-contents)**


## File & Folder Conventions

**Folder Structure:**

This is perhaps the area with more unique opinions than any other.  File structure can take on many forms.  I prefer a feature-based file structure simply because it scales in a way that makes it easy for a team member to work in a specific area of the code with all (or most) relevant files in the same folder.

Here is a suggested feature-based file structure example for development files.

```bash
app/
    common/
        app/   # examples of general-purpose, app-specific scripts
            api.coffee
            cookies.coffee
            paths.coffee
            routes-config.coffee
        app-module.coffee   # this is the script that kicks everything off
        directives/   # examples of general-purpose directives
            some_table/
                some_table.coffee
                some_table.html
            some_widget/
                ...
        filters/
            some_filter1.coffee
            some_filter2.coffee
        utils/   # examples of general-purpose scripts (not app-specific)
            lodash.coffee
            user_auth.coffee
        utils-module.coffee
    components/
        dashboard/
            dashboard-ctrl.coffee
            dashboard.coffee
            dashboard.html
            _dashboard.scss
            directives/   # examples of specialized directives
                some_table1/
                some_table2/
        dashboard-module.coffee
        home/
            ...
        home-module.coffee
    fonts/
    images/
    index.html
    main.scss   # register .scss files from various folders with import statements
```

*Notes about this example structure:*

  - Example above shows html, scss, and coffee files.  This could just as easily include jade, haml, less, sass, styl, etc.

  - All `.scss` (Sass) files other than `main.scss` are prefixed with a `_` so they can import into the `main.scss` file.

  - I place a small module file at the same level of the folder that contains the module files (this does not apply to directives).

  - If any of the folders become too large and unwieldy (for example, `components`), it is easy enough to create a subdirectory structure.

  - In the example above, `lodash.coffee` is simply a wrapper to bring lodash into the Angular dependency injection environment and remove it from the window object.  Details explained in [this video](https://www.youtube.com/watch?v=OvBlI9KuaBk).

  - Regardless of your favorite file structure, third party libraries will sometimes require a tweak of either the file structure or the library itself.

**File Naming Convention:**

  - File names should match the name of the Angular method that lives in it.

  - Convert any camelCase Angular names to snake_case file names, i.e. `angular.directive 'someCoolWidget', ...` translates into `some_cool_widget.coffee`.

  - Don't use UPPER CASE letters in file names.

  - For controller files, add `-ctrl` to the filename.  For module files, add `-module` to the filename.  For all other files, don't add the Angular mehod to the name.

  - ***Note:*** It is common to use other delimeters besides `-`, such as `.` or `_` or even camelCasing.

**[Back to top](#table-of-contents)**


## Tips & Tricks

This section is less about style and more about hints and performance recommendations.

### Publish & Subscribe (Pub/Sub) Events

  - **$scope:** Use the `$emit` and `$broadcast` methods to trigger events to direct relationship scopes only.

    ```coffeescript
    # up the $scope
    $scope.$emit 'customEvent', data

    # down the $scope
    $scope.$broadcast 'customEvent', data
    ```

  - **$rootScope:** Use only `$emit` as an application-wide event bus and remember to unbind listeners.

    ```coffeescript
    # all $rootScope.$on listeners
    $rootScope.$emit 'customEvent', data
    ```

  - ***Hint:*** Because the `$rootScope` is never destroyed, `$rootScope.$on` listeners aren't either, unlike `$scope.$on` listeners and will always persist, so they need destroying when the relevant `$scope` fires the `$destroy` event.

    ```coffeescript
    # call the closure
    unbind = $rootScope.$on 'customEvent'
    $scope.$on '$destroy', unbind
    ```

  - For multiple `$rootScope` listeners, use an Object literal and loop each one on the `$destroy` event to unbind all automatically.

    ```coffeescript
    rootListeners =
      'customEvent1': $rootScope.$on 'customEvent1'
      'customEvent2': $rootScope.$on 'customEvent2'
      'customEvent3': $rootScope.$on 'customEvent3'
    $scope.$on '$destroy', rootListeners[unbind] for unbind in rootListeners
    ```

**[Back to top](#table-of-contents)**


### Performance

  - **One-time Binding Syntax:** Since Angular 1.3, it is possible to use one-time binding syntax `{{ ::value }}`.  Binding once removes the watcher from the scope's `$$watchers` array after the `undefined` variable becomes resolved, thus improving performance in each dirty-check.

    ```html
    # avoid
    <h1>{{ sc.title }}</h1>

    # recommended
    <h1>{{ ::sc.title }}</h1>
    ```

  - **Consider $scope.$digest:** Use `$scope.$digest` over `$scope.$apply` where sensible. Only child scopes will update.  `$scope.$apply` will call `$rootScope.$digest`, which causes the entire application `$$watchers` to dirty-check again. Using `$scope.$digest` will dirty check current and child scopes from the initiated `$scope`.

**[Back to top](#table-of-contents)**


### Angular Wrapper References

  - **$document and $window:** Use `$document` and `$window` at all times to aid testing and Angular references.

    ```coffeescript
    # avoid
    dragUpload ->
      link: $scope, $element, $attrs ->
        document.addEventListener 'click', function ->
          ...


    # recommended
    dragUpload ->
      link: $scope, $element, $attrs, $document ->
        $document.addEventListener 'click', ->
          ...
    ```

  - **$timeout and $interval:** Use `$timeout` and `$interval` over their native counterparts to keep Angular's two-way data binding up to date.

    ```coffeescript
    # avoid
    dragUpload ->
      link: $scope, $element, $attrs ->
        setTimeout ->
          ..., 1000

    # recommended
    dragUpload $timeout ->
      link: $scope, $element, $attrs ->
        $timeout ->
          ..., 1000
    ```

**[Back to top](#table-of-contents)**


## Angular Docs
For anything else, including API reference, check the [Angular documentation](//docs.angularjs.org/api).


## Contributing

Open an issue first to discuss potential changes/additions.


## License

#### (The MIT License)

Copyright (c) 2015 Bob Holben

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
