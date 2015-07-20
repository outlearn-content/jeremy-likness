<!--
{
"name" : "top-5-angular-mistakes",
"version" : "0.0.1",
"title" : "The Top 5 Mistakes AngularJS Developers Make",
"description" : "A tool like Angular can be easily abused. Avoid common traps and pitfalls before they become a problem.",
"homepage" : "http://csharperimage.jeremylikness.com/2014/11/the-top-5-mistakes-angularjs-developers.html",
"canonicalSource" : "http://csharperimage.jeremylikness.com/2014/11/the-top-5-mistakes-angularjs-developers.html",
"freshnessDate" : 2014-11-26,
"license" : "CC BY-SA 3.0"
}
-->

<!-- @section, "title": "Introduction" -->

**Note:** This module combines together five blog posts.

Although [AngularJS](http://angularjs.org) is an extraordinarily popular framework, there is plenty of discussion and controversy over whether or not it truly adds value to projects. Having witnessed its value firsthand as I described in my recent post, [Angular from a Different Angle](http://csharperimage.jeremylikness.com/2014/10/a-different-angle-what-is-angularjs.html), I believe it can be a powerful tool when used correctly. Voltaire said, “With great power comes great responsibility.” A tool like Angular can be easily abused. This series is designed to help you avoid common traps and pitfalls before they become a problem.

The top five mistakes I see people make are:

1.  Heavy reliance on $scope (not using *controller as*) 
2.  Abusing $watch
3.  Overusing $broadcast and $emit
4.  Hacking the DOM
5.  Failing to Test

In this series I’ll cover examples of both how these items are abused and my suggested “better practice.” Let’s get started with the first mistake.

<!-- @section, "title": "Relying on $scope (Not Using *Controller As*)" -->

The canonical Angular tutorial inevitably introduces controllers and forces them to take on a dependency to [$scope](https://docs.angularjs.org/guide/scope), like this:

```html
<div ng-app="myApp">
    <div ng-controller="badCtrl">
        <input placeholder="Type your name" ng-model="name" />
        <button ng-click="greetMe()" ng-disabled="notValid()">Greet Me!</button>
    </div>

</div>
```

```javascript
(function () {
    var app = angular.module('myApp', []);

    function BadController($scope) {

        $scope.notValid = function () {
            var bad = !!$scope.name;
            return !bad || $scope.name.length < 1;
        };

        $scope.greetMe = function () {
            if ($scope.notValid()) {
                return;
            }
            alert('Hello, ' + $scope.name);
            $scope.name = '';
        };
    }


    app.controller('badCtrl', BadController);

    
})();
```

A controller in Angular is really what I would refer to as a “view model.” In the diagram below, just replace “XAML” with “HTML” to visualize the relationship.

[![main1](http://lh4.ggpht.com/-JU2Ljtfi2P0/VHW_GWilfnI/AAAAAAAABYU/c4PlgRBG4OI/main%25255B1%25255D_thumb%25255B7%25255D.png?imgmax=800 "main1")](http://lh5.ggpht.com/-cym4pKSkcV8/VHW_F7Vs7DI/AAAAAAAABYQ/GywtqBL-4Xg/s1600-h/main%25255B1%25255D%25255B7%25255D.png)

A controller in the traditional sense marshals activity between the application model and view but typically does so in a more active fashion. A view model, on the other hand, simply exposes properties and functions for data-binding. I like this approach because it allows your view models to simply exist as “state bags” – they may interact with services to grab lists or other items but in the end they just expose it. You can then bind to that data and represent it however you like.

Taking on the dependency to $scope does two things. First, it requires the controller to be aware that it is participating in data-binding when it doesn’t have to. It certainly simplifies things when you can just expose a list as a list and a selected item as a property and test that in isolation without worrying about what the glue looks like. The controller defined earlier really just exists as a proxy with the sole purpose of passing setup onto the $scope. Why take on that extra work?

Second, it complicates the testing story because now you have to ensure that a $scope is created and passed in to do something. The controller really becomes a facade to the $scope so you end up with a lot of code marshaling values between the two.

(queue [Dr. Evil](http://en.wikipedia.org/wiki/Dr._Evil) voice … “*If only there were a way for the controller to BE the $scope…”*)

Wait! There is! Using the *controller as* syntax enables you to treat and test controllers as standalone plain old JavaScript objects (POJOs). Consider this example:

```html
<div ng-controller="goodCtrl as ctrl">
    <input placeholder="Type your name" 
            ng-model="ctrl.name" />
    <button ng-click="ctrl.greetMe()" 
            ng-disabled="ctrl.notValid()">Greet Me!  </button>
</div>
```

```javascript
(function () {
    var app = angular.module('myApp', []);

    function GoodController() {
    }

    angular.extend(GoodController.prototype, {
        notValid: function () {
            var bad = !!this.name;
            return !bad || name.length < 1;
        },
        greetMe: function () {
            if (this.notValid()) {
                return;
            }
            alert('Hello, ' + this.name);
            this.name = '';
        }
    });

    app.controller('goodCtrl', GoodController);
})();
```

Although it uses [angular.extend](https://docs.angularjs.org/api/ng/function/angular.extend), the controller could just as easily have been defined via the constructor function prototype. This makes it a pure JavaScript object. It has no dependencies, and why should it? We haven’t written a controller complicated enough to warrant dependencies yet. $scope just muddies the waters.

I made a fiddle to [demonstrate the $scope vs. “controller as”](http://jsfiddle.net/jeremylikness/7w7h8ry3/) approaches side-by-side.

In fact, if you are taking a test-driven development (TDD) approach, you don’t even have to worry about Angular at first. You can create your controller as a POJO and write plenty of tests, then introduce it as a controller to the application. This provides more flexibility and less reliance on what version is running or even what the UI looks like.

An added bonus is that you can alias the controller in your code so that you either segregate it per section by name, or give it a common name like *ctrl* to make it easier to build boilerplate HTML code. This syntax is also available from your routes.

Of course, the first response I typically get when I mention not using $scope is “What about watches?” My reply is, “What about them?” Angular does a lot of watching for you, so the real question is whether you really need those watches, or if there is a better way. There may be, and that’s what I’ll cover in the next post.

<!-- @section, "title": "Abusing $watch" -->

When I posted the first article, one comment suggested that using *controller as* is fine for smaller controllers but large, complex controllers with dependencies might not work out as well. I disagree, and you’ll see why later in this series. I’ll get more into dependencies in the next post, but for now I’d like to take the concept of $scope one step further to discuss the second mistake.

There are plenty of reasons why you want to watch for changes to certain properties and respond. It is quite common on a complex form to render information or process an algorithm based on a selection. To keep the example simple I created a contrived scenario, but bear with me because I think you’ll see how it extrapolates to common, real world examples.

Let’s assume you are rendering a drop-down that enables the user to select gender. In a real world example you might have some specific algorithms to run or text to display based on the selection, but for our example we’ll simply display some different text based on the gender they choose.

```html
<div ng-app="myApp">
    <div ng-controller="badCtrl">
        <select ng-options="g as g for g in genders"
                ng-model="selectedGender"></select>
        It is a {{genderText}}!
    </div>
</div>
```

The script simply keeps track of two lists and watches for changes. If the gender changes, a property is updated to show the corresponding label.

```javascript
(function (app) {

    var genders = ['Male', 'Female'],
        labels = ['boy', 'girl'];

    function BadController($scope) {
        $scope.genders = genders;
        $scope.selectedGender = genders[0];
        $scope.$watch('selectedGender', function () {
            $scope.genderText =
                $scope.selectedGender === genders[0]
                ? labels[0] : labels[1];
        });
    }


    app.controller('badCtrl', BadController);

})(angular.module('myApp', []));
```

I’ve had to build the controller with $scope because I have to $watch for the changes and the only way to $watch is by having a reference to $scope, right? Perhaps. When we run this example the watch tree looks like this:

[![watchtree1](http://lh4.ggpht.com/-_iFIu28MOG4/VHjXni-BAUI/AAAAAAAABZE/zudSqcNW_9w/watchtree1_thumb%25255B1%25255D.png?imgmax=800 "watchtree1")](http://lh5.ggpht.com/-O42dM6M95G8/VHjXnW8DulI/AAAAAAAABZA/YhSkLiNMJSQ/s1600-h/watchtree1%25255B3%25255D.png)

For performance the majority of time is spent with the model watch, and the least amount of time on the watch for the property selectedGender that we added explicitly. That time can grow, however, and add complexity as I’ll get to in a moment. Is it even possible to “watch” for a change using *controller as*? Here is some new HTML:

```html
<div ng-app="myApp">
    <div ng-controller="goodCtrl as ctrl">
        <select ng-options="g as g for g in ctrl.genders"
                ng-model="ctrl.selectedGender"></select>
        It is a {{ctrl.genderText}}!
    </div>
</div>
```

Here is the watch tree when running the new example:

[![watchtree2](http://lh4.ggpht.com/-QogaHwZ2drk/VHjXoXzmClI/AAAAAAAABZU/eHtXMbQR3ok/watchtree2_thumb%25255B1%25255D.png?imgmax=800 "watchtree2")](http://lh6.ggpht.com/-hZpX4fhOCx4/VHjXoG45mJI/AAAAAAAABZM/Q_XG9s2lY40/s1600-h/watchtree2%25255B3%25255D.png)

As you can see, there is one less watch. Performance-wise the existing watches are about the same but we’ve completely eliminated the overhead of the explicit watch. Although it was only milliseconds on a desktop browser, if we extrapolate to a complex controller with dozens of watches you can imagine it adds up quickly and will be noticeable on a mobile device.

In this case, the combined watches on the bad controller averaged about 5.264ms of overhead, while the combined watches on the good controller averaged about 4.312ms of overhead. Doesn’t seem like much? The second approach averaged a 20% improvement for just one property. Consider controllers with multiple watches and eventually you will see tangible differences in the response time of your application. This is also testing in a browser; the overhead becomes amplified when you are running Angular on a mobile device.

Speaking of mobile: not only does this approach improve performance, but it also reduces memory overhead because AngularJS doesn’t have to keep track of as many references. So how did I achieve this? Here is the code for the second example:

```javascript
(function (app) {

    var genders = ['Male', 'Female'],

        labels = ['boy', 'girl']

    function GoodController() {
        this.genders = genders;
        this._selectedGender = genders[0];
        this.genderText = labels[0];
    }

    Object.defineProperty(GoodController.prototype,
        "selectedGender", {
        enumerable: true,
        configurable: false,
        get: function () {
            return this._selectedGender;
        },
        set: function (val) {
            if (val !== this._selectedGender) {
                this._selectedGender = val;
                this.genderText =
                    val === this.genders[0]
                    ? labels[0] : labels[1];
            }
        }
    });

    app.controller('goodCtrl', GoodController);

})(angular.module('myApp', []));
```

Instead of using a $watch I’m taking advantage of properties that were [introduced in ECMAScript 5](http://www.ecma-international.org/ecma-262/5.1/#sec-8.6.1). The property manages the selection and when the selection changes, updates the corresponding label. The reason this works is because of the way Angular handles data-binding. Angular operates on a digest loop. When the model is mutated (i.e. by the user changing a selection), Angular automatically re-evaluates other properties to determine if the UI needs to be updated. Angular is already doing the work, and when you add a $watch you are simply plugging into the loop so you can react yourself.

The problem is that Angular must now hold a reference to your watch. If the model mutates, Angular will re-evaluate the expression in your watch and call your function if it changes. Of course, because your code might mutate the model further, Angular must then re-evaluate all expressions *again* to make sure there aren’t further dependent changes. This can exponentially add to the overhead of your application.

On the other hand, using properties makes it simple. When the user mutates the model by changing the gender, Angular will automatically re-evaluate properties like the gender text to see if it needs to update the UI. Because the gender selection updated the property, Angular will recognize the change and refresh the UI. In this approach, you allow Angular to do the work instead of having to plug into the digest loop yourself and add overhead to the entire process.

There are a few more lines of code with this approach, but even that can be simplified tremendously if you use a tool like TypeScript to create the property definitions. It also enables you to build pure JavaScript objects and even test for the updates without involving Angular. That keeps your tests simple and ensures they run quickly with little overhead (i.e. “Given controller when the selected gender changes to male then the gender text should be updated to boy.”)

There is one more advantage. This approach allows Angular to handle the watches, and Angular is great about managing those watches appropriately. If you add the $watch yourself, you are now responsible for de-registering the $watch when it is no longer needed. If you don’t, it will continue to add overhead.

The full source code is available in [this jsFiddle](http://jsfiddle.net/jeremylikness/f4w620Lo/) of the two controllers running side-by-side.

Keep this technique in mind because I see another practice that adds overhead and doesn’t have to when it comes to communicating between controllers. Have you seen a model where a controller does a $watch and then uses $broadcast or $emit to notify other controllers something has changed? Once again, this approach forces you to depend on the $scope and adds another layer of complexity by relying on the messaging mechanism of $scope to communicate between controllers. In my next post, I’ll show you how to avoid this third common mistake.

<!-- @section, "title": "Overusing $broadcast and $emit" -->

When I posted the first article, one comment suggested that using *controller as* is fine for smaller controllers but large, complex controllers with dependencies might not work out as well. You’ll get to see some dependencies in this article, and hopefully a new way to think about how to manage interdependencies.

One advantage of using $scope is the variety of functions that are available like being able to $watch the model. I explained why this isn’t the best idea nor approach, and in this post will tackle another set of features. Fundamentally, the concept of events and being able to have custom events is a good one. In AngularJS, custom events are supported through the [$emit](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$emit) (bubble up) and [$broadcast](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$broadcast) (trickle down) functions to publish the event.

In Angular, scopes are hierarchical. Therefore, it is possible to communicate to child scopes or listen to children using these functions. The simple [$on](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$on) function is used to register for or subscribe to an event. Let’s take a simple example. Again, I’m using something contrived to show how it works but hopefully you can extrapolate to more complicated scenarios like master/detail lists or even pages with a lot of information that need some mechanism to auto-refresh when one area is updated.

For this example, I have three separate controllers. One is responsible for handling selection of a gender. A separate controller handles showing a related label and therefore must be aware of changes, and yet another controller counts how many times the gender is changed.

The HTML5 looks like this:

```html
<div ng-app="myApp">
    <div ng-controller="genderCtrl">
        <select ng-options="g as g for g in genders" 
                ng-model="selectedGender">        
        </select>
    </div>
    <div ng-controller="babyCtrl">
        It is a {{genderText}}!
    </div>
    <div ng-controller="watchCtrl">
        Changed {{watches}} times.
    </div>
</div>
```

The gender controller exposes the list and handles the current selection. Because other controllers depend on this selection, it watches for a change and broadcasts an event whenever it changes. Some of you may recognize this antique device we used in the past to broadcast music.

[![boombox](http://lh4.ggpht.com/-bmrlWucnb2c/VIBUpJlPFeI/AAAAAAAABaI/r6P9dMe2sAQ/boombox_thumb%25255B2%25255D.jpg?imgmax=800 "boombox")](http://lh5.ggpht.com/-u3gzE8iWfZI/VIBUolEKsRI/AAAAAAAABaA/-xTqFxEmTww/s1600-h/boombox%25255B4%25255D.jpg)

A common pattern I see implemented is to broadcast from the root scope because it’s guaranteed to reach all child scopes.

```javascript
function GenderController($scope, $rootScope) {

    $scope.genders = genders;

    $scope.selectedGender = genders[0];

    $scope.$watch('selectedGender', function () {

        $rootScope.$broadcast('genderChanged',

            $scope.selectedGender);

    });

}
```

The controller that exposes the label listens for the event and updates the text accordingly.

```javascript
function BabyController($scope) {
    $scope.genderText = labels[0];
    $scope.$on('genderChanged',
        function (evt, newGender) {

        $scope.genderText =
            newGender ===
            genders[0] ? labels[0] : labels[1];
    });
}
```

Finally, a third controller listens for the event and updates a counter every time it is fired.

```javascript
function WatchController($scope) {
    $scope.watches = 0;
    $scope.$on('genderChanged', function () {
        $scope.watches += 1;
    });
}
```

This application works ([check it out here](http://jsfiddle.net/jeremylikness/zba74rk3/2/)) but I think we can do better. Why don’t I like this approach? Here are a few reasons:

-   It creates a dependency on $scope that I’m not convinced is needed.
-   It further creates a dependency on $rootScope.
-   If I choose not to use $rootScope, then my controllers have to understand the $scope hierarchy and be able to $emit or $broadcast accordingly. Try testing that! 
-   To react to a change *also* requires a dependency on $scope.
-   I now have to understand how custom events work and use their convention correctly (notice, for example, the new value is in the second parameter, not the first).  
-   I’m now doing something the “Angular way” that I might be able to do with pure JavaScript. I’m pretty sure the closer to JavaScript I stay, the easier it will be to upgrade and migrate this code later on.

OK, so what’s the answer? One way to look at this is not as a sequence of events (i.e. user changes gender, which triggers a change, which triggers an event, which triggers a response) but instead look at the result. What really happens? The gender change really transitions the state of the model. The label state simply alternates based on the gender selection state, and the counter iterates. So, state change results in mutated model. Do I really need a *message* to communicate this?

Let me break this down a different way. First, when I think about something common across controllers, I immediately think of a *service*. Whether it is defined via a factory or service isn’t the point here (if you don’t know or understand the difference, read [Understanding Providers, Services, and Factories in Angular](http://csharperimage.jeremylikness.com/2014/01/understanding-providers-services-and.html)) but rather that there is a common state shared across controllers. So, I create a gender service:

```javascript
function GenderService() { }

angular.extend(GenderService.prototype, {
    getGenders: function () {
        return genders.slice(0);
    },
    getLabelForGender: function () {
        return this.selectedGender === genders[0] ?
            labels[0] : labels[1];
    },
    selectedGender: genders[0]
});
```

The service gives me a copy of the list of genders, allows me to get or set a selected gender and returns the label for the gender. All of the useful, common functionality is packaged in one component. Aside from the way I wired it up using Angular’s extend function, it is a pure POJO object I can test without depending on Angular.

Now I can create a controller that is also a POJO. It does rely on the gender service so it will use constructor injection. Although Angular will handle this for me in the app, I can easily create my own instance of the gender service or mock it and pass it in the constructor for testing and still not depend on Angular at all … or $scope … or $emit … or $broadcast.

```javascript
function GenderController(genderService) {
    this.genders = genderService.getGenders();
    this.genderService = genderService;
}

Object.defineProperty(GenderController.prototype,
    'selectedGender', {
        enumerable: true,
        configurable: false,
        get: function () {
            return this.genderService.selectedGender;
        },
        set: function (val) {
            this.genderService.selectedGender = val;
        }
    });
```

Notice our “controller” is really just an object that exposes the list of genders and has a property that proxies the selected gender through to the gender service. I could actually have just exposed the service and bound to it directly, but that in my opinion is too much information. My UI should just be concerned with the list and selection, not the underlying implementation of how it is managed. The end result now is that you can select a gender and the service will hold the selection. Notice it is not sending any messages, so how does the label controller handle changes? Take a look:

```javascript
function BabyController(genderService) {
    this.genderService = genderService;
}

Object.defineProperty(BabyController.prototype,
    'genderText', {
        enumerable: true,
        configurable: false,
        get: function () {
            return this.genderService.getLabelForGender();
        }
    });
```

See how simple that is? I can write a test that creates three POJOs (the service and controllers), mimics updating the selection on the selection controller and verify the gender label changes on the label controller. My UI simply binds to the property it is interested in (the genderText property) and doesn’t get muddied with any presentation or business logic going on “behind the scenes.” When the label property is referenced, it asks the service what the label should be and always returns the correct value based on the current selection.

By now you’ve probably guessed what the watcher controller looks like …

```javascript
function WatchController(genderService) {

    this.genderService = genderService;

    this._watches = 0;

    this._lastSelection = genderService.selectedGender;

}

Object.defineProperty(

    WatchController.prototype,

    'watches', {

        enumerable: true,

        configurable: false,

        get: function () {

            if (this.genderService.selectedGender !==

                this._lastSelection) {

                this._watches += 1;

                this._lastSelection =

                    this.genderService.selectedGender;

            }

            return this._watches;

        }

    });
```

It simply keeps track of watches internally, and whenever they are accessed externally checks to see if the selection changed. If you don’t like this type of mutation in the property getter, you can expose it as a method and data-bind to the result of the method call instead.

Now I have four POJOs with no Angular dependencies. I can test them to my heart’s content, I don’t need to spin up a $scope and I don’t even care how $emit or $broadcast are implemented. Take a look for yourself at the [working example with no $emit or $broadcast](http://jsfiddle.net/jeremylikness/zba74rk3/).

There is one thing to call out. Some of you may recognize that this approach (specifically for the watch controller) *does* depend on Angular *indirectly*. The implementation of the watcher is based on accessing the watches count. Independent of Angular, you could manipulate the selection several times and the watcher would miss it unless you polled it each time. I know in Angular it will always be refreshed due to the digest cycle so in the Angular context it will work fine, but if I wanted something more “independent” I’d need to expose an event from the gender service and register from the watcher service to update the count even when it isn’t polled. In that case I’d probably implement my own [event aggregator](http://csharperimage.jeremylikness.com/2012/11/building-javascript-event-aggregator.html) that doesn’t rely on $scope or $scope hierarchy to function.

The reason this works without watches or messages, by the way, comes back to the way the Angular $digest cycle works. When you mutate the model by changing the gender selection, Angular automatically checks the values of the other properties that are data-bound. This, of  course, calls through to the getter that was implemented and results in the correct value being returned and refreshed to the UI. Once again, we avoid extra watches.

Because watches and event handlers can mutate the model, Angular actually has to go back and revaluate expressions each time a watch function or event handler is called to make sure there are no further updates. In this model, the model itself is already refreshed so there is only one pass needed (you’ll see multiple passes because there are multiple bindings, but no extra passes due to watches).

This approach reduces dependencies and therefore the complexity of the code, makes it easier to test, and provides a performance benefit to boost. It really just involves thinking about things in terms of relationships and state rather than the idea of a “process” that “pushes” things. Let Angular handle all of that lifting and keep your design simple and straightforward. In a later post I’ll reference a more complex project that shows a fully working app using this principle with no dependencies on $scope inside controllers.

Although I’ve addressed some common scenarios related to data-binding, invariably you will run into situations that require you to manipulate the DOM. It may be something simple like refreshing the page title based on a context change, or something more complex like rendering SVG or setting the focus on a control based on a validation error. That often leads to the fourth mistake I see developers make, which is hacking the DOM directly, often from the controller itself!

In the next post I’ll share with you how to handle that using AngularJS.

<!-- @section, "title": "Hacking the DOM" -->

In the previous posts I’ve covered some nuances around controllers and how they communicate with each other and expose information for data-binding. In this post I’ll elaborate on the importance of data-binding and share why it’s important to avoid hacking the DOM when writing an Angular application.

In the end, web applications are about the DOM. You may be surprised to learn one of the most viewed posts of all time on this blog is a short one about a simple hack for IE 6.0 that I posted almost five years ago. I’m also not too shy to admit I really didn’t understand JavaScript at the time as evidenced by the “solution” (it worked, but wasn’t necessarily the best way to approach it) and considered it an inconvenience that I simply had to work around so web applications would work.

In fact, shortly after that post I did a proof of concept for an emerging technology (at the time) called Silverlight and ended up converting to it. To make a short story even shorter, there are many reasons why Silverlight made sense at the time and although it is no longer the main technology I use, I do consider Angular to be the modern HTML5 answer to Silverlight and the MVVM pattern.

The two biggest benefits I feel data-binding provides are:

1. A declarative UI that encourages testability
2. A designer/developer workflow

The second is really the result of the first. Let’s take a step back. What do I mean by declarative UI? Consider for a moment this code:

```javascript
var value = window.document.forms["myForm"]["name"].value;
if (value === null || value == '') {
    alert('Name is required.');
}
```

Now take a look at this solution instead (note the “required” attribute:

```html
<input id="name" name="name" required placeholder="Enter your name"/>
```

Answer this: which one is easier to understand, even by a non-developer? And which one scales better – in other words, when there are more fields that are required, which solution is easier to apply to the new fields as they are introduced?

The declarative approach provides building blocks that can be placed in mark-up. The imperative approach allows logical code that requires a deeper understanding. Imperative code may enable more complex manipulation and make sense for business logic, but in my opinion you should declare as much of your UI as possible to make it designer-friendly. That’s the second point: by having a nice separation between the UI and your more complex back-end logic, your designer can literally work on the UI and manipulate it in various ways independent of your programming.

Fortunately, Angular provides a solution that allows you to encapsulate imperative logic in a declarative *directive*. Before I explore that further, however, I need to address the “traditional” method for imperatively interacting with the DOM: jQuery.

**The jQuery Factor**

I’ve been in code bases with controllers that look something like this:

```javascript
function Controller($scope) {
    $scope.name = '';
    $scope.nameError = false;
    $scope.$watch('name', function() {
        if ($scope.name === null || $scope.name == '') {
            $scope.nameError = true;
            $('#name').focus();
        }
    });
}
```

Although this may work, my next post is going to cover testing and I think anyone would be challenged to test this piece of code without spinning up a browser page that has an input field with an identifier of “name.” That’s a lot of dependencies and tests are supposed to be easy to set up and run!

Now before I continue, let me make it clear I am a huge fan of jQuery. In fact, Angular will automatically fall through to use jQuery when it is present. When it is not, Angular provides it’s own lightweight version called [jqLite](https://docs.angularjs.org/api/ng/function/angular.element). Just to be clear, however, I think the biggest benefit of jQuery is this:

*jQuery is a tool for normalizing the DOM.*

What do I mean by this? Just take a look at this [comparison of web browsers](http://en.wikipedia.org/wiki/Comparison_of_web_browsers) and you’ll find things aren’t as standard as they may seem. Instead of having a lot of logic to detect which browser you are running in and use the appropriate APIs, jQuery normalizes this for you. You get a consistent interface to interact with the DOM and let jQuery worry about the implementation nuances across browsers.

In fact, people often ask me if Angular means “no jQuery.” Although Angular can greatly reduce the amount of jQuery used in an app, sometimes jQuery is still the right answer when heavy DOM manipulation is required. I just want to make sure it happens in the right place. Getting back to Angular, I have three very simple rules when it comes to interacting with the DOM, whether I’m using jQuery or jqLite or anything else. The rules are:

1. Any imperative DOM manipulation must happen inside a directive
2. Use services to mediate between controllers and directives or directives and other directives
3. Services never depend on directives

That’s it. Even though this is older code, if you look at my [6502 emulator](http://apps.jeremylikness.com/t6502/) written in TypeScript and Angular, you’ll find a graphics display and a console (to see it in action, click Load to load a source, Compile to build it, and Run to execute it). The main CPU, however, never references the DOM and is ignorant of how it renders. In fact, whenever a byte is set in an address range that represents the display, [this code](http://t6502.codeplex.com/SourceControl/latest#Source/T6502/Scripts/app/emulator/cpu.js) is executed:

```javascript
Cpu.prototype.poke = function (address, value) {
    this.memory[address & Constants.Memory.Max] =
        value & Constants.Memory.ByteMask;
    if (address >= Constants.Display.DisplayStart &&
        address <= Constants.Display.DisplayStart +
        Constants.Display.Max) {
        this.displayService.draw(address, value);
    }
};
```

Notice it simply passes the information to the service. The service is also testable in isolation because there is no dependency directly on a directive or the DOM. You can view the source [here](http://t6502.codeplex.com/SourceControl/latest#Source/T6502/Scripts/app/services/displayService.js). So what happens? The directive takes a dependency on the service, hooks into the callback and renders the pixels using rectangles in SVG as you can see [here](http://t6502.codeplex.com/SourceControl/latest#Source/T6502/Scripts/app/directives/display.js).

There are several advantages to this approach. One is testing that I’ll cover in the next post. Another is stability. The more you rely on specific ids or specific types of DOM elements, the less stable your code is. For example, let’s assume I decided that SVG was the wrong way to render the display and wanted to use WebGL or something different. In this example, the *only* code I need to change is inside of the display directive. The service and application remain the same. I call this “refactoring containment” because the clean separation is like a bulkhead keeping you from having to change massive portions of the codebase or regression test your entire app just because you are tweaking the UI.

Here is a conceptual view of how your application might interact with the DOM – note the key interactions are either directly via data-binding or indirectly through the service/directive chain with jQuery thrown in where it makes sense to normalize the DOM.

[![DOMmanipulation](http://lh6.ggpht.com/-zk1dgLmFi10/VIxXUgKTOoI/AAAAAAAABa0/pE7SjqIbr4w/DOMmanipulation_thumb%25255B2%25255D.jpg?imgmax=800 "DOMmanipulation")](http://lh4.ggpht.com/-4PVp5jUhPqY/VIxXUcp2sMI/AAAAAAAABaw/cIiTbIyYi4M/s1600-h/DOMmanipulation%25255B4%25255D.jpg) 

To better illustrate this I’ll share a few more practical examples. Let’s take the initial comparison between imperative and declarative. How do you set the focus on an element declaratively? One approach would be to consider the focus a “state.” When a field is in an invalid state, for example, you’ll want to set the focus so the user can easily correct it. The app should only care about the state, and the directive can take care of the focus. Here’s an idea for the directive:

```javascript
app.directive('giveFocus', ['$timeout',
        function ($timeout) {
    return {
        restrict: 'A',
        replace: false,
        scope: {
            binding: '=giveFocus'
        },
        link: function (scope, element) {
            scope.$watch('binding', function () {
                var giveFocus = !!scope.binding;
                if (giveFocus) {
                    $timeout(function () {
                        element[0].focus();
                    }, 0);
                }
            });
        }
    };
}]);
```

Place the directive on the element you want to have focus, and data-bind it to whatever property should trigger the focus. The timeout simply allows the current digest loop to finish before the focus is set.

**Note**: some of you may be lining up outside of my door waving pitchforks and throwing rotten eggs because I used $watch after writing an entire post about avoiding $watch. This is one of the cases where I believe $watch makes sense, because the [isolate scope](https://docs.angularjs.org/guide/directive#isolating-the-scope-of-a-directive) is such an intrinsic component of a directive.

Here’s an example of how you might use it:

```html
<input type="text" id="name" give-focus="ctrl.invalidName" />
```

Now for an example with a service. It is quite common for your app to need to know the current size it is running in. You can only do so much with media queries and CSS. For example, let’s say you are rendering a grid with server-size paging. How do you set the page size? A common approach is to simply fix it to a hard-coded value, but if you know the height of a row you can easily accommodate the device and resize the grid accordingly.

Here’s a size service that simply keeps track of the height and width of the window you are concerned with. Your app will watch the values to make adjustments as needed.

```javascript
function SizeService() {
    this.width = 0;
    this.height = 0;
}

app.service('sizeService', SizeService);
```

Here’s the size directive. It hooks into the resize event of the browser window and recalculates the size of the element it is bound to.

```javascript
app.directive('bindSize', ['sizeService',
        '$window',
        '$timeout',

    function (ss, w, to) {
        return {
            restrict: 'A',
            replace: false,
            link: function (scope, element) {
                function bindSize() {
                    scope.$apply(function () {
                        ss.width = element[0].clientWidth;
                        ss.height = element[0].clientHeight;
                    });
                }
                w.onresize = bindSize;
                to(bindSize, 0);
            }
        };
    }]);
```

To use it, simply place the directive on the element you want to track the size of:

```html
<div ng-app="myApp" ng-controller="ctrl as ctrl" bind-size="">
```

You can see a combined demo of the focus directive and size directive [here](http://jsfiddle.net/jeremylikness/wyvktbyy/). (If you want to support multiple elements, just pass in an identifier on the bind-size and change the service to track an indexed array of widths and heights).

Keep in mind you don’t necessarily have to use services just to mediate between controllers and directives. Sometimes the directives might use a service to communicate with each other! For the size example, it might be more practical to have another directive use the size service to set the page size on the grid, so the controller is never involved (or just queries the page size from the service, rather than trying to do any computation itself).

Don’t think you have to build these for yourself. Angular already has quite a few built-in features to help separate DOM concerns from logic concerns. For example, you can inject an abstraction of the window object using [$window](https://docs.angularjs.org/api/ng/service/$window#!). Using this approach allows you to mock and test it (i.e. $window.alert vs. the hard-coded window.alert).

For class manipulation check out my [Angular health app](http://jeremylikness.github.io/AngularHealthApp/). I use the BMI value to color code the tile based on whether the individual is in a healthy range or not. To swap the class I use the built-in [ng-class](https://docs.angularjs.org/api/ng/directive/ngClass) directive with a filter as you can see [here](https://github.com/JeremyLikness/AngularHealthApp/blob/master/index.html).

Modern applications don’t just focus on look and feel, but the entire user experience. That means motion study and animations where it makes sense for transitions. Although I risk exposing the true reasons why I am a developer and not a designer, the [Angular Tips and Tricks](http://jeremylikness.github.io/AngularTipsAndTricks/) app uses the [ngAnimate](https://docs.angularjs.org/api/ngAnimate#!) module with [animate.css](http://daneden.github.io/animate.css/) to create transitions. The most egregious examples are in the [*controller as piece*](http://jeremylikness.github.io/AngularTipsAndTricks/01-controller-as.html).

The key is that the animations are [declared in HTML](https://github.com/JeremyLikness/AngularTipsAndTricks/blob/master/01-controller-as.html) and automatically trigger based on state changes and transitions. The services and controllers in the app are completely ignorant of how the views are implemented. This allows me to test those pieces and even develop them completely independently of the designer. The designer can literally add or remove animations and tweak how those animations work without ever stepping on my toes even if we are working on the same area at the same time!

To recap, there are just three simple rules you need to follow to avoid hacking the DOM and create truly scalable, maintainable, testable, and fun to use Angular applications:

1. Any imperative DOM manipulation must happen inside a directive
2. Use se11rvices to mediate between controllers and directives or directives and other directives
3. Services never depend on directives

That’s it. I know I’ve mentioned testing several times, and I realize even today there are some shops that question its value and consider it to be overhead. Not only do I believe in the value of testing, I think it is so important that it makes my top five mistakes because too many Angular developers are missing out on one of the framework’s biggest benefits. Testing will be the topic of my next post.

<!-- @section, "title": "Failing to Test" -->

Is failing to test really a problem with Angular, or a programming philosophy in general? I think it’s a little bit of both, and in this post I’ll explain why.

If you’re not sure what testing has to do with Angular, take a look at their own [developer guide entry for unit testing](https://docs.angularjs.org/guide/unit-testing#!). I’ll just quote the following:

> Angular is written with testability in mind, but it still requires that you do the right thing.

That’s a pretty clear directive (some of you see what I did there). First, I’d like to walk through how Angular embraces testing. Then, I’ll explore why it matters and how it’s not just a philosophical decision when you’re building large Angular apps. Then I’ll wrap with some final thoughts. If that works for you, keep reading.

**Angular = Testing**

I am pretty confident with that statement. Let’s explore the facts:

-   The framework was written to be testable
-   Dependency injection is a large part of the framework and helps make unit testing easier
-   [Karma](http://karma-runner.github.io/0.12/index.html) was written specifically to make running tests easier
-   Angular provides a module called [ngMock](https://docs.angularjs.org/api/ngMock) right out of the box to make testing easier by injecting and mocking services such as [$http](https://docs.angularjs.org/api/ngMock/service/$httpBackend)
-   [Protractor](http://angular.github.io/protractor/#/). Seriously.

**So What?**

Some of you will say, “So what?” Just because you *can* do something doesn’t mean you *should*, right? Fortunately, most of my decisions in programming are pragmatic and based on real world experience, not hypothetical pontification. I’ve not only worked on Angular projects that didn’t have tests, or other projects that did have tests, but have the experience of being on large projects that started without tests and then introduced them so I have a pretty good idea of the impact.

And it’s all positive.

There are some obvious benefits of tests you’ll hear thrown around like:

-   Find defects as early as possible (haven’t you heard, bugs are more expensive to fix in production?)
-   Refactoring assistance – don’t you feel better about changing code when you know you can run some tests to figure out the side effects right away?
-   Automated regression – as your application evolves, the tests make it easier to determine how your code base is keeping up with the changes

For me, though, the real benefits of testing aren’t so obvious until you’ve experienced them firsthand.

**Requirements Refinement**

Believe it or not, a good testing strategy can improve the quality of your requirements. This is essential when you are managing a large application. In fact, one of my favorite projects involved a collaborative feedback loop between the business, testers, and coders (you know, that “agile thing”) and helped really refine requirements to something usable.

We made it simple: when defining a backlog item, there would be testable acceptance criteria. We’d work with the team to ensure that criteria really could translate to a test.

So, you might start with something like this:

> “Given a user when they enter their information then the system should calculate some fitness data.”

Not a very good requirement, is it? We could get more specific and state,

> “Given a user when they enter their information then the system should compute their basal metabolic rate.”

That’s better, but I still can’t write a test. How about a few examples? Here’s one that is specific:

> “Given a 40-year old 5 ft. 10 in. male who weighs 200 pounds when BMR is calculated then should compute 1929.”

Wow! Now *that* I can write a test for. In fact, I did [write one](http://jeremylikness.github.io/AngularHealthApp/spec/formulaBmrSpec.js) you can [execute here](http://jeremylikness.github.io/AngularHealthApp/test.html) and it looks like this:

```javascript
describe("Formula for BMR", function () {
    describe("Given a 40-year old 5 ft 10 in male who weighs 200 pounds",
        function () {
            it("should compute a BMR of 1929",
                function () {
            var actual = formulaBmr({
                isMale: true,
                height: 70,
                weight: 200,
                age: 40
            });
            expect(actual).toEqual(1929);
        });
    });
});
```

Of course I added a few other scenarios. Having  requirements with explicit acceptance criteria that translate directly to tests will completely transform how you deliver enterprise software.

**API Design**

The second impact of testing is that it directly impacts the design of your API. I’m not talking about afterthought, after-the-fact tests you write just so you can check a box that says, “I wrote tests.” I’m talking true test-driven development (TDD) that involves writing the test first, watching it fail, then writing the simplest piece of code you can to satisfy the test. You should try it sometime. It takes a lot of patience at first to change the way you approach it, but I found it creates clean, readable, maintainable code.

In the example above, for example, I originally had an interface that took several parameters for the calculation. I quickly realized it wasn’t evident looking at the call what was being passed or even what order. So I updated the API to take a parameter object instead with clearly defined labels (in hindsight I should have been even more explicit and made it heightInches and weightPounds).

That resulted in writing a service that looks like this:

```javascript
function formulaBmr(profile) {

    // Women - 655 + (4.35 x weight in pounds) + (4.7 x height in inches) - (4.7 x age in years)
    function woman(weight, height, age) {
        return Math.floor(655 + (4.35 * weight) + (4.7 * height) - (4.7 * age));     }

    // Men - 66 + (6.23 x weight in pounds) + (12.7 x height in inches) - (6.8 x age in years )
    function man(weight, height, age) {
        return Math.floor(66 + (6.23 * weight) + (12.7 * height) - (6.8 * age));     }

    return profile.isMale ? man(profile.weight, profile.height, profile.age) :
        woman(profile.weight, profile.height, profile.age);
}
```

Notice it’s a standalone, clean JavaScript object I could use in any application. It only becomes “Angularized” when I register it with dependency injection:

```javascript
(function (app) {
    app.factory('formulaBmrService',
        function () {
        return formulaBmr;
    });
})(angular.module('healthApp'));
```

In fact, following a test-driven approach almost always results in clean, modular, portable code. In many cases I find that 80% of the code isn’t tied to a specific framework but exists as domain objects that perform certain tasks, and only get pulled into a framework to resolve dependencies and participate in data-binding.

**Program Design**

Testing does more than just improve API design. When I set out to build a [6502 emulator](http://apps.jeremylikness.com/apps/t6502/), I had no idea where to begin. So, I started writing tests to [make the CPU do what I wanted](http://apps.jeremylikness.com/apps/t6502/tests.htm). The CPU wouldn’t really do much without software to make it run, and most example programs for the 6502 8-bit chip are written in assembly language that must be compiled to machine code. To make this happen I wrote a set of [compiler specs](http://apps.jeremylikness.com/apps/t6502/compilertests.htm) so I could load programs and test them in the emulator. Eventually I got the emulator up and running. To see it in action, simply load a sample program, compile, then run it.

Tests can help drive the design of your overall app, and that’s exactly how I wrote the emulator.

**Documentation**

The final not-so-obvious advantage to tests is that they document the system. If you’re not sure how “op codes” work for the 6502 chip, just take a look at the [op code tests](http://apps.jeremylikness.com/apps/t6502/opcodetests.htm). It should be clear what the various labels stand for, what their function is, and what the expected result is for various operations. Even end users can read well-written test specifications and understand how a system is supposed to work.

It doesn’t stop there! Developers can read the [source code of the specs](http://apps.jeremylikness.com/apps/t6502/Scripts/tests/opCodes/flagSpecs.js) to determine how the various APIs work. The example I linked to demonstrates the various flags that exist in the 6502 CPU and how they operate. A test can therefore drive design at multiple levels, provide documentation of requirements, and demonstrate exactly how to consume the components of the system.

**Test-Driven Development**

TDD is an approach that requires discipline, patience, and most importantly buy-in to implement if your team does not already embrace it. Even if you decide you do not want to follow a TDD approach, I strongly encourage you to make testing a part of your development lifecycle. Don’t just write tests to check a box or make a code coverage number, but instead integrate them as part of a meaningful approach to reap the benefits I’ve listed.

One technique I found is useful for adopting tests is to take a two-fold approach. The first is with requirements. Ensure requirements contain acceptance criteria that is specific enough to write tests for, and if you write no other tests, at least write the tests that will satisfy acceptance criteria.

Another way to integrate testing is through your defects resolution cycle. If a bug is entered into the system, the first step should be to write a test that fails because of the bug. You won’t believe how effective this simple approach can be if you aren’t already following it.

First, being able to duplicate the bug requires that you are able to gather enough information to reproduce it. This places the onus on your testing team to collect the information you need. It also requires intimate knowledge of the code so you know what components to write the test against that cause the failure. If you find it is difficult to write tests for the majority of bugs, you might need to take a step back and assess your architecture. The same design that makes it difficult to test could be contributing to the instability that is causing defects to surface in the first place.

Creating the test helps you get more familiar with the code that is related to the defect, and also creates a baseline for preventing future defects. Once you have a failing test, you can work to modify the code so the test passes. After it passes you can pass on the fix with confidence it was properly addressed, and slowly build a test suite that ensures the defects don’t reappear as side effects to later changes.

If you want an example of a full test-driven development cycle to build a reference application, check out my post entitled [Let’s Build an Angular App](http://csharperimage.jeremylikness.com/2014/10/lets-build-angularjs-app.html)! The deck will walk you through a series of github commits that represent an app that was written with a test-driven approach. You can see it evolve and watch how I took simple steps to layer components together that ultimately contributed to the [final result](http://jeremylikness.github.io/AngularHealthApp/).

**That’s All He Wrote!**

I saved this post for last because I believe the first 4 mistakes are actually easier to avoid when you take a test-driven approach. The Angular practices I’ve picked up over the past few years are based on hands-on experience architecting systems to scale with large teams and make it easy to integrate new features. When the team is focused on a test-driven approach, they tend to create solutions that:

-   Avoid $scope when it’s not needed because it’s easier to test properties and methods on a plain old JavaScript object (POJO)
-   Don’t require the overhead of extraneous $watch because they are architected based on states and transitions
-   Communicate between controllers without depending on hierarchy or the DOM
-   Maintain state and logic in code that can be tested without the presence of a browser or a DOM because it connects via data-binding and directives rather than explicit manipulation

As you can see, the top 5 mistakes are really all related! I hope this series helped improve your approach to providing business solutions with Angular. Let me know your thoughts and please feel free to add any of your own tips or common mistakes in the feedback bar on the right! I look forward to hearing from you and appreciate all of you who have contributed your questions, thoughts, suggestions and tips to my previous posts in this series.
