---
layout: post
title:  "Play routes import"
date:   2017-11-07 22:31:15 +0200
categories: playframefork
---

How to add import statements to routes?
---

While using Play Framework you have to describe all your endpoints in `conf/routes` file:

```play2htmlrouting
# Routes
# This file defines all application routes (Higher priority routes first)
# ~~~~

GET     /api/v1/people              controllers.PeopleController.add()
GET     /api/v1/people/:personId    controllers.PeopleController.get(personId: java.util.UUID)
POST    /api/v1/people              controllers.PeopleController.create()
```

So, our mission is to bring `java.util.UUID` into scope of `routes` file.

The solution is to edit a bit out `build.sbt` file: 

```sbtshell
import play.sbt.routes.RoutesKeys

RoutesKeys.routesImport += "java.util.UUID"
```

As you may know, `routes` file is compiled by Play Framework sbt plugin into byte code and looks similar to the following:


```scala
import _root_.java.util.UUID

// @LINE:5
private[this] lazy val com_github_abrasha_v1_controller_PeopleController_route0_route = Route("POST",
    PathPattern(List(StaticPart(this.prefix), StaticPart(this.defaultPrefix), StaticPart("api/v1/people")))
  )
  
private[this] lazy val com_github_abrasha_v1_controller_PeopleController_route0_route_invoker = createInvoker(
    PeopleController_3.route,
    play.api.routing.HandlerDef(this.getClass.getClassLoader,
    "people",
    "com.github.abrasha.web.v1.controller.PeopleController",
    "people",
    Nil,
    "POST",
    this.prefix + """api/v1/people""",
    """""",
    Seq()
  )
)
```

Yeah, a bit horrible, but really important thing is that there is a desired `import` statement:
```scala
import _root_.java.util.UUID
```

It means that we can simplify a bit our `routes` file:
```play2htmlrouting
GET     /api/v1/people              controllers.PeopleController.add()
GET     /api/v1/people/:personId    controllers.PeopleController.get(personId: UUID)
POST    /api/v1/people              controllers.PeopleController.create()
```