---
layout: post
title: A complete Backend only with Kotlin and your favourite text-editor!
date: 2018-09-08 20:32:20 +0300
summary: Creating a backend with Kotlin and Ktor framework just using text-editor with the help of Kscript
img: https://nikhilchaudhari.me/assets/img/ktor.jpg
---

What ?!! Okay, let me clarify. Let’s build a complete backend /web-app with
Kotlin and your favourite text-editor only. No, no need to install IntellJ IDEA,
No need to handle Gradle file stuff .. seriously, No! All you need is a simple
text-editor and Kotlin.<br> So… how to do that ?

For this recipe, we need two ingredients (apart from text-editor and
Kotlin).<br> I assume that you’ve installed **[Kotlin](https://kotlinlang.org)**
and you are set with Atom or Sublime or Vim.

### Installing [Kscript](https://github.com/holgerbrandl/kscript) and [KTor](https://ktor.io/)
<div align="center">

<img src="https://cdn-images-1.medium.com/max/800/1*eNOHLwmo78NiRScganL79Q.gif">

</div>

> What is [KTor](https://ktor.io/) ?<br> Ktor is a framework for building
> asynchronous servers and clients in connected systems using the powerful Kotlin
programming language.

KTor is a web-framework built with Kotlin to make powerful webapps. This is the
framework we are going to use.  Ktor is a framework to easily build connected
applications — web applications, HTTP services, mobile and browser applications.
Modern connected applications need to be asynchronous to provide the best
experience to users, and Kotlin coroutines provide awesome facilities to do it
in an easy and straightforward way.<br> If you want to read a more about Ktor
check the description **[here](https://ktor.io/quickstart/index.html)**.

> What is [Kscript](https://github.com/holgerbrandl/kscript) ?<br> Enhanced
> scripting support for [Kotlin](https://kotlinlang.org/) on \*nix-based systems.


This is the interesting part. It is a [library](https://github.com/holgerbrandl/kscript) maintained by [@HolgerBrandl](https://twitter.com/holgerbrandl) which provides enhanced and awesome scripting support for Kotlin. Even you can use this tool for data/text processing and also to run Kotlin.kt files. Watch **[this talk](https://www.youtube.com/watch?v=cOJPKhlRa8c)** for more details and what Kscript can do.


To install Kscript follow this procedure:<br> If you have the
**[sdkman](http://sdkman.io)** installed you can skip directly to step
**2.**<br> **1.** Install [sdkman](http://sdkman.io) You can follow
instructions [here](https://sdkman.io/install). or Run:

{% highlight javascript %}
	$ curl -s "https://get.sdkman.io" | bash
{% endhighlight %}

Then follow the instructions on the terminal screen to run something like this
{% highlight javascript %}
	$ source "$HOME/.sdkman/bin/sdkman-init.sh"
{% endhighlight %}

**2.** Install **Kscript** Once you have installed **sdkman**, you can run this
to install Kscript. (But prerequisite to Kscript is Kotlin and Maven, so make
sure you’ve Kotlin on your machine. Simply,  run `$ sdk install kotlin` and `$
sdk install maven` ).Now run the following command:
{% highlight bash %}
	$ sdk install kscript
{% endhighlight %}

It’ll download the Kscript and will add it to your path. 

Let me give you a quick overview of Kscript, so that you’ll get an idea about
it. To get started with Kscript and for complete reference please visit **[this
repo](https://github.com/holgerbrandl/kscript)**. The extension of file is
`.kts` or you can use `.kt`. Add this line at top of your file `#!/usr/bin/env
kscript` (point to it in the shebang line). Inside of the file you can write any
kotlin code. Check below simple `hello_world.kts` .

{%highlight java%}

    import java.util.*

    println("Belllloooo!!")
    //Rest of the code below 

{%endhighlight%}

And you can Run this using the following command. It has some compilation
overhead so it’ll take some time.

{% highlight bash%}
    $ kscript hello_world.kts
{% endhighlight %}

Please go through the [kscript repo](https://github.com/holgerbrandl/kscript) to
get more details of how to include library dependency, another file,
Kotlin-opts, JVM-args etc.

### Running a KTor Server

Now let’s look at KTor. To use KTor, we’ve this dependency <br> `compile
"io.ktor:ktor-server-netty:0.9.4"` to use it with gradle. But, as I said, we are
not maintaining any gradle file stuff at all. Here comes one of the best
features of Kscript, is that you can declare library dependencies in the same
file that you are working on. Let me show you what I mean by modifying the same
`hello_world.kts`file.

{% highlight kotlin%}

#!/usr/bin/env kscript
//Here I am specifying KTor dependency like gradle-style locators
@file:MavenRepository("bintray-ktor","https://dl.bintray.com/kotlin/ktor")
@file:MavenRepository("kotlinx","https://dl.bintray.com/kotlin/kotlinx")
@file:DependsOnMaven("io.ktor:ktor-server-netty:0.9.4")

import java.util.*
println("Belllloooo!!")
//Rest of the code below

{% endhighlight %}

Here the `@file:MavenRepository(...)` is same as specifying the `repositor{} `
in `build.gradle` file in your IntellJ IDEA. The repositories required for KTor
are maven repos. Also the the `@file:DependsOnMaven(...)` specifies the artifact
required for basic usage with Netty.

The dependencies will be downloaded and cached when you will try to run the
above file for the first time. To clear the cache you can run `kscript
--clear-cache` .

By modifying further the above file let’s see how we can start the server. I am
modifying the same hello_world.kts file again.

{% highlight kotlin%}
#!/usr/bin/env kscript
@file:MavenRepository("bintray-ktor","https://dl.bintray.com/kotlin/ktor")
@file:MavenRepository("kotlinx","https://dl.bintray.com/kotlin/kotlinx")
@file:DependsOnMaven("io.ktor:ktor-server-netty:0.9.4")

import java.util.*
import io.ktor.http.*
import io.ktor.application.*
import io.ktor.response.*
import io.ktor.routing.*
import io.ktor.server.engine.*
import io.ktor.server.netty.*

val server = embeddedServer(Netty, port = 8080) {
    routing {
        get("/") {
            call.respondText("Belloooo!!", ContentType.Text.Plain)
        }
        get("/minion") {
            call.respondText("Belloooo ! I am kevin !")
        }
    }
}
server.start(wait = true)
{% endhighlight %}

And you’re done. Just run the command `$ kscript hello_world.kts` . It’ll
download the dependencies for the first time (takes a some time for compilation
and downloading dependencies). You can ignore the warnings(something like SL4J
etc). <br> Visit the url `http://localhost:8080` on your browser. Hope you’ll
possibly see something there :p

We’ve just created a simple server with a text-editor and a kotlin. For further
completion you can check the server-backed development of KTor in **[the
documentation](http://ktor.io/servers/index.html)**. For client side
HTML-templating and requests handling check this **[documentation](http://ktor.io/clients/index.html)** of KTor. 

KTor is asynchronous server framework. I’ll be covering those topics in the next
part of this series. So, stay tuned for the next post.


Part-II is coming. Stay tuned !
