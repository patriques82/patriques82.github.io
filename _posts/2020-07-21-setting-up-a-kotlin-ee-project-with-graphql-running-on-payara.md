---
layout:     post
title:      Setting up a KotlinEE project with graphql running on Payara
date:       2020-07-21 11:13:00
author:     Patrik Nygren
summary:    See how to use Kotlin, Jakarta EE and GraphQL together for a modern
webstack
categories: kotlin, jakartaee, graphql, payara
thumbnail:  cogs
tags:
 - kotlin
 - jakartaee
 - graphql
 - payara
---

We will simply setup a JakartaEE project to use Kotlin first. Then we will jump
into how to use GraphQL instead of the default REST endpoints. But first we need
to write a build.gradle file.

##### Plugins

<div class="kotlin-code" theme="darcula" data-highlight-only>
plugins {
    id 'org.jetbrains.kotlin.jvm' version '1.3.72'
    id 'war'
}
</div>

We use the more modern plugins DSL instead of the legacy buildscript block to use external binary
plugins. We use kotlin.jvm plugin to enable tasks to compile Kotlin to JVM byte
code, and the war plugin to enable the war task, which we will enhance later.

##### Dependencies
 
<div class="kotlin-code" theme="darcula" data-highlight-only>
repositories {
    jcenter()
}

dependencies {
    // Align versions of all Kotlin components
    implementation platform('org.jetbrains.kotlin:kotlin-bom')
    implementation 'org.jetbrains.kotlin:kotlin-stdlib-jdk8'
    compileOnly 'javax:javaee-api:8.0'
}
</div>

We use the kotlin-bom dependency to align the kotlin SDK libraries to the same
version for compatability, and the javaee-api to use the Java EE stuff. Now all
we need is some project structure.

```
graphqldemo
├── build.gradle
└── src
    ├── main
    │   ├── kotlin
    │   │   └── graphqldemo
    │   │       ├── PersonResource.kt
    │   │       └── Person.kt
    │   ├── resources
    │   └── webapp
    │       └── WEB-INF
    │           └── beans.xml
    └── test
        ├── kotlin
        └── resources
```

Now all we need is to make this into a gradle project. 

```
gradle init
```

Next up is GraphQL.

<div class="kotlin-code" theme="darcula" data-highlight-only>
dependencies {
    ...
    implementation 'io.smallrye:smallrye-graphql-servlet:1.0.1'
}
</div>

##### Person.kt

<div class="kotlin-code" theme="darcula" data-highlight-only>
package graphqldemo

data class Person(val name: String, val surname: String, var id: Int?, var title: String?)
</div>

##### PersonResource.kt

<div class="kotlin-code" theme="darcula" data-highlight-only>
package graphqldemo

import org.eclipse.microprofile.graphql.Description
import org.eclipse.microprofile.graphql.GraphQLApi
import org.eclipse.microprofile.graphql.Name
import org.eclipse.microprofile.graphql.Query

@GraphQLApi
class ProfileGraphQLApi {

    @Query
    @Description("Get a person using the person's Id")
    fun getPerson(@Name("personId") personId: Int): Person {
        return Person("Patrik", "Nygren", 1, "worker")
    }

}
</div>

Thats it, all we need to do now is deploy this to Payara. You can download the
application server from [Payara](https://www.payara.fish/downloads/payara-platform-community-edition). Unzip the server to the location that suits you (I installed it in my home folder). `cd` into the payara folder and start the server 

```
./bin/asadmin start-domain 
```

Now run war task from the root of the project 

```
./gradlew war 
```

and copy the war file to the autodeploy folder in payara

```
graphqldemo
└── build
    └── libs
        └── graphqldemo.war
```

```
payara5
└── glassfish
    └── domains
        └── domain1
            └── autodeploy
                └── graphqldemo.war
```

Now we should have a running GraphQL endpoint running on top Payara!

```
curl localhost:8080/graphqldemo/schema.graphql
```
