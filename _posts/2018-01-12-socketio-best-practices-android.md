---
layout: post
title: Socket.IO best practices on Android Client
date: 2018-1-13 18:32:20 +0300
summary: How to implement the socket.Io in android, Best practices for implementing socket.io on android
img: https://nikhilchaudhari.me/assets/img/socket.jpg
categories: [Android,Socket.IO]
---

[Socket.IO](https://socket.io/ "Socket Io Project") enables real-time bidirectional event-based communication. It works on every platform, browser or device, focusing equally on reliability and speed. Socket.IO is built on top of the WebSockets API (Client side) and Node.js.

Socket.IO is not a WebSocket library with fallback options to other realtime protocols. It is a custom realtime transport protocol implementation on top of other realtime protocols. Its protocol negotiation parts cause a client supporting standard WebSocket to not be able to contact a Socket.IO server.
And a Socket.IO implementing client cannot talk to a non-Socket.IO based WebSocket or Long Polling Comet server. Therefore, Socket.IO requires using the Socket.IO libraries on both client and server side.

The official site says,

> Socket.IO enables real-time bidirectional event-based communication.

The Socket.IO enables the client and server to communicate in real time. The Socket.IO Node.Js server defines the events that to be triggered on a specific route.Those events are listened by server which are emitted by clients.Also, a server can emit an event to which a client can listen.

The Socket.IO can easily be implemented over web apps using NodeJs server as a backend. But when it comes the case of client as an Android the ?


![](https://i.imgflip.com/w1g40.jpg)

Then we have a best library created by **Naoyuki Kanezawa** ([@nkwaza](https://github.com/nkzawa)).
and one by official [Socket.IO](https://github.com/socketio "Github") for android as client of Socket.IO server. 

It depends on you what do you want to choose as your personal preference.
1. [Socket.Io Android-Java client](https://github.com/socketio/socket.io-client-java) **Or**
2. [Nkwaza Android Socket.IO client library](https://socket.io/blog/native-socket-io-and-android/)

you can use either of them. I would recommend you the **second** one because it's very handy and kinda easy to use.
You can go through the [official blog](https://socket.io/blog/native-socket-io-and-android/) on Socket.IO site by NKwaza himself.

To implement the Socket.IO android library in perfect and flawless way you should read further.
There are three steps for implementing SOcket.IO Android Client library.

1. Initialise the Socket
2. Connect the socket to the server
3. Start emitting events or listening events

Many a times when we use Socket.IO the problems generally faced are 1-Incorrect initialisation and it gives `NullPointerExecption`. 2- The proper event could not be emitted and listened as the socket is not connected.


To avoid this try the following approach.

Very first step **Add the following dependency** in your `app` level `build.gradle` file

{% highlight java %}
implementation 'com.github.nkzawa:socket.io-client:0.5.2'
{% endhighlight %}





### 1 Initialising the socket

To intialise the socket create an `<name-of your application>` class which extends `Application` class.

See the code snippet below



{% highlight java %}
		
import android.app.Application;
import com.github.nkzawa.socketio.client.IO;
import com.github.nkzawa.socketio.client.Socket;
import java.net.URISyntaxException;

public class RatKiller extends Application {
    private Socket mSocket;
    private static final String URL = "http://yoururl.com";

    @Override
    public void onCreate() {
        super.onCreate();
        try {
            mSocket = IO.socket(URL);
        } catch (URISyntaxException e) {
            throw new RuntimeException(e);
        }
    }
	public Socket getmSocket(){
    	return mSocket;
	}
}

{% endhighlight %}
	
Here in the application class, declare a private variable `socket` which is we are going to use as a reference you know.<br>
The helper method `getmSocket()` will return the `socket` instance which we can use in our whole project.


### 2 Connecting the socket

In oreder to connect the socket to server we need to use the `connect()` method on the socket instance returned from `getmSocket()` method.
Inside your `MainActivity` or any activity, add the following code in `onCreate()` method



{% highlight java %}

public class MainActivity extends AppCompatActivity {

private Socket mSocket;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);


    RatKiller app = (RatKiller)getApplication();
    mSocket = app.getmSocket();
    //
    //
    //...
}
{% endhighlight %}



To check whether connection is established or not you can check by using `connected()` method



{% highlight java %}

if (mSocket.connected()){
 	Toast.makeText(MainActivity.this, "Connected !!",Toast.LENGTH_SHORT).show();
 }
{% endhighlight %}


### 3 Listen or Emitt event

To emit event you can use `emit()` method.The `emit()` triggers an event on the server which you are going to implement as a button click or any actions response.


{% highlight java %}
	button.setOnCLickListener(view->mSocket.emit("kill",poisonObject));
{% endhighlight %}


To listen specific events sent by server, we use `on()` method. The `on()` method opens a channel provided and recieves the response from server.



{% highlight java %}

mSocket.on("rat_data", new Emitter.Listener() {
    @Override
    public void call(Object... args) {
        JSONObject data = (JSONObject)args[0];

        //here the data is in JSON Format
        //Toast.makeText(MainActivity.this, data.toString(), Toast.LENGTH_SHORT).show();

    }
});
{% endhighlight %}


Sometimes you nedd to listen the response of certain events which client/app is going to trigger either via button click or any action triggered etc. 
In such cases, we can call the `on()` method over `emit()` method. The `emit()` method will trigger some event on specific route/channel and the `on()` will immediatey listen the response send by the server.
You can achieve this by doing something like



{% highlight java %}

mSocket.emit("get_rat_data").on("rat_data", new Emitter.Listener() {
    @Override
    public void call(Object... args) {
        JSONObject data = (JSONObject)args[0];

        //data is in JSOn format
        

    }
});
{% endhighlight %}


### Data/Payload over request
The request payload or request body data must be send in `JSON format` to socket. The `JSON Object` is the input parameter of `emit()` method. The Java's `JSONObject` and `JSONArray` class helps us to construct a JSON body format of the data.
You can do this as 



{% highlight java %}
JSONArray poisonArray = new JSONArray();
poisonArray.put(1);
poisonArray.put(2);
JSONObject poisonObject = new JSONObject();

try {
    
    poisonObject.put("poisons",poisonArray);

} catch (JSONException e) {
    e.printStackTrace();
}
{% endhighlight %}


## Implementing the UI 

Every time when you receive the data as a response from the server, you need to update your UI. To update the UI using received data we need to perform all the UI bind login on `runOnUiThread()` method.

The `call()` method from `Emmiter.Listener{...}` used to work on a background thread. If you try to call the UI specific method inside the call,it'll blow up by blasting our favourite `Runtime Exceptions`.
You can achieve this by calling a `runOnUiThread(Runnable r)` method.



{% highlight java %}
mSocket.emit("kill",poisonObject).on("rat_data", new Emitter.Listener() {
    @Override
    public void call(Object... args) {
	    JSONObject data = (JSONObject)args[0];

	    //data is in JSOn format
	    runOnUiThread(new Runnable() {
	        @Override
	        public void run() {

	            Toast.makeText(MainActivity.this, "Haha !! All rats are killed !", Toast.LENGTH_SHORT).show();

	            ratTextView.setText("0");

	            //whatever your UI logic
	        }
	    });
    }
});
{% endhighlight %}


And done the problem is solved now 


![](https://i2.wp.com/insurancebusters.net/wp-content/uploads/2014/12/InsuranceBusters.net-problem-solved-meme.jpg?fit=312%2C312)

Any queries?let me know in comment section :-)

I'm gonna kill those rats in my house.





