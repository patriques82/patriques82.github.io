---
layout:     post
title:      Making a gradle dsl in Kotlin
date:       2019-02-07 18:15:00
author:     Patrik Nygren
summary:    See that the magic features of Kotlin that makes creating a dsl are not so magic at all.
categories: jekyll
thumbnail:  cogs
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

<div class="kotlin-code" theme="idea" data-highlight-only>
    <pre>
        <code class="hljs language-txt">
            "class Contact(val id: Int, var email: String) 

            fun main(args: Array&lt;String&gt;) {
                val contact = Contact(1, "mary@gmail.com")
                println(contact.id)                   
            }
            "
        </code>
    </pre>
</div>


