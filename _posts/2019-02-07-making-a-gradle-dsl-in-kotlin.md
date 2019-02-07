---
layout:     post
title:      Making a gradle dsl in Kotlin
date:       2019-02-07 18:15:00
author:     Patrik Nygren
summary:    See that the magic features of Kotlin that makes creating a dsl are not so magic at all.
categories: jekyll
thumbnail: cogs
tags:
 - kotlin
 - dsl
---

**Note** - This article is a derivative of ["See pixyll in action"][1], taken from the lovely jekyll theme [pixyll][4].

All links are easy to [locate and discern](#), yet don't detract from the harmony
of a paragraph. The _same_ goes for italics and __bold__ elements. Even the the strikeout
works if <del>for some reason you need to update your post</del>. For consistency's sake,
<ins>The same goes for insertions</ins>, of course.

### Code, with syntax highlighting

Code blocks use the [peppermint][2] theme.

{% highlight kotlin %}
fun createProject(init: Project.() -> Unit): Project =
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
{% endhighlight %}

