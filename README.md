### Introduction
Infobip RTC is an Android SDK which enables you to take advantage of Infobip's WebRTC platform. It gives you ability to enrich your web applications with real-time communications in minimum time, while focusing on your application's user experience and business logic. We currently support audio calls between two WebRTC users, and phone calls between WebRTC user and actual phone device.

Here you will find overview and quick guide how to connect to Infobip WebRTC platform. There is also in-depth reference documentation available.

### First time setup
In order to use Infobip RTC, you need to have WebRTC enabled on your account, and that's it! You are ready to make WebRTC calls. Please contact your account manager to enable WebRTC.

### Getting SDK
You can get distribution of our SDK through gradle dependency that you will pull from jCenter maven repository. Just add this snippet to your `build.gradle`:
```
dependencies {
    ...
    implementation ('com.infobip:infobip-rtc:+@aar') {
            transitive = true
    }
}
```

### Authentication
Since Infobip RTC is just SDK, it means you are developing your own application, and you only use Infobip RTC as dependency. Your application has your own users, which we wall call subscribers throughout this guide. So, in order to use Infobip RTC, you need to register your subscribers to our platform. Credentials your subscribers use to connect to your application are irrelevant to Infobip. We only need identity with which they will use to present themselves. And when we have their identity, we can generate token that you will assign for them to use. With that token, your subscribers can connect to our platform (using Infobip RTC SDK).

In order to generate these tokens for your subscribers, you need to call our [`/webrtc/token`](https://dev.infobip.com/webrtc/generate-token) HTTP API method with proper parameters. Also, there you will authenticate yourself against Infobip platform, so we can relate subscriber's token to you. Typically, generating token occurs after your subscribers are authenticated inside your application.
In response you will receive token, that you will use to instantiate InfobipRTC client in your web application.

### Permissions
Only permission with `dangerous` protection level SDK needs is [`RECORD_AUDIO`](https://developer.android.com/reference/android/Manifest.permission.html#RECORD_AUDIO). That means you need to ask for it in runtime, inside your application. [Here](https://developer.android.com/training/permissions/requesting) is how you can do that.  
There are also 3 permissions with `normal` protection level that are included in SDK's manifest: [`ACCESS_NETWORK_STATE`](https://developer.android.com/reference/android/Manifest.permission.html#ACCESS_NETWORK_STATE), [`INTERNET`](https://developer.android.com/reference/android/Manifest.permission.html#INTERNET), [`MODIFY_AUDIO_SETTINGS`](https://developer.android.com/reference/android/Manifest.permission.html#MODIFY_AUDIO_SETTINGS).

### Infobip RTC Client
After you received token via HTTP API, you are ready to instantiate [`InfobipRTC`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC) client. It can be done using these commands:

```
String token = obtainToken(); // here you call '/webrtc/token'
RTCOptions options = RTCOptions.builder().build();
Context context = getApplicationContext();

InfobipRTC infobipRTC = DefaultInfobipRTC.getInstance(token, options, context);
```

Note that this does not actually connect to Infobip WebRTC platform, it just creates new instance of [`InfobipRTC`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC). Connecting is done via [`connect`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#connect) method. Before connecting, it is useful to set-up event handlers, so you can do something when connection is set-up, when connection is lost, etc. Events are set-up via [`addEventListener`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#addEventListener) method:

```
infobipRTC.addEventListener(new RTCEventListener() {
    @Override
    public void onConnected(ConnectedEvent connectedEvent) {
        Log.d("WebRTC", "Connected with identity: " + connectedEvent.getIdentity());
    }

    @Override
    public void onDisconnected(DisconnectedEvent disconnectedEvent) {
        Log.d("WebRTC", "Disconnected with reason: " + disconnectedEvent.getReason());
    }

    @Override
    public void onReconnecting(ReconnectingEvent reconnectingEvent) {
        Log.d("WebRTC", "Reconnecting!");
    }

    @Override
    public void onReconnected(ReconnectedEvent reconnectedEvent) {
        Log.d("WebRTC", "Reconnected!");
    }
});
```

Now you are ready to connect:

```
infobipRTC.connect();
```

### Making a call
You can call another WebRTC subscriber, if you know it's identity. It is done via [`call`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call) method:

```
OutgoingCall outgoingCall = infobipRTC.call("Alice", CallOptions.builder().build());
```

As you can see, [`call`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call) method returns instance of [`OutgoingCall`](https://github.com/infobip/infobip-rtc-android/wiki/OutgoingCall) as a result. With it you can track status of your call and respond to events. Similar as for client, you can set-up event handlers, so you can do something when called subscriber answers the call, rejects it, when call is ended, etc. You set-up event handlers with this code:

```
outgoingCall.addEventListener(new CallEventListener() {
    @Override
    public void onEstablished(CallEstablishedEvent callEstablishedEvent) {
        Log.d("WebRTC", "Alice answered call!");
    }

    @Override
    public void onHangup(CallHangupEvent callHangupEvent) {
        Log.d("WebRTC", "Call is done! Status: " + callHangupEvent.getErrorCode().toString());
    }

    @Override
    public void onError(CallErrorEvent callErrorEvent) {
        Log.d("WebRTC", "Oops, something went very wrong! Message: " + callErrorEvent.getReason().toString());
    }

    @Override
    public void onRinging(CallRingingEvent callRingingEvent) {
        Log.d("WebRTC", "Call is ringing on Alice's device!");
    }
});
```

Most important part of call is definitively media that travels across subscribers. It starts flowing after `established` event is received. You do not have to do anything special to enable media flow (receive caller audio and send your audio to caller), it will be done automatically.

When event handlers are set-up and call is established, there are few things that you can do with actual call. One of them, of course, is to hangup. That can be done via [`hangup`](https://github.com/infobip/infobip-rtc-android/wiki/Call#hangup) method on call, and after that, both parties will receive `hangup` event upon hangup completion.

```
outgoingCall.hangup();
```

You can simulate digit press during the call, by sending DTMF codes (Dual-Tone Multi-Frequency). It is achieved via [`sendDTMF`](https://github.com/infobip/infobip-rtc-android/wiki/Call#sendDTMF) method. Valid DTMF codes are digits `0`-`9`, `*` and `#`.

```
outgoingCall.sendDTMF("*");
```

During the call, you can also mute (and unmute) your audio:

```
outgoingCall.mute(true);
```

### Receiving a call
Besides making outgoing calls, you can also receive incoming calls. In order to do that, you need to register `incoming-call` event handler of [`InfobipRTC`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC) client. There you can define behavior on incoming call. One of the most common thing to do there is to show Answer and Reject options on some UI. For purposes of this guide, let's see example that answers incoming call as soon as it arrives:

```
infobipRTC.addIncomingCallEventListener(new IncomingCallEventListener() {
    @Override
    public void onIncomingCall(IncomingCallEvent incomingCallEvent) {
        IncomingCall incomingCall = incomingCallEvent.getIncomingCall();
        Log.d("WebRTC", "Received incoming call from:  " + incomingCall.source());
        incomingCall.addEventListener(new CallEventListener() {...});
        incomingCall.accept(); // or incomingCall.decline();
    }
    ...
});
```

If you are in the middle of a call, naturally, you cannot receive second call. So, if someone makes incoming call to you while you are talking, you will receive a `missed-call` event:

```
infobipRTC.addIncomingCallEventListener(new IncomingCallEventListener() {
    @Override
    public void onMissedCall(MissedCallEvent missedCallEvent) {
        Log.d("WebRTC", "Received incoming call from:  " + missedCallEvent.getCaller());
    }
    ...
});
```

### Calling phone number
It is very much similar to calling regular WebRTC user, you just use [`callPhoneNumber`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#callPhoneNumber) method instead [`call`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call). This method accepts optional second parameter, options in which you can define from parameter. It's value will display on calling phone device as Caller ID. Result of [`callPhoneNumber`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#callPhoneNumber) is also [`OutgoingCall`](https://github.com/infobip/infobip-rtc-android/wiki/OutgoingCall) that you can do everything you could when using [`call`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call) method:

```
OutgoingCall outgoingCall = infobipRTC.callPhoneNumber("41793026727", CallPhoneNumberOptions.builder().from("33755531044").build());
```
