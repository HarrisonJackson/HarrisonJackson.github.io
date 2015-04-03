---
layout: post
title:  "Prototyping with Apiary and Angularjs"
date:   2015-03-31 00:00:00
categories: frontend
---

The best lean development teams can execute quickly on an idea, and inorder to do that they need to be able to build things fast and cheap. To do that, you gotta lean heavily on existing tools, frameworks, and even a little smoke and mirrors.

This tutorial, adapted from a lecture given in the SMU Spring 2015 GUI class, will cover rapid prototyping using Apiary and Angularjs.

[https://github.com/smu-gui-spring-2015/angular-apiary-inclass](https://github.com/smu-gui-spring-2015/angular-apiary-inclass)

[DEMO](http://smu-gui-spring-2015.github.io/angular-apiary-inclass/dist/#/)

###Apiary

Apiary is a tool for mocking up APIs. Using their extended markdown syntax you can quickly spec out a REST API and publish it to their `Cowboy` server that will respond to http requests - simulating the API calls and even returning whatever data you've dumped into the fake endpoints. I've used it as a specing tool that frontend and backend teams can use to work in parallel. Spec out an API, point frontend apps at the mocked endpoints while the backend team builds out APIs to meet the spec. Then when the backend team is done just point at your own servers - as long as you built everything to spec it can be a real timesaver. 

###Angularjs

Angularjs is a frontend framework that is ideal for building client-side single-page applications. It extends HTML and provides dynamic data binding. It is better for web app development than games or other apps that require faster rendering or dom manipulation.

###Building the prototype 

Our prototype will consist of a simple note taking application. You'll be able to see existing notes in a `note list view`, view a single note in a `note detail view`, and create new notes in a `create note view`.  It's important to remind you at this time that we can't actually **create** new notes with the dummy backend. We'll setup Apiary to respond with a `201 created` response code, but the note won't actually be persisted - this is a **medium fidelity prototype** and as such, is not fully functional. 


To start off - let's setup the most basic angularjs project. This assumes you have the latest node and npm installed.

```
npm install -g grunt bower yo generator-karma generator-angular
cd ~/Development/ # or wherever you want to create it
mkdir apiary-angularjs-demo
cd apiary-angularjs-demo
yo angular notes-app

```

Go ahead and accept all the defaults - select `yes` for everything include `bootstrap` compass... etc

Here is the project so far: `tree -L 1`

```
.
├── Gruntfile.js
├── README.md
├── app
├── bower.json
├── bower_components
├── node_modules
├── package.json
└── test

```

Take a look in the app directory `tree app`

```
app
├── 404.html
├── favicon.ico
├── images
│   └── yeoman.png
├── index.html
├── robots.txt
├── scripts
│   ├── app.js
│   └── controllers
│       ├── about.js
│       └── main.js
├── styles
│   └── main.scss
└── views
    ├── about.html
    └── main.html

```

You can run the app at this point to see the default site.

```
grunt
grunt serve # will launch your browser with the app so far

```
![Default site](https://zapier.cachefly.net/storage/photos/03b16ee8a1ef3ab30d3461137dff1b83.png)


Using your editor of choice (mine is Sublime Text 3) start editing the app

First, let's update the nav in `index.html`

```
...
<ul class="nav navbar-nav">
  <li class="active"><a href="#/">Home</a></li>
  <li><a ng-href="#/notes">Notes</a></li>
</ul>
...
```

If you are still running `grunt serve` you should see the site reload in your browser with the new nav.

Next add a notes controller with `yo angular:route notes`

This will do a few things:

* create `app/scripts/controllers/notes.js`
* create `test/spec/controllers/notes.js`
* create `views/notes.html`
* update `app.js` to include the notes route
* include `<script src="scripts/controllers/notes.js"></script>` in `index.html`

Now when we click the Notes link in the nav, we'll see `This is the notes view.`

![notes in nav](https://zapier.cachefly.net/storage/photos/98aa48e07bfb528d65660fe8a763de4c.png)

Now setup the REST handler using an Angularjs Factory

```
$ yo angular:factory note
>   create app/scripts/services/note.js
>   create test/spec/services/note.js
```

###Register Apiary

To start, just create an account and an app. We will stick with their default demo app that has the `Notes` resource already defined.

Update the app settings so that CORS are enabled.

![settings](https://zapier.cachefly.net/storage/photos/6589af00b0e4c07f9bbaa61e8a564cc4.png)

![cors](https://zapier.cachefly.net/storage/photos/4623ed551a5aa17fcf773c0c4e83b0de.png)

Then copy the url for the notes list resource - should look something like this:

`http://private-ad0b0-demoapi24.apiary-mock.com/notes/`

###Back over to Angular

Modify the `services/note.js` factory with that url

```
angular.module('notesApp')
  .factory('note', function ($resource) {
    return $resource("http://private-ad0b0-demoapi24.apiary-mock.com/notes/:id");
  });

```

This is going to create a factory for the `note` resource and map it to the API.

**Add** `:id` to the url - this is gonna tell Angular how to modify the list view to get a single resource.

Update the notes controller to load the note model.

```
angular.module('notesApp')
  .controller('NotesCtrl', function ($scope, note) {
    note.query(function(data) {
        $scope.notes = data;
    });
  });
```

Go ahead and update the notes view to dump that object.

`views/notes.html`

```
{% raw %}
{{ notes }}
{% endraw %}
```

It should spit out something like this:

```
[
    {
        "id": 1,
        "title": "Jogging in park"
    },
    {
        "id": 2,
        "title": "Pick-up posters from post-office"
    }
]
```

Don't worry if it's not formatted - we just want to make sure the api data is propagating through the angular app.

Next, create a new route for a single note `yo angular:route note --uri=notes/:id`. This will add another entry in the app.js routes, another controller, and another view.

Update its controller like so:

```
angular.module('notesApp')
  .controller('NoteCtrl', function ($scope, note, $routeParams) {
    note.get({ id: $routeParams.id }, function(data) {
        $scope.note = data;
    });
  });

```

And its template `note.html`

```
{% raw %}
{{ note }}
{% endraw %}

```

Which will spit out 

```
{
    "id": 2,
    "title": "Pick-up posters from post-office",
    "text": "Some longer text that contains the whole note woohoo this is more content that we'd only display in the detail view"
}
```

**Note**: I added a `text` field to my apiary note resource so that there'd be extra data to show on the detail view.

So... now we have a list template and detail template - let's clean those up and we'll be well on the way to a **working** notes angular app.

`views/notes.html`

```
{% raw %}
<ul style="list-style:none;">
    <li ng-repeat="note in notes">
      <a href="/#notes/{{note.id}}"><h3>{{note.title}}</h3></a>
    </li>
</ul>
{% endraw %}
```

`views/note.html`

```
{% raw %}
<h3>{{ note.title }}</h3>
<p>{{ note.text }}</p>
{% endraw %}
```

Now, that should be looking sweet.

Next steps include saving new notes, deleting notes, and updating notes.

Take a look at the example code for the first 2 - I leave it as an exercise to the reader to finish the update functionality.

[https://github.com/smu-gui-spring-2015/angular-apiary-inclass](https://github.com/smu-gui-spring-2015/angular-apiary-inclass)


[DEMO](http://smu-gui-spring-2015.github.io/angular-apiary-inclass/dist/#/)





