### RT-MVC
#### Real Time Model/View/Controller Applications 
 
 Hey guys, thanks for coming out today.
 
 My name is Daniel Erickson, I’m a JavaScript Engineer at Yammer(soft) and a core contributor to Geddy, the original MVC framework for Node.js
 
 I’ll be talking to you today about some of the pain that you’ll deal with when trying to integrate the “old” (but still relevant) ideas of MVC frameworks with the “new” (and often necessary) realtime hotness and I’ll show you what we did with Geddy to alleviate some of these pains.

* * *

### So what’s going to win - realtime or MVC?

When starting a new project, there are a few questions that every development team needs to ask itself, one of which is:

“What, if any, framework are we going to use?”

Most of the time the answer will be highly dependent on the expertise of the members of the team. But sometimes the project’s requirements would have you step outside of your comfort zone - creating a realtime connection between a server and it’s clients is one of those situations for many developers.

So the question becomes:

“Do we continue with the MVC framework we’re used to and hack realtime into its own service? Or do we start with the realtime and hack our MVC into it’s framework?”

Really, when you take into account hiring new developers, the question becomes:

“What’s going to win, realtime or MVC?”

* * *

### Is realtime going to win?
#### Yeah!

Some people say that realtime is the new hottness, and if you’re using an MVC framework you’re doing it wrong. They’ll tell you that it doesn’t scale well, and they’ll say that if you want to do realtime correctly you’ve got to be close to the metal.

* * *

###  But MVC is here to stay

They may be right for some of those points, but MVC is here to stay. Modern MVC frameworks give teams the structure to get things done quickly and efficiently. You can hire people that already know how MVC frameworks are organized. Someone who knows Django is very likey to “get” Rails pretty quickly. 

All of that is great, but I think modern MVC frameworks have something that no other frameworks have yet: They’er easy to get started with. When a noob is about to start a project, they can get up and running quickly and easily with a framework like Rails, Django, or Geddy.

* * *

### Is MVC going to win?
#### Totally

When you’re trying to add a realtime stack to your application, things get complicated quickly. You’re already running this Rails app on this one server, now you’ve got this weird Event Machine up and running on this other server and you’ve got a message queue system in there somewhere and somehow that event machine server is hooked up to something called a websocket and the client is talking to it somehow and ...

Sounds pretty messy right?

* * *

### Realtime is also here to stay though

Whoa, it sounds like I’m telling you not to bother with realtime right?

Nope, not at all:

Realtime is great for your users! It allows you to responsively show your users the content that they want to see as soon as it’s available to your app. When one user adds and item here, all other connected users can see it. When it works, it looks like magic.

* * *

### So wait, who’s going to win then?

There has to be a winner right? We like things to win.

* * *

### Both!

Realtime and MVC actually mix pretty well.

You’ve already got all your models and templates defined on your server, why not use them on the front end too?

Whenever a model instance is created, changed, or deleted on the server, let all your connected users know about it.

Keeping all this logic in your model is actually a really nice way to handle realtime events.

* * *

### So let me walk you through how we solved this with Geddy.
#### Wait, whats Geddy?

It’s the original MVC framework for Node.js, and it’s inspired by popular MVC frameworks like Rails and Django. So if you know either of those frameworks, you’ll feel right at home in Geddy.

We’ve implemented a really amazing ORM layer that can connect to Postgres, MongoDB and Riak. It handles all your model definition, validation, instance methods, and static methods. The reall cool thing is, the model layer runs on the front end as well.

Geddy also handles a bunch of different templating languages - EJS and Jade are among the more popular ones.

It’s got everything else you’d expect from a modern framework to: translations, app generation, scaffolding, and environment specific config files.

* * *

### Now it supports realtime right out of the box.

Geddy now has some infrastructure and some scaffoling code to allow you to generate realtime enabled resources. It automatically includes your models on the front end and hooks up a realtime pipe via socket.io to connect the client and the server. Geddy can do this without you having to write a single line of code.

* * *

### Lets generate an app

(switch to terminal)

If you don’t have Geddy installed already:

```bash
$ npm install -g geddy
```

Generate an app with the `rt` option. This will tell the Geddy app generator to include the socket.io code that you’ll need for your app. If you have an app already and you’d like to add realtime support to it, you’ll want to take a look at the code that’s generated here and copy the relevant parts into your existing app.

```
$ geddy app -rt demo
```

Next, cd into your app:

```bash
$ cd demo
```

Once you’re there, you’ll want to scaffold a new resource, again with the `rt` option. For our demo purposes I’ll just create a `thing` with a `title` and a `description` property.

```bash
$ geddy scaffold -rt thing title description
```

Now all you have to do is start it up.

```bash
$ geddy
```

* * *

### Now to test it out

Lets open up localhost:4000/things in two browser windows. For the max effect I’ll put one window on one side of the screen and the other on the other side.

In one of the windows we’ll go ahead and create a new `thing`. I’ll just use some dummy data here.

When I click add, we should see that the test thing has been added to the index view here, without having to refresh the page, and without me having to write any code.

This works for updates as well.

It also works for removals.

Update and remove events are also passed to the show views by default, that means you can show changes in real time.

* * *

### So what just happened?

We generated an app with the realtime code enabled using `geddy app -rt demo`. This created your Main controller, your default routes, and all of your folder structure.

We scaffolded a resource and made sure it included the realtime code with `geddy scaffold -rt thing title description`. This created a thing model for us, all the necessary restful routes needed to interact with our things, a controller and some default CRUD actions, and some views with some code in them that connects to the server’s realtime pipe.

* * *

### How does it work?

#### on the server side

When Geddy generated the app for us it flipped a bit in the app’s config file to tell the app that it should enable Geddy’s built in realtime code.

Every time your server starts up, it copies your model code into a public js file so that it can be used on the front end.

Geddy then sets up lifecycle event listeners for every model that you’ve defined - events like save, update and remove get proxied down to all connected clients.

#### on the client side

The client registers lifecycle event listeners to all model related events.

It then proxies these events to the representation of the models on the front end.

The scaffolding adds some boilerplate code to help get you started with using these events on the front end.

* * *

### That’s pretty cool
#### But is it easy to change?

You might be tempted to think that this is just demo candy, it isn’t really. You’ve got access to all the code that was generated for you, it’s pretty easy to see how this stuff works.

Right now it just listens for a ‘save’ event and adds a new thing to the list - if I wanted to increment a count somewhere, or save it to a local repository, it would pretty trivial to add in.

On the server side, you can emit any events that you’d like, you can then listen to those events using the `geddy.io.on()` function.

I think the coolest thing about this though is that you can use the same code for your models on the front end as you do on the backend. All of your validations only have to be written once, and they work in both places.

* * *

### So what’s next on the roadmap?

You can use all the stuff that I showed you right now, but we’ve been thinking of some ways to make this a little better:

- Create an AJAX adapter for the front-end ORM
- Real time queries
- Per instance real time events
- A better way to share templates

* * *

### Conclusion

Its not impossible to bridge the gap between modern MVC frameworks and these new realtime services. It’s possible to get the best of both worlds:

- Structured code on both the server and the client
- code reuse when it makes sense
- model related lifecycle events on both sides of http

* * *

### Any questions?