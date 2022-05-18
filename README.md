### Introduction

Infobip RTC is an Android SDK which enables you to take advantage of Infobip platform since it gives you the ability to enrich
your Android applications with real-time communications in minimum time,
allowing you to focus on your application's user experience and business logic.
We currently support audio and video calls between two web or app users, and phone calls between a web or an app user and an actual phone device.

Here you will find an overview and a quick guide on how to connect to Infobip platform. There is also an in-depth reference documentation available.

### First-time setup

In order to use Infobip RTC, you need to have Web and In-app Calls enabled on your account and that's it!
You are ready to make Web and In-app calls. To learn how to enable them see [the documentation](https://www.infobip.com/docs/voice-and-video/webrtc#set-up-web-and-in-app-calls).

### Getting SDK

You can get our SDK through Gradle dependency which you pull from the Maven Central repository. Just add this snippet to your `build.gradle`:

```groovy
dependencies {
    ...
    implementation ('com.infobip:infobip-rtc:+@aar') {
            transitive = true
    }
}
```

### Authentication

Since Infobip RTC is an SDK, it means you develop your own application, and you only use Infobip RTC as a dependency.
Your application has your own users, which we will call subscribers throughout this guide. So, in order to use Infobip RTC,
you need to register your subscribers on our platform. The credentials your subscribers use to connect to
your application are irrelevant to Infobip. We only need the identity they will use to present themselves.
When we have the subscriber's identity, we can generate a token assigned to that specific subscriber.
With that token, your subscribers can connect to our platform (using Infobip RTC SDK).

To generate these tokens for your subscribers, you need to call our
[`/webrtc/1/token`](https://dev.infobip.com/webrtc/generate-token) HTTP API method using proper parameters.
There you authenticate yourself against Infobip platform, so we can relate the subscriber's token to you.
Typically, generating a token occurs after your subscribers are authenticated inside your application.
You will receive the token in a response that you will use to make and receive calls via
[`InfobipRTC`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC) client in your Android application.

### Permissions

The only `dangerous` SDK permissions needed are the
[`RECORD_AUDIO`](https://developer.android.com/reference/android/Manifest.permission.html#RECORD_AUDIO)
and [`CAMERA`](https://developer.android.com/reference/android/Manifest.permission#CAMERA) permissions.
That means you need to ask for them in runtime, inside your application.
Here is how you can do that for [record audio](https://developer.android.com/reference/android/Manifest.permission.html#RECORD_AUDIO) and
how for [camera](https://developer.android.com/reference/android/Manifest.permission#CAMERA).
There are also three permissions with the `normal` protection level that are included in the SDK's manifest:

- [`ACCESS_NETWORK_STATE`](https://developer.android.com/reference/android/Manifest.permission.html#ACCESS_NETWORK_STATE)
- [`INTERNET`](https://developer.android.com/reference/android/Manifest.permission.html#INTERNET)
- [`MODIFY_AUDIO_SETTINGS`](https://developer.android.com/reference/android/Manifest.permission.html#MODIFY_AUDIO_SETTINGS)

### Making a call

You can call another subscriber if you know their identity.
It is done via [`call`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call) method:

```java
String token = obtainToken();
CallRequest callRequest = new CallRequest(
    token,
    getApplicationContext(),
    "Alice",
    new DefaultCallEventListener()
);

OutgoingCall call = InfobipRTC.call(callRequest);
```

Or if you want to initiate video call:  

```java
String token = obtainToken();
CallRequest callRequest = new CallRequest(
    token,
    getApplicationContext(),
    "Alice",
    new DefaultCallEventListener()
);
CallOptions callOptions = CallOptions.builder().video(true).build();

OutgoingCall call = InfobipRTC.call(callRequest, callOptions);
```

As you can see, [`call`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call) method returns an instance of
[`OutgoingCall`](https://github.com/infobip/infobip-rtc-android/wiki/OutgoingCall) as a result.
With it, you can track status of your call and invoke call actions.
The `callEventListener` parameter, lets you set up event handlers,
so you can perform something when the called subscriber answers the call, rejects it, the call is ended, etc.
You set up event handlers with the following code:

```java
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

The most important part of the call is definitely the media that travels between subscribers. It starts after the `established` event is received.
In case of audio call, you don’t have to do anything to enable the media flow (receive the caller audio and send your audio to the caller),
it is done automatically. In case of video call, you need to attach the video track received in `established` event to video UI element.
You can use `com.infobip.webrtc.sdk.api.video.VideoRenderer` class to display remote video (and local, if you wish) on the UI:  

```xml
<com.infobip.webrtc.sdk.api.video.VideoRenderer
  android:id="@+id/remote_video"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content" />
```

Then, in your code, set up this renderer:  

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    //...
    VideoRenderer remoteVideoRenderer = findViewById(R.id.remote_video);
    remoteVideoRenderer.init();
    remoteVideoRenderer.setScalingType(RendererCommon.ScalingType.SCALE_ASPECT_FIT); // choose scaling type that best fits your needs
}
```

When `established` event is received, connect the video track to renderer:  

```java
@Override
public void onEstablished(CallEstablishedEvent callEstablishedEvent) {
    Log.d("WebRTC", "Alice answered call!");
    VideoRenderer remoteVideoRenderer = findViewById(R.id.remote_video);
    RTCVideoTrack remoteRTCVideoTrack = callEstablishedEvent.getRemoteRTCVideoTrack();
    remoteRTCVideoTrack.addSink(remoteVideoRenderer);
    // do same for local video, if needed
}
```

When event handlers are set up and the call is established, there are a few things that you can do with the actual call.
One of them, of course, is to hang up.
That can be done via the [`hangup`](https://github.com/infobip/infobip-rtc-android/wiki/Call#hangup) method on the call,
and after that, both parties receive `hangup` event upon hang up completion.

```java
outgoingCall.hangup();
```

You can simulate a digit press during the call, by sending DTMF codes (Dual-Tone Multi-Frequency).
This is achieved via [`sendDTMF`](https://github.com/infobip/infobip-rtc-android/wiki/Call#sendDTMF) method.
Valid DTMF codes are digits from `0`-`9`, `*` and `#`.

```java
outgoingCall.sendDTMF("*");
```

During the call, you can also mute (and unmute) your audio:

```java
outgoingCall.mute(true);
```

To check if audio is muted, call [`muted`](https://github.com/infobip/infobip-rtc-android/wiki/Call#muted) method in the following manner:

```java
boolean audioMuted = outgoingCall.muted();
```

Sound can also be played on the speakerphone during the call. That option can be toggled as many times as you like,
just call the [`speakerphone`](https://github.com/infobip/infobip-rtc-android/wiki/Call#speakerphone) method with the appropriate parameter.
By default, it is disabled, you can enable it as shown below:

```java
outgoingCall.speakerphone(true);
```

To check if the speakerphone is enabled,
call the [`speakerphone`](https://github.com/infobip/infobip-rtc-android/wiki/Call#isSpeakerphone) method as shown in the example below:

```java
boolean speakerphoneEnabled = outgoingCall.speakerphone();
```

Also, you can check the [`call status`](https://github.com/infobip/infobip-rtc-android/wiki/CallStatus):

```java
CallStatus status = outgoingCall.status();
```

Also, you can check information such as [`duration`](https://github.com/infobip/infobip-rtc-android/wiki/Call#duration),
[`start time`](https://github.com/infobip/infobip-rtc-android/wiki/Call#startTime),
[`establish time`](https://github.com/infobip/infobip-rtc-android/wiki/Call#establishTime)
and [`end time`](https://github.com/infobip/infobip-rtc-android/wiki/Call#endTime) by calling these methods:

```java
int duration = outgoingCall.duration();
Date startTime = outgoingCall.startTime();
Date establishTime = outgoingCall.establishTime();
Date endTime = outgoingCall.endTime();
```

### Calling phone number

It is very much similar to calling a regular WebRTC user,
you just use the [`callPhoneNumber`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#callPhoneNumber) method instead of a
[`call`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call).
There is also the method overload that accepts a second parameter, where you can define the `from` parameter.
Its value will display the calling phone device as the Caller ID.
The result of the [`callPhoneNumber`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#callPhoneNumber) is also the
[`OutgoingCall`](https://github.com/infobip/infobip-rtc-android/wiki/OutgoingCall) with which you can do everything as when using the
[`call`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call) method:

```java
String token = obtainToken();
CallRequest callRequest = new CallRequest(
    token,
    getApplicationContext(),
    "41793026727",
    new DefaultCallEventListener()
);
CallPhoneNumberOptions callPhoneNumberOptions = CallPhoneNumberOptions.builder().from("33712345678").build();

OutgoingCall call = InfobipRTC.callPhoneNumber(callRequest, callPhoneNumberOptions);
```

### Receiving a call

There are two ways of receiving a call. Each one requires you to listen for incoming calls.

#### Receiving a call via push notification

> Note: In order for push notifications to work, they have to be enabled for your application, as explained in [the documentation](https://www.infobip.com/docs/voice-and-video/web-and-in-app-calls#create-and-configure-applicationhttps://www.infobip.com/docs/voice-and-video/web-and-in-app-calls#create-and-configure-application).

The first way is to listen for push notifications which we send from our platform on your behalf to the correct device and in that case you
need to provide us your FCM server key.
After that, you handle them in your application by extending
[`FirebaseMessagingService`](https://firebase.google.com/docs/reference/android/com/google/firebase/messaging/FirebaseMessagingService)
and overriding the `onMessageReceived` method.
There you can relay push notification to our SDK via
[`handleIncomingCall`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#handleIncomingCall) method.
[Here](https://firebase.google.com/docs/android/setup) you can find complete tutorial on how to add Firebase to your app.
  
This is the recommended approach since it doesn't use much battery, as the connection is not kept alive,
it only listens for incoming push notifications.  
  
This is an example how to listen for push notifications:

```java
String token = obtainToken();
InfobipRTC.enablePushNotification(
    token,
    getApplicationContext()
);
```

After which you can handle that in your application as shown below:

```java
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

If you take a closer look at how to implement FCM in your application,
you will see that there is a unique device token associated with one app installation on a single device.
When you call the [`enablePushNotification`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#enablePushNotification) method
on a device, we take that device's token and associate it with an identity on our Infobip WebRTC platform.

As device tokens can change during app lifetime, you need to handle these changes.
We implemented an Android service named `FcmTokenRefresher` that handles changes, you just need to include it in your app's manifest:

```xml
<application>
    <service
        android:name="com.infobip.webrtc.sdk.impl.fcm.FcmTokenRefresher"
        android:exported="false" />
</application>
```

#### Receiving a call via active connection

The second way to receive a call is to connect once via WebSocket connection to our Infobip WebRTC platform, keep it active,
and receive calls via that connection. All this is implemented in our SDK, you just need to call the
[`registerForActiveConnection`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#registerForActiveConnection) method
to actually start listening for incoming calls. The third parameter is the listener that is fired upon the incoming call.

The downside of this approach is that your app will consume a significant amount of battery, because it persists the connection.
We recommend the first approach.

```java
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

### Conference call

You can have a conference call with other participants that are also in the same conference room.
The conference call will start as soon as at least one participant joins.

Conference call is in the beta stage and available for audio only, with a maximum limit of 12 participants.

Joining the room is done via the [`joinConference`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#joinConference) method:

```java
String token = obtainToken();
String conferenceId = "conference-demo"
ConferenceRequest conferenceRequest = new ConferenceRequest(
    getApplicationContext(), 
    conferenceId, 
    token, 
    new ConferenceEventListener() { 
        @Override
        public void onJoined(JoinedEvent joinedEvent) {
            Toast.makeText(getApplicationContext(), "You have joined the conference!", Toast.LENGTH_LONG);
        }
    
        @Override
        public void onLeft(LeftEvent leftEvent) {
            Toast.makeText(getApplicationContext(), "You left the conference!", Toast.LENGTH_LONG);
        }
    
        @Override
        public void onUserJoined(UserJoinedEvent userJoinedEvent) {
            Toast.makeText(getApplicationContext(), userJoinedEvent.getUser().getIdentity() + " joined the conference!", Toast.LENGTH_LONG);
        }
    
        @Override
        public void onUserMuted(UserMutedEvent userMutedEvent) {
            Toast.makeText(getApplicationContext(), userMutedEvent.getIdentity() + " muted himself!", Toast.LENGTH_LONG);
        }
    
        @Override
        public void onUserUnmuted(UserUnmutedEvent userUnmutedEvent) {
            Toast.makeText(getApplicationContext(), userMutedEvent.getIdentity() + " unmuted himself!", Toast.LENGTH_LONG);
        }
    
        @Override
        public void onUserLeft(UserLeftEvent userLeftEvent) {
            Toast.makeText(getApplicationContext(), userMutedEvent.getIdentity() + " left the conference!", Toast.LENGTH_LONG);
        }
    
        @Override
        public void onError(ErrorEvent errorEvent) {
            Toast.makeText(getApplicationContext(), "Conference error!", Toast.LENGTH_LONG);
        }
    };);
    Conference conference = InfobipRTC.joinConference(conferenceRequest);
```

After the user successfully joined the conference, the [`JoinedEvent`](https://github.com/infobip/infobip-rtc-android/wiki/JoinedEvent) event with a list of
`users` that are already in that conference room, will be emitted, so that you can show those users on his screen.
Also, the rest of the users will receive [`UserJoinedEvent`](https://github.com/infobip/infobip-rtc-android/wiki/UserJoinedEvent) event,
with information about the user that just joined the conference, so that you could show that user on their screens.

When you made [`ConferenceRequest`](https://github.com/infobip/infobip-rtc-android/wiki/ConferenceRequest), you have already implemented
[`ConferenceEventListener`](https://github.com/infobip/infobip-rtc-android/wiki/ConferenceEventListener) as an event handler for these events.

As you can see, the [`joinConference`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#joinConference) method returns an instance of
[`Conference`](https://github.com/infobip/infobip-rtc-android/wiki/Conference) as the result.
With it, you can track the status of your conference call and there are a few actions (mute, leave, speakerphone...) that you can do with the actual conference.

Leaving the conference might be done via the [`leave`](https://github.com/infobip/infobip-rtc-android/wiki/Conference#leave) method at the conference.
On the side of the other participants, the [`UserLeftEvent`](https://github.com/infobip/infobip-rtc-android/wiki/UserLeftEvent) event will be fired upon leave completion.

```java
conference.leave();
```

During the conference call, you can also mute (and unmute) your audio,
by calling the [`mute(shouldMute)`](https://github.com/infobip/infobip-rtc-android/wiki/Conference#mute) method in the following way:

```java
conference.mute(true);
```

On the side of the other participants, the [`UserMutedEvent`](https://github.com/infobip/infobip-rtc-android/wiki/UserMutedEvent) or
[`UserUnmutedEvent`](https://github.com/infobip/infobip-rtc-android/wiki/UserUnmutedEvent) event will be fired.

To check if the audio is muted, call the [`muted()`](https://github.com/infobip/infobip-rtc-android/wiki/Conference#muted) method in the following way:

```java
boolean audioMuted = conference.muted();
```

During the conference call, you can play audio on the speakerphone,
by calling the [`speakerphone(enabled)`](https://github.com/infobip/infobip-rtc-android/wiki/Conference#speakerphone) method in the following way:

```java
conference.speakerphone(true);
```

To check if audio is on the speakerphone,
call the [`speakerphone()`](https://github.com/infobip/infobip-rtc-android/wiki/Conference#speakerphone) method in the following way:

```java
boolean speakerphone = conference.speakerphone();
```

### Supported API Levels

The SDK supports Android API Level 21 (Lollipop) and higher.

### Java Compatibility

The SDK source and target compatibility is set to Java 8. Reference the following snippet:

```groovy
android {
    compileOptions {
        sourceCompatibility 1.8
        targetCompatibility 1.8
    }
}
```
