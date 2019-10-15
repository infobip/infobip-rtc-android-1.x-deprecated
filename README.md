### Introduction
Infobip RTC is an Android SDK which enables you to take advantage of Infobip's WebRTC platform. It gives you the ability to enrich your Android applications with real-time communications in minimum time while focusing on your application's user experience and business logic. We currently support audio calls between two WebRTC users and phone calls between WebRTC user and actual phone device.

Here you will find an overview and quick guide on how to connect to Infobip WebRTC platform. There is also in-depth reference documentation available.

### First-time setup
In order to use Infobip RTC, you need to have WebRTC enabled on your account, and that's it! You are ready to make WebRTC calls. Please contact your account manager to enable WebRTC.

### Getting SDK
You can get a distribution of our SDK through Gradle dependency that you will pull from jCenter maven repository. Just add this snippet to your `build.gradle`:
```
dependencies {
    ...
    implementation ('com.infobip:infobip-rtc:+@aar') {
            transitive = true
    }
}
```

### Authentication
Since Infobip RTC is just SDK, it means you are developing your own application, and you only use Infobip RTC as dependency. Your application has your own users, which we will call subscribers throughout this guide. So, in order to use Infobip RTC, you need to register your subscribers to our platform. Credentials your subscribers use to connect to your application are irrelevant to Infobip. We only need identity with which they will use to present themselves. And when we have their identity, we can generate a token that you will assign for them to use. With that token, your subscribers can connect to our platform (using Infobip RTC SDK).

In order to generate these tokens for your subscribers, you need to call our [`/webrtc/1/token`](https://dev.infobip.com/webrtc/generate-token) HTTP API method with proper parameters. Also, you will authenticate yourself against Infobip platform there, so we can relate the subscriber's token to you. Typically, generating token occurs after your subscribers are authenticated inside your application.
You will receive the token in response, that you will use to make calls and start listening for incoming calls.

### Permissions
The only permission with `dangerous` protection level SDK needs is [`RECORD_AUDIO`](https://developer.android.com/reference/android/Manifest.permission.html#RECORD_AUDIO). That means you need to ask for it in runtime, inside your application. [Here](https://developer.android.com/training/permissions/requesting) is how you can do that.  
There are also 3 permissions with a `normal` protection level that are included in SDK's manifest: [`ACCESS_NETWORK_STATE`](https://developer.android.com/reference/android/Manifest.permission.html#ACCESS_NETWORK_STATE), [`INTERNET`](https://developer.android.com/reference/android/Manifest.permission.html#INTERNET), [`MODIFY_AUDIO_SETTINGS`](https://developer.android.com/reference/android/Manifest.permission.html#MODIFY_AUDIO_SETTINGS).

### Making a call
You can call another WebRTC subscriber, if you know it's identity. It is done via [`call`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call) method:

```
String token = obtainToken();
CallRequest callRequest = new CallRequest(
    token,
    getApplicationContext(),
    "Alice",
    new DefaultCallEventListener()
);

OutgoingCall call = InfobipRTC.call(callRequest);
```

As you can see, [`call`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call) method returns an instance of [`OutgoingCall`](https://github.com/infobip/infobip-rtc-android/wiki/OutgoingCall) as a result. With it, you can track status of your call and invoke call actions. With `callEventListener` param, you can set-up event handlers, so you can do something when called subscriber answers the call, rejects it, when the call is ended, etc. You set-up event handlers with this code:

```
CallEventListener callEventListener = new CallEventListener() {
    @Override
    public void onRinging(CallRingingEvent callRingingEvent) {
        Log.d("WebRTC", "Call is ringing on Alice's device!");
    }

    @Override
    public void onEarlyMedia(CallEarlyMediaEvent callEarlyMediaEvent) {
        Log.d("WebRTC", "Ringback tone received!");
    }

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
};
```

A most important part of the call is definitively media that travels across subscribers. It starts flowing after the `established` event is received. You do not have to do anything special to enable media flow (receive caller audio and send your audio to the caller), it will be done automatically.

When event handlers are set-up and call is established, there are few things that you can do with the actual call. One of them, of course, is to hangup. That can be done via [`hangup`](https://github.com/infobip/infobip-rtc-android/wiki/Call#hangup) method on call, and after that, both parties will receive `hangup` event upon hangup completion.

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

To check if audio is muted, call [`muted`](https://github.com/infobip/infobip-rtc-android/wiki/Call#muted) method like this:

```
boolean audioMuted = outgoingCall.muted();
```

Sound can also be played on speakerphone during the call. That option can be toggled as many times as you like, just call [`speakerphone`](https://github.com/infobip/infobip-rtc-android/wiki/Call#speakerphone) method with the appropriate parameter. By the default, it is disabled, you can enable it like this:

```
outgoingCall.speakerphone(true);
```

To check if speakerphone is enabled, call [`speakerphone`](https://github.com/infobip/infobip-rtc-android/wiki/Call#isSpeakerphone) method like this:

```
boolean speakerphoneEnabled = outgoingCall.speakerphone();
```

Also, you can check [`call status`](https://github.com/infobip/infobip-rtc-android/wiki/CallStatus):

```
CallStatus status = outgoingCall.status();
```

### Calling phone number
It is very much similar to calling regular WebRTC user, you just use [`callPhoneNumber`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#callPhoneNumber) method instead [`call`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call). There is also method overload that accepts second parameter, options in which you can define from parameter. It's value will display on calling phone device as Caller ID. Result of [`callPhoneNumber`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#callPhoneNumber) is also [`OutgoingCall`](https://github.com/infobip/infobip-rtc-android/wiki/OutgoingCall) that you can do everything you could when using the [`call`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call) method:

```
String token = obtainToken();
CallRequest callRequest = new CallRequest(
    token,
    getApplicationContext(),
    "41793026727",
    new DefaultCallEventListener()
);
CallPhoneNumberOptions callPhoneNumberOptions = CallPhoneNumberOptions.builder().from("33755531044").build();

OutgoingCall call = InfobipRTC.callPhoneNumber(callRequest, callPhoneNumberOptions);
```

### Receiving a call
There are two ways of being able to receive a call. Each one requires you starting listening for incoming calls.

#### Receiving a call via push notification
First way is listening for push notifications, that we will send from our Infobip WebRTC platform on your behalf to correct device. In that case you need to provide us your server key, so we can send push notifications on your behalf. That can be achieved by contacting your account manager. Then, you handle them in your application by extending [`FirebaseMessagingService`](https://firebase.google.com/docs/reference/android/com/google/firebase/messaging/FirebaseMessagingService) and overriding `onMessageReceived` method. There you can relay push notification to our SDK via [`handleIncomingCall`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#handleIncomingCall) method. [Here](https://firebase.google.com/docs/android/setup) you can find complete tutorial on how to add Firebase to your app.
  
This is recommended approach since it does not use much battery, because connection is not kept alive, it only listens for incoming push notifications.  
  
This is an example how you start listening to push notifications:
```
String token = obtainToken();
InfobipRTC.enablePushNotification(
    token,
    getApplicationContext()
);
```

And then, you can handle it in your application like this:
```
class FcmService extends FirebaseMessagingService {
    @Override
    public void onMessageReceived(RemoteMessage remoteMessage) {
        Map<String, String> payload = remoteMessage.getData();
        InfobipRTC.handleIncomingCall(
            payload,
            getAplicationContext(),
            new IncomingCallEventListener() {
                @Override
                public void onIncomingCall(IncomingCallEvent incomingCallEvent) {
                    IncomingCall incomingCall = incomingCallEvent.getIncomingCall();
                    Log.d("WebRTC", "Received incoming call from:  " + incomingCall.source().getIdentity() + " " + incomingCall.source().getDisplayName() );
                    incomingCall.setEventListener(new DefaultCallEventListener());
                    incomingCall.accept(); // or incomingCall.decline();
                }
            }
        );
    }
}
```
If you take a closer look at how to implement FCM in you application, you will see that there is unique device token associated with one app install on single device.  
When you call [`enablePushNotification`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#enablePushNotification) method on a device, we take that device's token, and associate it with identity on our Infobip WebRTC platform.  

But, those device tokens can change during app lifetime, so you need to handle those changes. We implemented Android service named `FcmTokenRefresher` that handles changes, you just need to include it in your app's manifest:
```
<application>
    <service
        android:name="com.infobip.webrtc.sdk.impl.fcm.FcmTokenRefresher"
        android:exported="false" />
</application>
```

#### Receiving a call via active connection
Second way is to connect once via WebSocket connection to our Infobip WebRTC platform, keep it active, and receive calls via it. All that is implemented in our SDK, you just need to call [`registerForActiveConnection`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#registerForActiveConnection) method to actually start listen for incoming calls. Third parameter is listener that will be fired on incoming call.  
  
Downside of this approach is that your app will consume significant amount of battery, because it persists connection. First approach is recommended.

```
String token = obtainToken();
InfobipRTC.registerForActiveConnection(
    token,
    getApplicationContext(),
    new IncomingCallEventListener() {
        @Override
        public void onIncomingCall(IncomingCallEvent incomingCallEvent) {
            IncomingCall incomingCall = incomingCallEvent.getIncomingCall();
            Log.d("WebRTC", "Received incoming call from:  " + incomingCall.source().getIdentity() + " " + incomingCall.source().getDisplayName() );
            incomingCall.setEventListener(new DefaultCallEventListener());
            incomingCall.accept(); // or incomingCall.decline();
        }
    }
);
```


