---
layout: post
hero-bg-color: "#fff"
uid: web-design
worktype: "Web Design"
categories: project
title: Asynkio - Write asynchronous network and IO calls painlessly
small_title: Asynkio
description: Write your network requests, IO calls in android with Kotlin seamlessly.
img: 'http://nikhilchaudhari.me/assets/img/asynkio.png'
link: https://curiousnikhil.github.io/AsynKio/
---

Write your network requests, IO calls in android with Kotlin seamlessly. Asynkio Inspired by python's asyncio

What I mean is..

	async {
	    //All network requests on couroutines
	    val response = await {
	        //Get the data
	        val resp = get("https://awesome.com/lib", params = mapOf("lib" to "Asynkio"))

	        //Post the data
	        return@await post("https://youareonfire.com/lib", data = mapOf("id" to resp.jsonObject["id"]))
	    }
	    //Process the result on UI thread
	    if (response.statusCode == 200){
	        yoTextView.text = response.text
	    }
	}

Yes, that's it. No retrofit. No Volley. Java/Kotlin are very bad at handling the http requests, but still Asynkio is the optimal way. No extra overhead, Seriously...No bullshit!

Another example

	async {
	    filename = await {
	        longRunningFileOperation(content)
	    }
	}.onError {
	    Toast.makeText(context, "Oops ! it failed",Toast.LENGTH_LONG).show()
	}.finally {
	    closeTheFile(filename)
	}

Want to use it? Checkout full documentation over here [Getting Started](https://curiousnikhil.github.io/AsynKio/)
