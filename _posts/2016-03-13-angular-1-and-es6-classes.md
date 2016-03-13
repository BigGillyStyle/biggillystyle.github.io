---
title: 'Angular 1.x and ES6 Classes'
---

At work we have a reasonably large Rails app that contains an also-reasonably-large Angular 1.3 app (~40,000 lines of code).  Right now we're using the [sprockets-es6 gem](https://github.com/TannerRogalsky/sprockets-es6) so that we can start to use ES6 syntax.  Most of the existing Angular code and Jasmine tests are written in CoffeeScript, but we decided that going forward we should start to use pure JavaScript...since that's where the community is heading (in addition to embracing TypeScript in particular in the Angular 2 space).  For various reasons we're not ready to make the Angular 2 jump yet, but I wanted to start writing our services and directive controllers using ES6 classes.  That seems to be a good match to how an Angular 2 ES6 setup will be done in particular.  I did fairly extensive research for my team and presented an "article" on our wiki page outlining what I thought the benefits were to this approach.  I thought it would be worth repeating that article here:

Regarding my recent PR and the motivation/reason behind taking an ES6 class-based approach to our Angular codebase...here are some articles for reference (in ascending chronological order):

1. http://blog.thoughtram.io/angularjs/es6/2015/01/23/exploring-angular-1.3-using-es6.html
2. http://www.sitepoint.com/writing-angularjs-apps-using-es6/
3. http://www.michaelbromley.co.uk/blog/350/exploring-es6-classes-in-angularjs-1-x
4. http://essenceofcode.com/2015/08/21/awesome-angular-controllers-with-es6-6-easy-steps/
5. http://chariotsolutions.com/blog/post/hitchikers-guide-ecmascript-2015es6-typescript-angular2-beginner/
6. http://learnangular2.com/lifecycle/

In reading through each of these articles again, here is a summary of their recommendations on taking this approach:

1. The `class` can be written separate from the rest of the Angular code in which you (1) define your controller/service/etc (`angular.module('x'). controller..`) and (2) define your injected dependencies.
  1. With the emergence of the ES6 module system (which our codebase does not currently use but more-or-less does in our NodeJS repo), we can then only export/import the very code we need for runtime and testing rather than placing everything effectively at the "global scope" level.
  2. This also makes testing in isolation easier.
2. Angular 2 is written in TypeScript (a superset of ES6), and all examples and documentation provided by the Angular 2 team are currently written in TypeScript with ES6/7 to follow.  There is no intention of supporting ES5 or CoffeeScript in Angular 2.
3. Direct quote from one article: "A best practice for defining Angular components has emerged whereby named functions are favored over the anonymous callbacks which are often seen in tutorials and documentation".
4. Angular 1.x defined too many different things (providers, services, factories, etc) that have slightly different ways of defining them while often providing the same functionality.  Angular 2 is attempting to do away with this redundancy (and the corresponding confusion) by favoring components and services that can be defined simply with pure JavaScript classes...with the Angular syntax remaining separate.
5. The constructor function in an ES6 class encourages all bindable properties not passed in through the directive definition to be initialized in the constructor function itself and not littered about the controller.  When combined with the methods defined on the class this provides a more "unified vision" (and a clear picture to other developers) of the exposed elements of the controller.

You'll notice that I've also defined my directives and corresponding controllers using the `controllerAs` syntax.  This eliminates the dependency on `$scope`, which has many pitfalls (performance, coupling between components, etc).  Additionally, `$scope` is **eliminated** in Angular 2 (not deprecated...*eliminated*).

The code I've written is not actually the *end goal*, though it may be for our Angular 1.x codebase, because we will most likely face the decision whenever we transition to Angular 2 as to whether to use ES7 decorators and/or TypeScript.  At this time, though, this incremental change appears to me to be the proper step in the right direction.

In our Angular chat codebase, we actually did start out with a single `ChatController`, as it made sense in the beginning stages to keep things simple.  As more of us started working simultaneously in the same area and as `ChatController` became larger and took on too many responsibilities, we started breaking our Angular templates down into element-level directives ("components") to (1) follow SRP, (2) make it easier to reason about what a single component was doing, and (3) allow for multiple people to work on the chat codebase at once without as many merge conflicts.  I think any of us who have worked on the Angular chat codebase can talk about how we can start taking some of these principles to the other areas of our Angular application (which as of this writing stands at about 40,000 lines of code!).  It is not something that can be done all at once, but in fact is much more practical in that a little bit of change can be made at a time.  Oh...and it should be noted that standalone controllers (i.e. when you define `ng-controller` in a template and then write a controller function) are also *completely removed* from Angular 2.  For this reason and the others I've mentioned, we'll need to start moving our Angular functionality into a component structure.

## The Testing Story ##

It is worth showing what some of the benefits are to taking a pure-JS, POJO-based approach to writing using ES6 classes like I've suggested.  Here's an admittedly simple example of the spec for the `<chat-client>` directive's controller (which is the only place for the directive where any logic exists):

The Controller Class
```javascript
const ChatClientListControllerContext = (() => {
  class ChatClientListController {
    constructor(ChatClientManager) {
      this.ChatClientManager = ChatClientManager;
    }

    clients() {
      return this.ChatClientManager.getClientsByName();
    }
  }

  angular.module('liveworld').controller('ChatClientListController', ChatClientListController);

  ChatClientListController.$inject = ['ChatClientManager'];

  return ChatClientListController;
})();
```

The Test Code
```javascript
#= require application

describe 'ChatClientListController', ->
  beforeEach ->
    @clients = ['pierre', 'jacques']
    ChatClientManager =
      getClientsByName: => @clients
    @ctrl = new ChatClientListControllerContext(ChatClientManager)

  it 'returns the list of clients', ->
    expect(@ctrl.clients()).toBe(@clients)
```

Notice that I only needed to bring in our application code (where `ChatClientListController` is defined)...and didn't even need the `spec_helper`. The more-complex `ChatClientController` and corresponding specs are the same.  There's no need to call `globalSetup()` or deal with Angular injecting mocks/stubs into the class.  In the `beforeEach` function I just manually (and simply) stub out the only function of `ChatClientManager` that my controller-under-test needs (the `getClientsByName` function).  When dealing with promises and such the approach won't be as simple as this, but I still think this is a "win".
