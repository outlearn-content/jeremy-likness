<!--
{
"name" : "10-reasons-learn-angularjs",
"version" : "0.0.1",
"title" : "10 Reasons Web Developers Should Learn AngularJS",
"description" : "Why Angular is so powerful and pointers to resources that will get you up to speed quickly.",
"homepage" : "http://csharperimage.jeremylikness.com/2013/09/10-reasons-web-developers-should-learn.html",
"freshnessDate" : 2014-04-24,
"license" : "CC BY-SA 3.0"
}
-->

<!-- @section -->

# Overview

There is no doubt that [AngularJS](http://angularjs.org/) – the self-proclaimed “superheroic JavaScript framework” – is gaining traction. I’ll refer to it frequently as just “Angular” in this post. I’ve had the privilege on working on an enterprise web application with a large team (almost 10 developers, soon growing to over 20) using Angular for over half of a year now. What’s even more interesting is that we started with a more traditional MVC/SPA approach using pure JavaScript and [KnockoutJS](http://knockoutjs.com/) before we switched over to using the power-packed combination of [TypeScript](http://www.typescriptlang.org/) and Angular. It’s important to note that we added comprehensive testing using [Jasmine](http://pivotal.github.io/jasmine/) but overall the team agrees the combination of technologies has increased our quality and efficiency: we are seeing far fewer bugs and delivering features far more quickly.

<!-- @section -->

## Executive Summary

If you are familiar with Angular, this post may give you some ideas to think about you hadn’t encountered before. If you know Angular and are trying to justify its adoption at your company or on your project, this post can provide you with some background information that may help. If you have no idea what Angular is, read on because I’ll share why it’s so powerful and then point you to resources that will get you up to speed, quickly.

I can only assume other organizations are seeing positive results after adopting Angular. According to [Google Trends](http://www.google.com/trends/) the popularity of AngularJS (blue) compared to KnockoutJS (red) and “Single Page Applications” (yellow) is exploding.

![angulartrends](https://raw.githubusercontent.com/outlearn-content/jeremy-likness/master/assets/angulartrends_2.jpg)

<!-- @link, "url" : "https://goo.gl/r4k22s", "text": "Check out an updated Google Trends chart comparing AngularJS, KnockoutJS, React.js, Ember.js, and Backbone.js." -->

One of the first single-track AngularJS conferences, [ng-conf](http://ng-conf.org/), sold out hundreds of tickets in just a few minutes.

This post isn’t intended to bash KnockoutJS or Ember or Backbone or any of the other popular frameworks that you may already be using and are familiar with. Instead, I’d like to focus on why I believe AngularJS is gaining so much momentum so quickly and is something anyone who works on web applications should take very seriously and at least learn more about to decide if it’s the right tool to put in your box.


<!-- @section -->

## 1. AngularJS Gives XAML Developers a Place to Go on the Web

I make this bullet a little “tongue-in-cheek” because the majority of developers using Angular probably haven’t touched XAML with a 10 foot pole. That’s OK, the reasons why XAML became popular in the Microsoft world through WPF, Silverlight, and now Windows Store app development are important to look at because they translate quite well to Angular. If you’re not familiar with XAML, it is a declarative language based on XML used to instantiate object graphs and set values. You can define various types of objects with properties and literally mark up a set that will get created. The most common types of objects to create are user interface elements such as panels and controls that create a display. XAML makes it easy to layout complex UIs that may change over time. XAML supports inheritance (properties defined as children of parents can pick up values set higher in the tree) and bubbles events similar to the HTML DOM.

Another interesting component of XAML is the support for data-binding. This allows there to exist a declared relationship between the presentation layer and your data without creating hard dependencies between components. The XAML layer understands there is a contract – i.e. “I expect a name to be published” and the imperative code simply exposes a property without any knowledge of how it will be rendered. This enables any number of testing scenarios, decouples the UI from underlying logic in a way that allows your design to be volatile without having to refactor tons of code, and enables a truly parallel workflow between designers and developers.

This may sound like lip-service but I’ve been on many projects and have seen it in action. I recall two specific examples. One was a [project with Microsoft](http://www.wintellect.com/Consulting/Projects/Project-Project-Looking-Glass) that we had to finish in around 4 months. We estimated a solid 4 months of hands-on development and a separate design team required about 4 months of design before all was said and done – they went from wireframes to comps to interactive mock-ups and motion study and other terms that make me thankful I can let the designers do that while I focus on code. Of course if we followed the traditional, sequential approach, we would have missed our deadline and waited 8 months (4 months of design followed by 4 months of coding). XAML allowed us to work in parallel, by agreeing upon an interface for a screen – “These are the elements we’ll expose.” The developers worked on grabbing the data to make those properties available and wrote all of the tests around them, and the designers took the elements and manipulated, animated, and moved them around until they reached the desired design. It all came together brilliantly in the end.

The other real world example was a pilot program with a cable company. We were building a Silverlight-based version of their interactive guide. The only problem was that they didn’t have the APIs ready yet. We were able to design the system based on a domain model that mapped what the user would experience – listings, times, etc. – then fill those domain objects with the APIs once they were defined and available. Again, it enabled a parallel workflow that greatly improved our efficiency and the flexibility of the design.

I see these same principles reflected in the Angular framework. It enables a separation of concerns that allows a true parallel workflow between various components including the markup for the UI itself and the underlying logic that fetches and processes data.

<!-- @task, "text" : "If you want a refresher on separation of concerns, check out https://en.wikipedia.org/wiki/Separation_of_concerns"-->

<!-- @section -->

## 2. AngularJS Gets Rid of Ritual and Ceremony

![egyptian-gods](https://raw.githubusercontent.com/outlearn-content/jeremy-likness/master/assets/egyptian-gods_2.jpg)
<span style="font-size: xx-small;">Picture Credit:</span> [<span style="font-size: xx-small;">Piotr Siedlecki</span>](http://www.publicdomainpictures.net/browse-author.php?a=43016)

Have you ever created a text property on a model that you want to bind to your UI? How is that done in various frameworks? In Angular, this will work without any issues and immediately reflect what you type in the span:

```html
<span style="font-family: consolas;">
<input data-ng-model=’synchronizeThis’/>
<span>{{synchronizeThis}}</span></span>
```

Of course you’ll seldom have the luxury of building an app that simple, but it illustrates how easy and straightforward data-binding can be in the Angular world. There is very little ritual or ceremony involved with standing up a model that participates in data-binding. You don’t have to derive from an existing object or explicitly declare your properties and dependencies – for the most part, you can just pass something you already have to Angular and it just works. That’s very powerful. If you’re curious how it works, Angular uses [dirty tracking](http://angular-tips.com/blog/2013/08/watch-how-the-apply-runs-a-digest/).

Although I understand some other frameworks have gotten better with this, moving away from our existing framework where we had to explicitly map everything over to an interim object to data-bind to Angular was like a breath of fresh air … things just started coming together more quickly and I felt like was duplicating less code (who wants to define a contact table, then a contact domain object on the server, then a contact JSON object that then has to be passed to a contact client-side model just to, ah, display details about a contact?)

<!-- @multipleChoice -->

Angular cuts down on ritual and ceremony because

- [ ] It does not allow JSON objects
- [ ] It does very little dirty tracking
- [X] It simplifies data-binding

Remember that the example talks about synchronizing a model with the UI.

<!-- @end -->

<!-- @section -->

## 3. AngularJS Handles Dependencies

Dependency injection is something Angular does quite well. I’ll admit I was skeptical we even needed something like that on the client, but I was used to the key scenario being the dynamic loading of modules. Oh, wait – what did you say? That’s right, with libraries like [RequireJS](http://www.requirejs.org/) you can dynamically load JavaScript if and when you need it. Where dependency injection really shines however is two scenarios: testing and Single Page Applications.

For testing, Angular allows you to divide your app into logical modules that can have dependencies on each other but are initialized separately. This lets you take a very tactical approach to your tests by bringing in only the modules you are interested in. Then, because dependencies are injected, you can take an existing service like Angular’s [$HTTP service](http://docs.angularjs.org/api/ng.$http) and swap it out with the [$httpBackend mock](http://docs.angularjs.org/api/ngMock.$httpBackend) for testing. This enables true unit testing that doesn’t rely on services to be stood up or browser UI to render, while also embracing the ability to create end-to-end tests as well.

Single Page Applications use dynamic loading to present a very “native application” feel from a web-based app. People like to shout the SPA acronym like it’s something new but we’ve been building those style apps from the days of Atlas and Ajax. It is ironic to think that Ajax today is really what drives SPA despite the fact that there is seldom any XML involved anymore as it is all JSON. What you’ll find is these apps can grow quickly with lots of dependencies on various services and modules. Angular makes it easy to organize these and grab them as needed without worrying about things like, “What namespace does it live in?” or “Did I already spin up an instance?” Instead, you just tell Angular what you need and Angular goes and gets it for you and manages the lifetime of the objects for you (so, for example, you’re not running around with 100 copies of the same simple service that fetches that contact information).

<!-- @section -->

## 4. AngularJS Allows Developers to Express UI Declaratively and Reduce Side Effects

There are many advantages to a declarative UI. I mentioned several when I discussed XAML earlier in this post, but HTML is in the same boat. Having a structured UI makes it easier to understand and manipulate. Designers who aren’t necessarily programmers can learn markup far easier than they can programming. Using jQuery you end up having to know a lot about the structure of your documents. This creates two issues: first, the result is a lot of unstable code working as “glue” that is tightly coupled to changes in the UI, and second, you end up with plenty “magic” because it’s not evident from looking at the markup just what the UI will do. In other words, you may have a lot of behaviors and animations that are wired up “behind the scenes” so it’s not apparent from looking at the form tags that any validation or transitions are taking place.

By declaring your UI and placing markup directly in HTML, you keep the presentation logic in one place and separated from the imperative logic. Once you understand the extended markup that Angular provides, code snippets like the one above make it clear where data is being bound and what it is being bound to. The addition of tools like directives and filters makes it even more clear what the intent of the UI is, but also how the information is being shaped because the shaping is done right there in the markup rather in some isolated code.

[Maintaining large systems](http://wintellect.com/html5-javascript-experts-angular-consultants) – whether large software projects or mid-sized projects with large teams – is about reducing side effects. A side effect is when you change something with unexpected or even catastrophic results. If your jQuery depends on an id to latch onto an element and a designer changes it, you lose that binding. If you are explicitly populating options in a dropdown and the designer (or the customer, or you) decides to switch to a third party component, the code breaks. A declarative UI reduces these side effects by declaring the bindings at the source, removing the need for hidden code that glues the behaviors to the UI, and allowing data-binding to decouple the dependency on the idea (i.e. “a list”) from the presentation of the idea (i.e. a dropdown vs. a bulleted list).

<!-- @multipleChoice -->

Expressing UI declaratively helps developers

- [X] Work more effectively with designers
- [ ] Create new kinds of business logic
- [ ] Manipulate the "glue" that connects the work of designers and developers


<!-- @end -->

<!-- @section -->

## 5. AngularJS Embraces ‘DD … Er, Testing

![man-1378643715O0o](https://raw.githubusercontent.com/outlearn-content/jeremy-likness/master/assets/man-1378643715o0o_2.jpg)
<span style="font-size: xx-small;">Photo credit:</span> [<span style="font-size: xx-small;">George Hodan</span>](http://www.publicdomainpictures.net/browse-author.php?a=8245)

It doesn’t matter if you embrace Test-Driven Development, Behavior-Driven Development, or any of the driven-development methodologies, Angular embraces this approach to building your application. I don’t want to use this post to get into all of the advantages and reasons why you should test (I’m actually amazed that in 2013 people still question the value) but I’ve recently taken far more of a traditional “test-first” approach and it’s helped. I believe that on our project, the introduction of Jasmine and the tests we included were responsible for reducing defects by up to 4x. Maybe it’s less (or it could be more) but there was a significant drop-off. This isn’t just because of Angular – it’s a combination of the requirements, good acceptance criteria, understanding how to write tests correctly and then having the framework to run them – but it certainly was easier to build those tests.

If you want to see what this looks like, take a look at my [6502 emulator](http://apps.jeremylikness.com/apps/t6502/) and then browse the [source code](http://t6502.codeplex.com/). Aside from some initial plumbing, the app was written with a pure test-first approach. That means when I want to add an op code, I write tests for the op code then I turn around and implement it. When I want to extend the compiler, I write a test for the desire outcome of compilation that fails, then I refactor the compiler to ensure the test passes. That approach saved me time and served to both change the way I structured and thought about the application, but also to document it – you can look at the specs yourself and understand what the code is supposed to do. The ability to mock dependencies and inject them in Angular is very important and as you can see from the example, you can test everything from UI behaviors down to your business logic.

<!-- @section -->

## 6. AngularJS Enables Massively Parallel Development.

One of the biggest issues we encountered early in the project was developers stepping on each other’s toes. Part of this is just a discipline and even with raw JavaScript you can follow patterns that make it more modular, but Angular just took it to another level. That’s not to say it completely eliminates dependencies, but it certainly makes them easier to manage. As a specific case in point, there is a massive grid in the application that is used to drive several key operations. In a traditional JavaScript application it could have been a merge nightmare to scale this across a large team. With Angular, however, it was straightforward to break down the various actions into their own services and sub-controllers that developers could independently test and code without crashing into each other as often.

Obviously for larger projects, this is key. It’s not just about the technology from the perspective of how it enables something on the client, but actually how it enables a workflow and process that empowers your company to scale the team.

<!-- @section -->

## 7, AngularJS Enables a Design <—> Development Workflow.

OK, who am I kidding, right? You can get this with HTML and CSS and other fun technologies. The reason this gets its own bullet, however, is because of just how well this works with Angular. The designer can add markup without completely breaking an application because it depends on a certain id or structure to locate an element and perform tasks. Instead, rearranging portions of code is as easy as moving elements around and the corresponding code that does the binding and filtering moves with it. Although I haven’t yet seen a savvy environment where the developers share a “design contract” with the UI/UX team, I don’t doubt it’s far off. Essentially the teams agree on the elements that will be displayed, then design goes and lays it out however they want while development wires in the [$scope](http://docs.angularjs.org/guide/scope) with their controllers and other logic, and the two pieces just come together in the end. That’s how we did it with XAML and there is nothing preventing you from doing the same with Angular.

If you’re a Microsoft developer and have worked with Blend … wouldn’t it be cool to see an IDE that understands Angular and could provide the UI to set up bindings and design-time data? The ability is there, it just needs to be built, and with the popularity I’m seeing I don’t doubt that will take long.

<!-- @section -->

## 8. AngularJS Gives Developers Controls.

One of the most common complaints I heard about moving to MVC was “what do we do with all of those controls?” The early perception was that controls don’t work/wouldn’t function in the non-ASP.NET space but web developers who use other platforms know that’s just not the case. There are a variety of ways to embrace reusable code on the web, from the concept of [jQuery plugins](http://plugins.jquery.com/) to third-party control vendors like one of my favorites, [KendoUI](http://www.kendoui.com/).

Angular enables a new scenario known as a “directive” that allows you to create new HTML elements and attributes. In the earlier example, the directive for the “data-ng-model” allowed data-binding to take place. In my emulator, I use a directive to create two new tags: a “console” tag that writes the console messages and a “display” tag that uses SVG to render the pixels for the emulator (OK, by this time if you’ve checked it out I realize it’s more like a simulator). This gives developers their controls – and more importantly, control over the controls.

Our project has evolved with literally dozens of directives and they all participate in previous points:

*   Directives are testable
*   Directives can be worked on in parallel
*   Directives enable a declarative way to extend the UI, rather than using code to wire up new constructs
*   Directives reduce ritual and ceremony
*   Directives participate in dependency injection

Remember how I mentioned the huge grid that is central to the project? We happen to use a lot of grids (as does almost every enterprise web application ever written). We use the KendoUI variant, and there are several steps you must take to initialize the grid. For our purposes, many of the configuration options are consistent across grids, so why make developers type all of the code? Instead, we enable them to drop a new element (directive), tag it with a few attributes (directives), and they are up and running.

<!-- @multipleChoice -->

Directives

- [ ] Are compared to how controls work in ASP.NET and related frameworks
- [X] Are related to other benefits mentioned in the post
- [ ] Are a tool specifically suited to large grid layouts in web apps

Note the list of benefits from directives included in this section.

<!-- @end -->

<!-- @section -->

## 9. AngularJS Helps Developers Manage State.

I hesitate to add this point because savvy and experienced web developers understand the concept of what HTTP is and how to manage their application state. It’s the “illusion” of state that was perpetuated by ASP.NET that confuses developers when they shift to MVC. I once read on a rather popular forum a self-proclaimed architect declare that MVC was an inferior approach to web design because he had to “build his own state management.” What? That just demonstrates a complete lack of understanding of how the web works. If you rely on a 15K view state for your application to work, you’re doing it wrong.

I’m referring more to client state and how you manage properties, permissions, and other common cross-cutting concerns across your app in the browser. Angular not only handles dependency injection, but it also manages the lifetime of your components for you. That means you can approach code in a very different way. Here’s a quick example to explain what I mean:

One of the portions of the application involved a complex search. It is a traditional pattern: enter your search criteria, click “search” and see a grid with the results, then click on a row to see details. The initial implementation involved two pages: first, a detailed criteria page, then a grid page with a pane that would slide in from the right to reveal the details of the currently selected row. Later in the project, this was refactored to a dialog for the search criteria that would overlay the grid itself, then a separate full screen page for the details.

In a traditional web application this would involve rewriting a bit of logic. I’d likely have some calls that would get detail information and expect to pass them on the same page to a panel for the detail, then suddenly have to refactor that to pass a detail id to a separate page and have that page make the call, etc. If you’ve developed for the web for any amount of time you’ve had to suffer through some rewrites that felt like they were a bit much for just moving things around. There are multiple pieces of “state” to manage, including the selection criteria and the identifier for the detailed record being shown.

In Angular, this was a breeze. I created a controller for the search dialog, a controller for the grid, and a controller for the detail page. A parent controller kept track of the search criteria and current detail. This meant that switching from one approach to the other really meant just reorganizing my markup. I moved the details to a new page, switched the criteria to a dialog, and the only real code I had to write was a new function to invoke the dialog when requested. All of the other logic – fetching and filtering the data and displaying it – remained the same. It was a fast refactoring. This is because my controllers weren’t concerned with how the pages were organized or flowed – they simply focused on obtaining the information and exposing it through the scope. The organization was a concern of routing and we used Angular’s routing mechanisms to “reroute” to the new approach while preserving the same controllers and logic behind the UI. Even the markup for the search criteria remained the same – it just changed from a template that was used as a full page to a template that was used within a dialog.

Of course, this type of refactoring was possible due to the fact the application is a hybrid Single Page Application (SPA).

<!-- @section -->

## 10. AngularJS Supports Single Page Applications.

In case you missed it, this point continues the last one. Single Page Applications are becoming more popular for a good reason. They fill a very specific need. More functionality is being moved to the web, and the browser is finally realizing its potential as a distributed computing node. By design, SPA applications are far more responsive (even though some of that is perception). They can provide an experience that feels almost like a native app in the web. By rendering on the client they cut down load on the server as well as reduce network traffic – instead of sending a full page of markup, you can send a payload of data and turn it into markup at the client.

In [our experience](http://wintellect.com/html5-javascript-experts-angular-consultants), large apps make more sense to build as hybrid SPA apps. By hybrid I mean instead of treating the entire application as a single page application, you divide it into logical units of work or paths (“activities”) through the system and implement each of those as a SPA. You end up with certain areas that result in a full page refresh, but the key interactions take place in a series of different SPA modules. For example, administration might be one “mini” SPA app while configuration is another. Angular provides all of the necessary infrastructure from routing (being able to take a URL and map it to dynamically loaded pages), templates, to journaling (deep linking and allowing users to use the built-in browser controls to navigate even though the pages are not refreshing) needed to stand up a functional SPA application, and is quite adept at enabling you to share all of the bits and pieces across individual areas or “mini-SPA” sections to give the user the experience of being in a single application.

<!-- @section -->

## Conclusion

Whether you already knew about Angular and just wanted to see what my points were, or if you’re getting exposed for the first time, I have an easy “next step” for you to learn more. I recorded a video that lasts just over an hour covering all of the fundamentals you need to get started with writing Angular applications today as part of a longer course about mastering Angular. Although the video is on our “on demand training” site, I have a code you can use to get free access to both the video and the rest of the courses we have on [WintellectNOW](http://www.wintellectnow.com/). Just head over to my [Mastering AngularJS video series](http://www.wintellectnow.com/Course/Detail/mastering-angularjs) and use the code **<span style="color: #ff0000;">LIKNESS-13</span>** to get your free access. Be sure to checkout everything else we can offer for [Angular consulting and training](http://wintellect.com/html5-javascript-experts-angular-consultants).
