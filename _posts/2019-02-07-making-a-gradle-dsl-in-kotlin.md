---
layout:     post
title:      Making a gradle dsl in Kotlin
date:       2019-02-07 22:04:00
author:     Patrik Nygren
summary:    See that the magic features of Kotlin that makes creating a dsl are not so magic at all.
categories: kotlin
thumbnail:  cogs
tags:
 - kotlin
 - dsl
 - gradle
---

I long have preferred [Gradle][1] to Maven, due to its flexibility. The concepts in Gradle seem to fall more naturally for me than the Maven model. I must confess I actually don´t even know Maven that well, but I could still lookup stuff and adding dependencies is no biggie. I think the main reason Gradle seems more natural to me is its syntax, I kind of prefer a internal DSL (Domain Specific Language) to a XML declared project configuration. 

With that said; one thing that still bugged me with Gradle is the lack of support from the IDE when you are creating a build script, I guess this has a lot to do with the dynamic nature of [Groovy][2], which Gradle was initially based upon. But last year Gradle released [Kotlin DSL 1.0][3] which means that you can write your Gradle scripts in Kotlin in a similar fashion like you could with Gradle, but now with full support from the IDE so that you can navigate to declarations and get errors before you run your script. This is a huge timesaver. I love anything that empowers me and lets me explore code instead of resorting to searching for answers on Google or Stackoverflow.  

But this blog post is not about the what Kotlin DSL is but more about its implementation (Gradle has [awesome documentation][4] which lets you see side-by-side how something is declared both in Groovy and in the Kotlin DSL). What I want to explore is how to create a Gradle-like DSL in Kotlin.

<div class="kotlin-code" theme="dracula" data-highlight-only>
    <pre>
        <code class="hljs language-txt">
            group = "kotlin-play"
            version = "1.0-SNAPSHOT"
            repositories {
                mavenCentral()
            }
            dependencies {
                compile(kotlin("stdlib-jdk8"))
                compile(kotlin("reflect"))
            }
            dependencies.compile(files("libs/json-lib-2.4-jdk15.jar"))
        </code>
    </pre>
</div>

This is what I want to implement. To do this I think the best way is to go top-to-bottom and simply open file, write a main function with the code and fill in the gaps.

####Main function
 
<div class="kotlin-code" theme="dracula" data-highlight-only>
    <pre>
        <code class="hljs language-txt">
            val project: Project = createProject {
                group = "kotlin-play"
                version = "1.0-SNAPSHOT"
                repositories {
                    mavenCentral()
                }
                dependencies {
                    compile(kotlin("stdlib-jdk8"))
                    compile(kotlin("reflect"))
                }
                dependencies.compile(files("libs/json-lib-2.4-jdk15.jar"))
            }
        </code>
    </pre>
</div>

The Project is the representation of the configuration. If we would like this configuration could then be feed to some system to fetch dependencies from maven central, version the project, or whatever. Following the top-down approach we next focus on the `createProject` function and the `Project` type.

<div class="kotlin-code" theme="dracula" data-highlight-only>
    <pre>
        <code class="hljs language-txt">
            fun createProject(init: Project.() -> Unit): Project = // Lambda with receiver
                Project().apply(init)

            class Project {
                var group: String = ""
                var version: String = ""
                val repositories = Repositories(mutableListOf())
                val dependencies = Dependencies(mutableListOf())

                fun repositories(init: Repositories.() -> Unit) {  // Lambda with receiver
                    repositories.init()
                }

                fun dependencies(init: Dependencies.() -> Unit) {
                    dependencies.init()
                }
            }
        </code>
    </pre>
</div>

The secret sauce is the `init` parameter passed to createProject, it´s a lambda with receiver. A lambda with a receiver allows you to call methods or set properties on an object without using a qualifier. This means that the lambda we pass to the createProject function implicitly expects to be used on a Project instance, and that is what we do when we call `Project().apply(init)`. This is why we, for instance, can call `group = "kotlin-play"` in the lambda. When the lambda is applied to the project instance this translates to `project.group = "kotlin-play"` which makes more sense. 

We now turn to the `repositories` function in Project, which will allow us to write

<div class="kotlin-code" theme="dracula" data-highlight-only>
    <pre>
        <code class="hljs language-txt">
            repositories {
                mavenCentral()
            }
        </code>
    </pre>
</div>

What we end up with is the following

<div class="kotlin-code" theme="dracula" data-highlight-only>
    <pre>
        <code class="hljs language-txt">
            open class Repository(val name: String)

            class Maven: Repository("maven")

            class Repositories(private val repositories: MutableList<Repository>) :
                Iterable<Repository> by repositories {
                fun mavenCentral() {
                    repositories += Maven()
                }
            }
        </code>
    </pre>
</div>

The dependencies function is implemented in a similar way. I hope this shows one way of implementing an internal DSL in Kotlin. What I´m curious about is how to do this in a functional style, right now my mind always bends this way; I always have an underlying object that gets to be set, I know Haskell is also well-known for being a great DSL builder, so maybe I can steal some ideas from there. 


[1]: https://gradle.org/
[2]: http://groovy-lang.org/
[3]: https://blog.gradle.org/gradle-kotlin-dsl-release-candidate
[4]: https://docs.gradle.org/current/userguide/tutorial_using_tasks.html