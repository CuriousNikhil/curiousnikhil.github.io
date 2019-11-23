---
layout: post
title: Creating a Music Streaming Server and live music playing android client
date: 2018-05-02 20:32:20 +0300
summary: An android app listening the realtime audio stream streamed by the simple streaming server created by your own.
img: "https://firebasestorage.googleapis.com/v0/b/todo-a326b.appspot.com/o/music.jpg?alt=media&token=bef54bbf-1a9c-4bee-8486-2e251e1563bd"
---

I'd an awesome experience when I was in [Smart India Hackathon Finals](https://innovate.mygov.in/sih2018/) and we'd worked on one problem of public announcement system. So, I've decided to write a blog on how to make a simple audi/music streaming server in NodeJs and a simple android app which will stream the realtime audio. By using following code you can make your own local simple music streaming server and that music can be listened on an  android client or a browser in realtime. So, let's open the oven to cook this recipe. 


### Creating a simple Music Streaming Server
You can make your own streaming server by using anything but the simplest way I've found here is to use the open source software **SWYH** ( Stream What You Hear ). You can check the official site [here](https://www.streamwhatyouhear.com/). This application is availabel only for windows and can be downloaded from [here](https://www.streamwhatyouhear.com/download/). You can also use the **[IceCast](http://icecast.org/)** as well. 

The important thing here is that we are not lifting that creepy load of maintaining a buffer and then streaming the audio chunks over the request with the help of some code. We are using these software to handle the weight for us, thus it becomes easy to get a **url** on which an audio is streamed.

Once you've installed the **SWYH** follow the steps from here [Getting-Started](https://www.streamwhatyouhear.com/getting-started/). If everything goes right you'll end up with the following sreenshot:

![](http://www.streamwhatyouhear.com/wp-content/uploads/2012/11/4.png)

Here you can check the _URL_ as some local address of the server and the route of **/stream/swyh.mp3**.
To listen the music you have to connect to the same **wifi-network** to which the machine with SWYH is connected. You can create a simple hotspot or you may connect to your own network in home. _This is must that those devices who want to listen the stream should be connected to the same network_. Now we're ready to move forward.

You can simply just paste the _URL_ shown by the SWYH to browser and Bingoo!! You can listen the music on that device as well. Of course, someone needs to play the music on the machine with SWYH installed.
Now you have your streaming server ready and we can build the android app. Don't worry if you did not understand the above thing so far, it'll get clear as we go further or you may check the _documentation_ of _SWYH_ to get clear understanding.

### Creating a NodeJs streaming server
Let's suppose that you want to make your streaming public and everyone in the world can listen the awesome playlist you have with Weekend, Linkin-Park, Queen (My favourite one: I want to break free ;-)etc. You can make a simple nodejs server to **pipe** the stream which is currently streamed by the **SWYH** software. 

To pipe the stream to some different URL we can use `request` `node module`. All you have to do is to first install `request` node module.

{% highlight javascript %}
	
    $ npm install request --save
{% endhighlight %}

Once you have installed the `request` module, you can use this in your `Express server`

{% highlight javascript %}
	
    var express = require('express');
    var path = require('path');
    var request = require('request');
    
    var app = express()
    
    app.get('/playback.mp3', function(req, res){
    
    //the request module is piping the steam over the get request coming on playback.mp3
    request.get('http://<your-swyh-url:56789>/stream/swyh.mp3').pipe(res, function(error){
       		console.log(error);
       });
    });
    
    //here the server is localhost and listening on the port 8000
    app.listen(8000,function(){
		console.log("Listening on 8000");
	})
    
{% endhighlight %}

As you can see we are piping the streaming from one _URL_ to othe _URL_ i.e from one server to other server. Here, I would like to mention that you can use any link in the `request.get(...)` function. 
As an example, you can use [this one](http://playerservices.streamtheworld.com/api/livestream-redirect/KUFXFM_SC) as well. 

{% highlight javascript %}
	
    request.get('http://playerservices.streamtheworld.com/api/livestream-redirect/KUFXFM_SC').pipe(res,function(error){
    ....
    });
    ....
{% endhighlight %}

And Bingoo!! Now you know how to make a simple Music Streaming Server !!!
Here half of the recipe gets completed.

### Baking an android client
Creating an android client app for listening the live stream audio is very easy. We are going to use the existing `MediaPlayer` class from android framework. Checkout the description [here](https://developer.android.com/reference/android/media/MediaPlayer).

It handles the audio buffering in background and audio can be listened from the speakers of android device. 
Here is the code inside of an `onCreate()` method: 

{% highlight java %}

    public class MainActivity extends AppCompatActivity {
    //....//declaring the MediaPlayer
    MediaPlayer mp;
        
    @Override
    protected void onCreate(Bundle savedInstanceState) {
    	super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //...
        //...
        String streamURL = "your-music-streaming-url-whatever";
        //Instantiating the MediaPlayer class
        
        mp = new MediaPlayer();
        
        //setting the audio stream type to Streaming music
        try{

            mp.setAudioStreamType(AudioManager.STREAM_MUSIC);
            mp.setDataSource(streamURL);
            mp.prepareAsync();

        }catch (Exception e){
            e.printStackTrace();
        }
        
        //catching the error if any
        mp.setOnErrorListener((mediaPlayer,what,extra)->{
            mediaPlayer.reset();
            return false;
        });
       
    }
{% endhighlight %}

Here, we've initialised the `MediaPlayer` and the audio stream type is set to `STREAM_MUSIC`. 
The `DataSource`is set to the `URL` we are using to steam the audio/music.

The interesting thing is the `prepareAsync()` method. This method creates an **asynchronous request** to the URL and keeps the network out of the `MainThread`. The `MediaPlayer` asynchromously calls the `GET` method of our server and in response to request the audi packets are sent. The audio packets are captured and stored in buffer. 

As the buffer is loaded enough to play some bit of music the Media player is prepared to send the sound to speakers. `SetOnPreparedListener` is used to check the above case and we can `play()` the music. The below code shows `SetOnPreparedListener` written right below the above code. 

{% highlight java %}
	
    mp.setOnPreparedListener((mediaPlayer)->{
    	//TODO:use some button to play and pause the music
        // you can use this with some button click action as well
    	if(playButton.isChecked()){
        	mp.start();
        }else{
        	mp.pause();
        }
        
    });
{% endhighlight %}

`mp.start()` call will play the sound from your speaker and you can enjoy the delightful music. Wow !!


Now, as we've made some mess while making this recipe, we need to place the things right unless we will be facing the tonnes of memory leak in android.
In you `onDestroy()` and `onStop()` method add the following code and you are good to go: 

{% highlight java %}

    //...
    @Override
    protected void onDestroy() {
        super.onDestroy();

        if (mp!=null){
            if (mp.isPlaying()){
                mp.stop();
            }
            mp.release();
            mp = null;
        }
     //...
{% endhighlight %}

>_Genieße die Musik_ 

Alright, I'm going to listen some Queen and Coldplay. Any queries? Comment section is below ↓ .
