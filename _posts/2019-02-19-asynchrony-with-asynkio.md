---
layout: post
title: Asynchrony made much more simpler with Asynkio !!
date: 2019-02-18 20:32:20 +0300
summary: Write your Network Requests and IO calls painlessly with Asynkio in Android/Kotlin
img: https://cdn-images-1.medium.com/max/2600/1*X5FJtXXFWY2yL5FqSWlLUw.png
img_alt: Asynchrony with Asynkio
categories: Kotlin Android
---

So far we all have been using many libraries to handle asynchrony efficiently in
JVM world. With the rise of **[Kotlin
Coroutines](https://kotlinlang.org/docs/reference/coroutines-overview.html)**
it has become more easy and even more efficient.

Still on android, we use lot of code to make a network request or any IO call
asynchronously. Specifically talking about the network requests, Java and so as
Kotlin are very bad at creating an http connection. Though some libraries have
achieved to make network calls elegantly, e.g.,
[Retrofit](https://square.github.io/retrofit/),
[Volley](https://developer.android.com/training/volley/),
[Okhttp](https://square.github.io/okhttp/) etc.

However while using these libraries in your codebase, I personally think that
you need to write a bit more code to make a simple connection request. Even
further to handle asynchrony, you have to rely on another asynchronous libraries
like Rx , coroutines, Executors, AsyncTask(No offence!) etc. That adds more code
into your codebase.

What if I tell you that you can achieve all the asynchrony with multiple network
requests or a database queries or an IO file operations in just two to three
lines of code ?

> [Asynkio](https://curiousnikhil.github.io/AsynKio/#/): A new library to write
> asynchronous code painlessly

A…yes you can do that with
**[Asynkio](https://curiousnikhil.github.io/AsynKio/#/)**.
**[Asynkio](https://curiousnikhil.github.io/AsynKio/#/)** is a library based
on **kotlin coroutines** which helps you in writing all your network and IO calls
painlessly in an asynchronous fashion.
**[Asynkio](https://curiousnikhil.github.io/AsynKio/#/)** is pretty much
inspired by Python’s [asyncio](https://docs.python.org/3/library/asyncio.html)
library which helps to write **concurrent** code using the `async/await` syntax.

What I mean is… let me show you in action:

{% highlight kotlin %}

	async {
	    //All network requests on couroutines
	    val response = await {
	        //Get the data
	        val firstResponse = get("https://isthisawesome.com/library", params = mapOf("library" to "Asynkio"))

	        //Post the data and return post response
	        return@await post("https://youareonfire.com/library", data = mapOf("id" to firstResponse.jsonObject["id"]))
	    }
	    //Process the result on UI thread
	    if (response.statusCode == 200){
	        yoTextView.text = response.text
	    }
	}

{% endhighlight %}


Yeah, that’s it. No retrofit. No volley. No other crapy things to make a simple
http connection. Seriously, No!<br>
**[Asynkio](https://curiousnikhil.github.io/AsynKio/#/)** uses the two functions
`async` & `await`.

`async` accepts a suspending lambda and it’s ran over on a controlled
coroutine([AsynkHandler](https://github.com/CuriousNikhil/AsynKio/blob/master/asynkio/src/main/java/me/nikhilchaudhari/asynkio/core/AsynkHandler.kt)).
`async` is an extension to `Any`, `Activity` and `Fragment` scope so you can use
`async` anywhere.

`await`, which is itself a suspending function, takes a lambda that has to be
run for long time. *It can be anything like a network request, an IO call to
perform file operations of a database query, distributed task queues* etc.

As you can see the method `get(...)` is an HTTP GET request over the URL
specified as a parameter in the method. The `response` of first `get` request is
assigned to variable `firstResponse` which in turn used in a second `post(...)`
request (It’s HTTP POST) and finally response of the last `post` is returned by
the `await`. Pretty simple! <br> Returned `response` can be processed on
`MainThread` , here I populated the `TextView` with result.

`async` adds the suspending lambda in queue so that the code you’ve written is
executed sequentially. While the long running and Network/IO bound code is
handled by `await` as it accepts a suspending lambda. All this became easier by
means of **Kotlin coroutines**.

Please checkout the documentation for more reference
[Asynkio](https://curiousnikhil.github.io/AsynKio/#/)

### What else it can do ?

* Well, how about a file operation?

{% highlight kotlin %}

		async {
	  
	    filename = await {
	        longRunningFileOperation(content)
	    }
	  
		}.onError {
		  
		    Toast.makeText(context, "Oops ! it failed",Toast.LENGTH_LONG).show()
		  
		}.finally {
		    closeTheFile(filename)
		}

{% endhighlight %}

* How about all HTTP requests with specific methods with no extra overhead?

{% highlight kotlin %}

		class MainActivity:AppCompatActivity(){
		    //..
		    //...
		    override fun onCreate(savedInstanceState: Bundle?){
		        //...

		        async{
		            val result = await{ get("https://your-website.com") }
		            //process result on UI thread
		            if (result.statusCode == 200){
		                textView.text = result.text
		            }

		            // similarly you can have other methods
		            //POST
		            val result = await{ post("https://your-website.com/register",
		                data = mapOf("name" to "john doe"))
		            }

		            //DELETE
		            val result = await{ delete("https://...", ...) }

		            //PUT
		            val result = await{ put("https://...", ...) }

		            // PATCH
		            val result  = await{ patch("https://..." , ...) }

		            //HEAD
		            val result = await { head("https://your-website.com/", ...) }

		            //OPTION
		            val result = await{ option("https://...", ...) }
		        }
		    }
		}

{% endhighlight %}

* How about sending custom headers and parameters with request ?<br> You often
need to send the parameters along with url in the form of `key=value`. For
sending the parameters one can use `map` for and pass it as a params value.

{% highlight kotlin %}

		//
		// Passing parameters with URL
		//
		val payload = mapOf("token" to user.token, "lang" to "en")
		async{
		    val r = await{ get("https://your-api.com", params = payload) }
		    println(r.url)
		}


		//
		//Passing headers with URL
		
		async{
		    val r = await{ get("https://your.api", headers=mapOf("X-API-Key" to "secret")) }
		}

		// There are many times that you want to send data that is not form-encoded. 
		// If you pass in any object except for a Map, that data will be posted directly (via the toString() method).

		val data = mapOf("this" to "this")
		async {
		    val r = await { get("https://your.api", json = data }
		}
		
{% endhighlight %}


### Hmm…Okay, anything else ?

* It supports [RxKotlin](https://github.com/ReactiveX/RxKotlin) and
[RxJava](https://github.com/ReactiveX/RxJava): You can pass an `Observable`
inside `await` and it’ll wait for the `Observable` to emit a value.


{% highlight kotlin %}

		async {
		    val observable = Observable.just("Belllooo")
		    result = await { observable }
		}

{% endhighlight %}

* This is better part, You can create your own `await` implementations. Here is
example of `rxjava` and `livedata` extensions to give you idea. Just return the
result of calling `AsynkHandler.await` with your own lambda implementation. The
code inside `await` block will be run on a background thread.


{% highlight kotlin %}

		suspend fun <V> AsynkHandler.await(observable: Observable<V>): V = this.await {
		    observable.blockingFirst()
		}

		suspend fun <V> AsynkHandler.await(liveData: LiveData<V>):V? = this.await {
		    liveData.value
		}

{% endhighlight %}

### Avoid Memory leaks

Long running background code referencing any view/context may produce memory
leaks. To avoid such memory leaks, call `async.cancelAll()` when all running
coroutines referencing current object should be interrupted, like

{% highlight kotlin %}

		override fun onDestroy(){
		    //...
		    //..
		    async.cancelAll()
		}

{% endhighlight %}

*****

There is lot you can do with this library. Please refer the
[documentation](https://curiousnikhil.github.io/AsynKio/#/) for further
references :

Please note that this library is in alpha. Soon will be adding many features.
PRs and contributions are always welcome.

*****

Stay tuned for the next post. I’ll cover how to use
[Asynkio](https://curiousnikhil.github.io/AsynKio/#/) into production code in
the next post. Follow me on twitter for the updates. Thanks.

