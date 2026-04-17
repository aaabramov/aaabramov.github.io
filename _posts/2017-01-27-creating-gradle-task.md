---
layout: post
title:  "Creating gradle task"
date:   2017-01-16 22:21:15 +0200
categories: gradle
---

How to create custom task in Gradle?
---

Sometimes you need to declare custom tasks in your project.

For instance, you need to copy all `*.log` files from `files` directory to the `_logs_` folder:

~~~groovy
task copyLogs(type: Copy){
    from 'logs'     // from which directory to copy
    include '*.log' // what patterns to include
    into 'archive'  // where to copy these files
}
~~~
See: [Copy task type][Copy task docs]  
See: [Groovy strings guide][Groovy strings guide]

---

How to declare tasks chain?
---

What if you need to zip them afterwards?
To follow the declarative style of Gradle you would better create chain of tasks rather that creating one but complex.
Logically the steps are:

* Zip all log files
* Move it to the `archive` directory

So let's define corresponding zip task:

~~~groovy
task zipLogs(type: Zip) {

    // tell that we should copy the logs at first
    dependsOn 'copyLogs'

    // setting archive name using GString enhancement
    archiveName "logs-${LocalDateTime.now()}.zip" 

    // from what directory collect the files
    from 'logs'

    // filter closure
    include { file -> file.name.endsWith('log') }

    // where to put the result archive
    destinationDir file('archive')
}
~~~
Read more about GString: [GString documentation][GString docs]  
Read more about Groovy closure: [Groovy closure][Groovy closure]  

In this way can call these tasks separately but tasks are logically separated and in the same time they are chained.

In this way, you can create new task, e.g. `publishLogs` and say that it `dependsOn` zipping the files and just operate the state after `zipLogs` task.

For example:

~~~groovy
task publishLogs(dependsOn: 'zipLogs') {
    doLast {
        // publish the logs or send them via e-mail
    }
}
~~~

[Copy task docs]: https://docs.gradle.org/current/dsl/org.gradle.api.tasks.Copy.html
[GString docs]: http://docs.groovy-lang.org/latest/html/documentation/#_double_quoted_string
[Groovy closure]: http://groovy-lang.org/closures.html
[Groovy strings guide]: https://abrasha.github.io/groovy/2017/01/16/groovy-strings.html