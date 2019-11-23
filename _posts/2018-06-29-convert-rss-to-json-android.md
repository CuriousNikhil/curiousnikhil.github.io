---
layout: post
title: Converting RSS feeds to JSON using my API on Android-Kotlin
date: 2018-06-28 12:00:20 +0300
summary: My free api to convert the RSS feeds to JSON and how to use it in android kotlin
img: https://nikhilchaudhari.me/assets/img/api.png
---

So, I was working on one podcast app during my internship with the RSS feed. Basically, an RSS feed is a metadata and contains actual data in specific format. The RSS is based on `XML` format and includes data inside `xml tags`. I wanted to extract the URLS, descriptions and other stuffs from that RSS. 
There are many libraries to parse the `XML` into readable data but I thought of making an **API**  which will convert the RSS response to `JSON`. Because `JSON` format is what we love ;-)

### You can check the api here : [Send-RSS-Get-JSON](https://send-rss-get-json.herokuapp.com/)

You will get the description of that API on the site also the examples with other languages.
For android you can refer to the following example:
Here, I am showing you the `Retrofit2` approach,
I am assuming that you have already added the required dependencies to use retrofit in `build.gradle` file.
If you don't know where to start then I will suggest you to take the FutureStudio's [this tutorial](https://futurestud.io/tutorials/retrofit-getting-started-and-android-client).

Our URL will be: 
` https://send-rss-get-json.herokuapp.com/convert/?u=<YOUR_URL>`

And our Result will be:
{% highlight json %}
{
	  "title": "ISS On-Orbit Status Report",
    "description": "",
    "url": "https://blogs.nasa.gov/stationreport",
    "image": "",
    "items": [
        {
            "title": "ISS Daily Summary Report – 6/26/2018",
            "description": "Predetermined Debris Avoidance Maneuver (PDAM) Status: Last night’s possible PDAM for object #99999 was not required. A second update to the object’s trajectory was received overnight and the Probability of Collision (PC) had dropped well into the green threshold. Concern level was low and PDAM planning was no longer needed. MagVector 3D: The crew relocated …",
            "link": "https://blogs.nasa.gov/stationreport/2018/06/26/iss-daily-summary-report-6262018/",
            "url": "https://blogs.nasa.gov/stationreport/2018/06/26/iss-daily-summary-report-6262018/",
            "created": 1530028819000
        },
        ...
        ...
}
{% endhighlight %}

# Data Model classes
We'll create one Model class to get the `JSON` response and model it according to result
here I am taking just `description` from response:

{% highlight kotlin %}

object Model {
    data class Result(val items: Items)
    data class Items(val description: String)
}

{% endhighlight %}

# Creating Retrofit2 Service
The following interface will define the retrofit service in your app:
Note that the result is return as Model.Result, and as a Observable, which is a `Rxjava object` that could analog as the endpoint fetcher result generator.

{% highlight kotlin %}
interface ApiService {

    @GET("convert/")
    fun getResult(@Query("u") url: String): Observable<Model.Result>

}
{% endhighlight %}


# Retrofit Builder
In Kotlin `companion object` is used to make the create function below resembler the `static` method of `Java Factory pattern`. you can add this below your `ApiService interface`
{% highlight kotlin %}
companion object {
  fun create(): ApiService {

    val retrofit = Retrofit.Builder()
          .addCallAdapterFactory(
              RxJava2CallAdapterFactory.create())
          .addConverterFactory(
              GsonConverterFactory.create())
          .baseUrl("https://send-rss-get-json.herokuapp.com/")
          .build()

    return retrofit.create(ApiService::class.java)
  }
}
{% endhighlight %}


# Get the result

Before performing actual call just add the two global variables  and then the function to call the api
{% highlight kotlin %}

val apiserve by lazy {
    ApiService.create()
}
var disposable: Disposable? = null

private fun fetchResult(url: String) {
  disposable = 
     apiserve.getResult("https://blogs.nasa.gov/stationreport/feed/")
      .subscribeOn(Schedulers.io())
      .observeOn(AndroidSchedulers.mainThread())
      .subscribe(
       { result -> showResult(result.items.description) },
       { error -> showError(error.message) }
      )
}

{% endhighlight %}

And some housekeeping stuff 
{% highlight kotlin %}

override fun onPause() {
    super.onPause()
    disposable?.dispose()
}

{% endhighlight %}
That's it.

I know I didn't dig up the topics in this post because there's a lot to talk.This post is to just describe my API.