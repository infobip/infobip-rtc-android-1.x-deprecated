### WebRTC SDK 1.x deprecation

Following the major release of our new RTC SDK 2.0, we are deprecating the SDK 1.x releases. The SDK 1.x will be out of
service on 31/10/2023. All new WebRTC customers must use the SDK 2.x, and customers still using SDK 1.x must migrate
to the newer release before the end of service date. To migrate from RTC SDK 1.x to 2.x, consult our
[migration guides](https://github.com/infobip/infobip-rtc-android/wiki/Migration-guides).

The deprecated [SDK 1.x Github repository](https://github.com/infobip/infobip-rtc-android-1.x-deprecated) can still be
consulted until the end of service date.

### Introduction

Infobip RTC is an Android SDK which enables you to take advantage of Infobip platform since it gives you the ability to enrich
your Android applications with real-time communications in minimum time, allowing you to focus on your application's user experience and business logic.
We currently support WebRTC calls between two web or app users, phone calls between a web or app user and
a phone endpoint, Viber calls, calls to the Infobip Conversations platform, as well as Room calls - calls with multiple participants.

Here you will find an overview and a quick guide on how to connect to Infobip platform. There is also an in-depth reference documentation available.

### First-time setup

In order to use Infobip RTC, you need to have Web and In-app Calls enabled on your account and that's it!
You are ready to make Web and In-app calls. To learn how to enable them see [the documentation](https://www.infobip.com/docs/voice-and-video/webrtc#set-up-web-and-in-app-calls).

### Getting SDK

You can get our SDK through Gradle dependency which you pull from the Maven Central repository. Just add this snippet to your `build.gradle`:

```groovy
dependencies {
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

### Making a WebRTC call

You can call another WebRTC endpoint if you know their identity.
This is done via [`callWebrtc`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call-webrtc) method:

```java
String token = obtainToken();
CallWebrtcRequest callWebrtcRequest = new CallWebrtcRequest(
    token,
    getApplicationContext(),
    "Alice",
    new DefaultWebrtcCallEventListener()
);

WebrtcCall call = InfobipRTC.callWebrtc(callWebrtcRequest);
```

Or if you want to initiate video call:  

```java
String token = obtainToken();
CallWebrtcRequest callWebrtcRequest = new CallWebrtcRequest(
    token,
    getApplicationContext(),
    "Alice",
    new DefaultWebrtcCallEventListener()
);
WebrtcCallOptions webrtcCallOptions = WebrtcCallOptions.builder().video(true).build();

WebrtcCall call = InfobipRTC.callWebrtc(callWebrtcRequest, webrtcCallOptions);
```

As you can see, [`callWebrtc`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call-webrtc) method returns an instance of
[`WebrtcCall`](https://github.com/infobip/infobip-rtc-android/wiki/WebrtcCall) as a result.
With it, you can track the status of your call, respond to events, and invoke call actions.

In order to respond to certain actions during the call, you need to set up event handlers.
You can forward said listener through the last parameter `webrtCallEventListener` in the WebRTC call request when making a WebRTC call (as shown in previous example), 
or you can set it up when call is created via the [`setEventListener`](https://github.com/infobip/infobip-rtc-android/wiki/WebrtcCall#set-event-listener) 
method as shown in the following example:

```java
WebrtcCallEventListener webrtcCallEventListener = new WebrtcCallEventListener() {
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
        Log.d("WebRTC", "Alice answered the call!");
    }

    @Override
    public void onHangup(CallHangupEvent callHangupEvent) {
        Log.d("WebRTC", "Call ended! Status: " + callHangupEvent.getErrorCode().toString());
    }

    @Override
    public void onError(ErrorEvent errorEvent) {
        Log.d("WebRTC", "Oops, something went wrong! Message: " + errorEvent.getErrorCode().toString());
    }
    
    @Override
    public void onCameraVideoAdded(CameraVideoAddedEvent cameraVideoAddedEvent) {
        Log.d("WebRTC", "Local camera video added!");
    }
    
    @Override
    public void onCameraVideoUpdated(CameraVideoUpdatedEvent cameraVideoUpdatedEvent) {
        Log.d("WebRTC", "Local camera video updated!");
    }
    
    @Override
    public void onCameraVideoRemoved() {
        Log.d("WebRTC", "Local camera video removed!");
    }
    
    @Override
    public void onScreenShareAdded(ScreenShareAddedEvent screenShareAddedEvent) {
        Log.d("WebRTC", "Local screen share added!");
    }
    
    @Override
    public void onScreenShareRemoved() {
        Log.d("WebRTC", "Local screen share removed!");
    }
    
    @Override
    public void onRemoteCameraVideoRemoved() {
        Log.d("WebRTC", "Remote camera video removed!");
    }
    
    @Override
    public void onRemoteScreenShareAdded(ScreenShareAddedEvent screenShareAddedEvent) {
        Log.d("WebRTC", "Remote screen share added!");
    }
    
    @Override
    public void onRemoteScreenShareRemoved() {
        Log.d("WebRTC", "Remote screen share removed!");
    }
    
    @Override
    public void onRemoteMuted() {
        Log.d("WebRTC", "Remote muted!");
    }
    
    @Override
    public void onRemoteUnmuted() {
        Log.d("WebRTC", "Remote unmuted!");
    }
};

webrtcCall.setEventListener(webrtcCallEventListener);
```

To get the current event listener, use the [`getEventListener`](https://github.com/infobip/infobip-rtc-android/wiki/WebrtcCall#get-event-listener) method:

```java
WebrtcCallEventListener webrtcCallEventListener = webrtcCall.getEventListener();
```

When event handlers are set up and the call is established, there are a few things that you can do with the actual call.
You can find the full list of actions available in the [`documentation`](https://github.com/infobip/infobip-rtc-android/wiki/WebrtcCall)

One of them, of course, is to hang up.
That can be done via the [`hangup`](https://github.com/infobip/infobip-rtc-android/wiki/WebrtcCall#hangup) method on the call,
and after that, both endpoints receive a `hangup` event upon hang up completion.

```java
webrtcCall.hangup();
```

You can simulate a digit press during the call, by sending DTMF codes (Dual-Tone Multi-Frequency).
This is achieved via [`sendDTMF`](https://github.com/infobip/infobip-rtc-android/wiki/Call#send-dtmf) method.
Valid DTMF codes are digits from `0`-`9`, `*` and `#`.

```java
webrtcCall.sendDTMF("*");
```

During the call, you can also mute (and unmute) your audio:

```java
webrtcCall.mute(true);
```

To check if audio is muted, call [`muted`](https://github.com/infobip/infobip-rtc-android/wiki/Call#muted) method in the following manner:

```java
boolean isAudioMuted = webrtcCall.muted();
```

Sound can also be played on the speakerphone during the call. That option can be toggled as many times as you like,
just call the [`speakerphone`](https://github.com/infobip/infobip-rtc-android/wiki/Call#set-speakerphone) method with the appropriate parameter.
By default, it is disabled, you can enable it as shown below:

```java
webrtcCall.speakerphone(true);
```

To check if the speakerphone is enabled,
call the [`speakerphone`](https://github.com/infobip/infobip-rtc-android/wiki/Call#get-speakerphone) method as shown in the example below:

```java
boolean isSpeakerphoneEnabled = webrtcCall.speakerphone();
```

Also, you can check the [`call status`](https://github.com/infobip/infobip-rtc-android/wiki/CallStatus):

```java
CallStatus status = webrtcCall.status();
```

Also, you can check call information, such as [`duration`](https://github.com/infobip/infobip-rtc-android/wiki/Call#duration),
[`start time`](https://github.com/infobip/infobip-rtc-android/wiki/Call#start-time),
[`establish time`](https://github.com/infobip/infobip-rtc-android/wiki/Call#establish-time)
and [`end time`](https://github.com/infobip/infobip-rtc-android/wiki/Call#end-time) by calling these methods:

```java
int duration = webrtcCall.duration();
Date startTime = webrtcCall.startTime();
Date establishTime = webrtcCall.establishTime();
Date endTime = webrtcCall.endTime();
```

At any time during the WebRTC call, users can add or remove their camera videos.  
In order to handle the local video media, You should set up the `onCameraVideoAddedEvent`, `onCameraVideoUpdatedEvent` and `onCameraVideoRemovedEvent` listeners.   
In order to handle remote video media, You should set up the `onRemoteCameraVideoAddedEvent` and `onRemoteCameraVideoRemovedEvent` listeners.

For handling both the local video and the remote video media, you need to attach the video track received in `cameraVideoAddedEvent` event to video UI element.

You can use `com.infobip.webrtc.sdk.api.video.VideoRenderer` class to display remote and local videos on the UI:

```xml
<com.infobip.webrtc.sdk.api.video.VideoRenderer
  android:id="@+id/remote_video"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content" />
```

Then, in your code, set up these renderers:

```java

@Override

protected void onCreate(Bundle savedInstanceState) {
    //...
    VideoRenderer localVideoRenderer = findViewById(R.id.local_video);
    localVideoRenderer.init();
    localVideoRenderer.setScalingType(RendererCommon.ScalingType.SCALE_ASPECT_FIT); // choose scaling type that best fits your needs
    
    VideoRenderer remoteVideoRenderer = findViewById(R.id.remote_video);
    remoteVideoRenderer.init();
    remoteVideoRenderer.setScalingType(RendererCommon.ScalingType.SCALE_ASPECT_FIT);
}
```

When the `cameraVideoAddedEvent` event or the `participantCameraVideoAddedEvent` is received, connect video tracks to their respective renderers:

```java
@Override
public void onCameraVideoAdded(CameraVideoAddedEvent cameraVideoAddedEvent) {
    Log.d("WebRTC", "Camera video added!");
    
    VideoRenderer localVideoRenderer = findViewById(R.id.local_video);
    RTCVideoTrack localRTCVideoTrack = cameraVideoAddedEvent.getTrack();
    localRTCVideoTrack.addSink(localVideoRenderer);
}

@Override
public void onRemoteCameraVideoAdded(CameraVideoAddedEvent cameraVideoAddedEvent) {
    Log.d("WebRTC", "Remote camera video added!");
    
    VideoRenderer remoteVideoRenderer = findViewById(R.id.remote_video);
    RTCVideoTrack remoteRTCVideoTrack = cameraVideoAddedEvent.getTrack();
    remoteRTCVideoTrack.addSink(remoteVideoRenderer);
}
```

You can turn on the camera using the
[`cameraVideo`](https://github.com/infobip/infobip-rtc-andorid/wiki/WebrtcCall#camera-video) method.

```java
webrtcCall.cameraVideo(true);
```

Users can also start or stop sharing their screen.
You should handle following events for local screen share: `screenShareAddedEvent` and `screenShareRemovedEvent`, and
for the remote screen share, you should handle the following events: `participantScreenShareAddedEvent` and `participantScreenShareRemovedEvent`

Upon receiving `screenShareAddedEvent` or `participantScreenShareAddedEvent`, You need to connect the appropriate video track to the appropriate renderer, just like handling camera video:

```java
@Override
public void onScreenShareAdded(ScreenShareAddedEvent screenShareAddedEvent) {
    Log.d("WebRTC", "Screen share added!");
    
    VideoRenderer localScreenShareRenderer = findViewById(R.id.local_screen_share);
    RTCVideoTrack localRTCScreenShareTrack = screenShareAddedEvent.getTrack();
    localRTCScreenShareTrack.addSink(localScreenShareRenderer);
}

@Override
public void onRemoteScreenShareAdded(ScreenShareAddedEvent screenShareAddedEvent) {
    Log.d("WebRTC", "Remote Screen share added!");
    
    VideoRenderer remoteScreenShareRenderer = findViewById(R.id.remote_screen_share);
    RTCVideoTrack remoteRTCScreenShareTrack = screenShareAddedEvent.getTrack();
    remoteRTCScreenShareTrack.addSink(remoteScreenShareRenderer);
}
```

You can turn on the screen share using the
[`startScreenShare`](https://github.com/infobip/infobip-rtc-andorid/wiki/WebrtcCall#start-screen-share) method.

```java
webrtcCall.startScreenShare();
```

And to turn the screen share off, You can use
[`stopScreenShare`](https://github.com/infobip/infobip-rtc-andorid/wiki/WebrtcCall#stop-screen-share) method.

```java
webrtcCall.stopScreenShare();
```

### Making a phone call

Making a phone call is similar to calling a regular WebRTC endpoint,
you just use the [`callPhone`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call-phone) method instead of
[`callWebrtc`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call-webrtc).
There is also a method overload which accepts a second parameter, [`phoneCallOptions`](https://github.com/infobip/infobip-rtc-android/wiki/PhoneCalloptions),
through which you can define parameters such as `audio`, `audioOptions` and `from`,
whose value will display the calling phone device as the Caller ID.
The result of the [`callPhone`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call-phone) is an instance of
[`PhoneCall`](https://github.com/infobip/infobip-rtc-android/wiki/PhoneCall), with which you can do several actions,
such as muting the call, hanging it up, checking its start time, answer time, duration and more.

```java
String token = obtainToken();
CallPhoneRequest callPhoneRequest = new CallPhoneRequest(
    token,
    getApplicationContext(),
    "41793026727",
    new DefaultPhoneCallEventListener()
);
PhoneCallOptions phoneCallOptions = PhoneCallOptions.builder().from("33712345678").build();

PhoneCall call = InfobipRTC.callPhone(callPhoneRequest, phoneCallOptions);
```

### Making a Viber call

Using the [`callViber`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call-viber) method is similar to
previously described methods. In this case, call's destination is Viber application. Unlike the
[`callPhone`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call-phone) method, parameter `from` is required and is
passed as part of the [`CallViberRequest`](https://github.com/infobip/infobip-rtc-android/wiki/CallViberRequest). Additionally, it has to be a Viber Voice number.
The result of the [`callViber`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#call-viber) is an instance of
[`ViberCall`](https://github.com/infobip/infobip-rtc-android/wiki/ViberCall) on which you can do several actions, such as
muting the call, hanging it up, checking its start time, answer time, duration and more.

```java
String token = obtainToken();
CallViberRequest callViberRequest = new CallViberRequest(
    token,
    getApplicationContext(),
    "41793026727",
    "33712345678",
    new DefaultViberCallEventListener()
);

ViberCall call = InfobipRTC.callViber(callViberRequest);
```

### Receiving a WebRTC call

There are two ways of receiving a call. Each one requires you to listen for incoming calls.

#### Receiving a call via push notification

> Note: In order for push notifications to work, they have to be enabled for your application, as explained in [the documentation](https://www.infobip.com/docs/voice-and-video/web-and-in-app-calls#create-and-configure-applicationhttps://www.infobip.com/docs/voice-and-video/web-and-in-app-calls#create-and-configure-application).

The first way is to listen for push notifications which we send from our platform on your behalf to the correct device and in that case you
need to provide us your FCM server key.
After that, you handle them in your application by extending
[`FirebaseMessagingService`](https://firebase.google.com/docs/reference/android/com/google/firebase/messaging/FirebaseMessagingService)
and overriding the `onMessageReceived` method.
There, you can relay push notification to our SDK via
[`handleIncomingCall`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#handle-incoming-call) method.
[Here](https://firebase.google.com/docs/android/setup), you can find complete tutorial on how to add Firebase to your app.
  
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

After which you can handle an incoming call in your application as shown below:

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
                public void onIncomingWebrtcCall(IncomingWebrtcCallEvent incomingWebrtcCallEvent) {
                    IncomingWebrtcCall incomingWebrtcCall = incomingWebrtcCallEvent.getIncomingWebrtcCall();
                    Log.d("WebRTC", "Received an incoming call from:  " + incomingWebrtcCall.source().identifier() + " " + incomingWebrtcCall.source().displayIdentifier());
                    incomingWebrtcCall.setEventListener(new DefaultWebrtcCallEventListener());
                    incomingWebrtcCall.accept(); // or incomingWebrtcCall.decline();
                }
            }
        );
    }
}
```

If you take a closer look at how to implement FCM in your application,
you will see that there is a unique device token associated with one app installation on a single device.
When you call the [`enablePushNotification`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#enable-push-notification) method
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

The second way to receive a call is to connect once via WebSocket connection to our Infobip WebRTC platform, keep the connection active,
and receive calls via that connection. All this is implemented in our SDK, you just need to call the
[`registerForActiveConnection`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#register-for-active-connection) method
to actually start listening for incoming calls. The third parameter is the listener that is fired upon an incoming call.

The downside of this approach is that your app will consume a significant amount of battery, because it persists the connection.
We recommend the first approach.

```java
String token = obtainToken();
InfobipRTC.registerForActiveConnection(
    token,
    getApplicationContext(),
    new IncomingCallEventListener() {
        @Override
        public void onIncomingWebrtcCall(IncomingWebrtcCallEvent incomingWebrtcCallEvent) {
            IncomingWebrtcCall incomingWebrtcCall = incomingWebrtcCallEvent.getIncomingWebrtcCall();
            Log.d("WebRTC", "Received incoming call from:  " + incomingWebrtcCall.source().identifier() + " " + incomingWebrtcCall.source().displayIdentifier());
            incomingWebrtcCall.setEventListener(new DefaultWebrtcCallEventListener());
            incomingWebrtcCall.accept(); // or incomingWebrtcCall.decline();
        }
    }
);
```

### Joining a room call

You can join a room call with other WebRTC endpoints. The room call will start as soon as at least one participant joins.
There can be up to 15 participants in a room simultaneously.

Joining the Room is done via the [`joinRoom`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#join-room) method:

```java
String token = obtainToken();
RoomRequest roomRequest = new RoomRequest(
    getApplicationContext(), 
    "room-demo", 
    token,
    new DefaultRoomCallEventListener()
);
RoomCall roomCall = InfobipRTC.joinRoom(roomRequest);
```

Also, You can join the room with video:

```java
String token = obtainToken();
RoomRequest roomRequest = new RoomRequest(
    getApplicationContext(), 
    "room-demo", 
    token,
    new DefaultRoomCallEventListener()
);
RoomCallOptions options = RoomCallOptions.builder().video(true).build()

RoomCall roomCall = InfobipRTC.joinRoom(roomRequest, options);
```

After the participant successfully joined the room, the [`RoomJoinedEvent`](https://github.com/infobip/infobip-rtc-android/wiki/RoomJoinedEvent) event, with a list of
`participants` that are already in that room, will be emitted, so that you can show those participants on the screen.
Also, the rest of the participants will receive [`ParticipantJoinedEvent`](https://github.com/infobip/infobip-rtc-android/wiki/ParticipantJoinedEvent) event,
with information about the participant that just joined the room, so that you can show that participant's information on other screens.

As you can see, the [`joinRoom`](https://github.com/infobip/infobip-rtc-android/wiki/InfobipRTC#join-room) method returns an instance of
[`RoomCall`](https://github.com/infobip/infobip-rtc-android/wiki/RoomCall) as the result.
With it, you can track the status of your room call and do some actions (mute, leave, speakerphone...) in the room.

In order to respond to certain actions during the room call, you need to set up event handlers.
You can forward said listener through the parameter `roomCallEventListener` in the room request when joining a room (as shown in previous example),
or you can set it up when the room call is created via the [`setEventListener`](https://github.com/infobip/infobip-rtc-android/wiki/RoomCall#set-event-listener)
method as shown in the following example:

```java
RoomCallEventListener roomCallEventListener = new RoomCallEventListener() {
    @Override
    public void onRoomJoined(RoomJoinedEvent roomJoinedEvent) {
        Log.d("Room", "Room joined!");
    }
    
    @Override
    public void onRoomLeft(RoomLeftEvent roomLeftEvent) {
        Log.d("Room", "Room left!");
    }
    
    @Override
    public void onError(ErrorEvent errorEvent) {
        Log.d("Room", "Oops, something went wrong! Message: " + errorEvent.getErrorCode().toString());
    }
    
    @Override
    public void onCameraVideoAdded(CameraVideoAddedEvent cameraVideoAddedEvent) {
        Log.d("Room", "Local camera video added!");
    }
    
    @Override
    public void onCameraVideoUpdated(CameraVideoUpdatedEvent cameraVideoUpdatedEvent) {
        Log.d("Room", "Local camera video updated!");
    }
    
    @Override
    public void onCameraVideoRemoved() {
        Log.d("Room", "Local camera video removed!");
    }
    
    @Override
    public void onScreenShareAdded(ScreenShareAddedEvent screenShareAddedEvent) {
        Log.d("Room", "Local screen share added!");
    }
    
    @Override
    public void onScreenShareRemoved() {
        Log.d("Room", "Local screen share removed!");
    }
    
    @Override
    public void onParticipantJoining(ParticipantJoiningEvent participantJoiningEvent) {
        Log.d("Room", "Participant joining!");
    }
    
    @Override
    public void onParticipantJoined(ParticipantJoinedEvent participantJoinedEvent) {
        Log.d("Room", "Participant joined!");
    }
    
    @Override
    public void onParticipantLeft(ParticipantLeftEvent participantLeftEvent) {
        Log.d("Room", "Participant left!");
    }
    
    @Override
    public void onParticipantCameraVideoAdded(ParticipantCameraVideoAddedEvent participantCameraVideoAddedEvent) {
        Log.d("Room", "Participant added camera video!");
    }
    
    @Override
    public void onParticipantCameraVideoRemoved(ParticipantCameraVideoRemovedEvent participantCameraVideoRemovedEvent) {
        Log.d("Room", "Participant removed camera video!");
    }
    
    @Override
    public void onParticipantScreenShareAdded(ParticipantScreenShareAddedEvent participantScreenShareAddedEvent) {
        Log.d("Room", "Participant added screen share!");
    }
    
    @Override
    public void onParticipantScreenShareRemoved(ParticipantScreenShareRemovedEvent participantScreenShareRemovedEvent) {
        Log.d("Room", "Participant removed screen share!");
    }
    
    @Override
    public void onParticipantMuted(ParticipantMutedEvent participantMutedEvent) {
        Log.d("Room", "Participant muted!");
    }
    
    @Override
    public void onParticipantUnmuted(ParticipantUnmutedEvent participantUnmutedEvent) {
        Log.d("Room", "Participant unmuted!");
    }
    
    @Override
    public void onParticipantDeaf(ParticipantDeafEvent participantDeafEvent) {
        Log.d("Room", "Participant deaf!");
    }
    
    @Override
    public void onParticipantUndeaf(ParticipantUndeafEvent participantUndeafEvent) {
        Log.d("Room", "Participant undeaf!");
    }
    
    @Override
    public void onParticipantStartedTalking(ParticipantStartedTalkingEvent participantStartedTalkingEvent) {
        Log.d("Room", "Participant started talking!");
    }
    
    @Override
    public void onParticipantStoppedTalking(ParticipantStoppedTalkingEvent participantStoppedTalkingEvent) {
        Log.d("Room", "Participant stopped talking!");
    }
};

roomCall.setEventListener(roomCallEventListener);
```

To get the current event listener, use the [`getEventListener`](https://github.com/infobip/infobip-rtc-android/wiki/RoomCall#get-event-listener) method:

```java
RoomCallEventListener roomCallEventListener = roomCall.getEventListener();
```
When event handlers are set up and the Room is joined, there are a few things that you can do with the actual call.
You can find the full list of actions available in the [`documentation`](https://github.com/infobip/infobip-rtc-android/wiki/RoomCall)

Leaving the room can be done via the [`leave`](https://github.com/infobip/infobip-rtc-android/wiki/RoomCall#leave) method.
On the side of the other participants, the [`ParticipantLeftEvent`](https://github.com/infobip/infobip-rtc-android/wiki/ParticipantLeftEvent) event will be fired upon leave completion.

```java
roomCall.leave();
```

During the room call, you can also mute (and unmute) your audio,
by calling the [`mute(shouldMute)`](https://github.com/infobip/infobip-rtc-android/wiki/RoomCall#mute) method in the following way:

```java
roomCall.mute(true);
```

On the side of the other participants, the [`ParticipantMutedEvent`](https://github.com/infobip/infobip-rtc-android/wiki/ParticipantMutedEvent) or
[`ParticipantUnmutedEvent`](https://github.com/infobip/infobip-rtc-android/wiki/ParticipantUnmutedEvent) event will be fired.

To check if the audio is muted, call the [`muted()`](https://github.com/infobip/infobip-rtc-android/wiki/RoomCall#muted) method in the following way:

```java
boolean isAudioMuted = roomCall.muted();
```

During the room call, you can play audio on the speakerphone,
by calling the [`speakerphone(enabled)`](https://github.com/infobip/infobip-rtc-android/wiki/RoomCall#speakerphone) method in the following way:

```java
roomCall.speakerphone(true);
```

To check if audio is on the speakerphone,
call the [`speakerphone()`](https://github.com/infobip/infobip-rtc-android/wiki/RoomCall#speakerphone) method in the following way:

```java
boolean isSpeakerphoneEnabled = roomCall.speakerphone();
```

In a Room call, multiple participants can turn their camera video on and off.  
In that case, whenever a participant turns their camera on, the `participantCameraVideoAddedEvent` is fired, and in order to handle the participant's video, You should set up the handler for this event in a similar fashion as the one for handling remote videos in WebRTC calls:

```java
protected void onCreate(Bundle savedInstanceState) {
    //...
    VideoRenderer participantVideoRenderer = findViewById(R.id.participant_video);
    remoteVideoRenderer.init();
    remoteVideoRenderer.setScalingType(RendererCommon.ScalingType.SCALE_ASPECT_FIT);
}

...

@Override
public void onParticipantCameraVideoAdded(ParticipantCameraVideoAddedEvent participantCameraVideoAddedEvent) {
    Log.d("WebRTC", "Participant camera video added!");

    VideoRenderer participantVideoRenderer = findViewById(R.id.remote_video);
    RTCVideoTrack participantRTCVideoTrack = participantCameraVideoAddedEvent.getVideoTrack();
    participantRTCVideoTrack.addSink(participantVideoRenderer);
}
```

Also, when a participant turns their camera off, the `participantCameraVideoRemovedEvent` is fired.

Participants can also turn their screen share on and off.
In that case, we listen for `participantScreenShareAddedEvent`, and handle it in a similar way as when the participant turns their camera on:

```java
@Override
public void onParticipantScreenShareAdded(ParticipantScreenShareAddedEvent participantScreenShareAddedEvent) {
    Log.d("WebRTC", "Participant screen share added!");

    VideoRenderer participantScreenShareRenderer = findViewById(R.id.participant_screen_share);
    RTCVideoTrack participantRTCScreenShareTrack = participantScreenShareAddedEvent.getVideoTrack();
    participantRTCScreenShareTrack.addSink(participantScreenShareRenderer);
}
```

When a participant turns their screen share off, the `participantScreenShareRemovedEvent` is fired.

To turn the camera on in a Room call, You can use the
[`cameraVideo`](https://github.com/infobip/infobip-rtc-andorid/wiki/RoomCall#camera-video) method.

```java
roomCall.cameraVideo(true);
```

To turn the screen share on in a Room call, You can use the
[`startScreenShare`](https://github.com/infobip/infobip-rtc-andorid/wiki/RoomCall#start-screen-share) method.

```java
roomCall.startScreenShare();
```

And to turn the screen share off in a Room call, You can use
[`stopScreenShare`](https://github.com/infobip/infobip-rtc-andorid/wiki/RoomCall#stop-screen-share) method.

```java
roomCall.stopScreenShare();
```

Handling local camera video and local screen share is done in the same way as described earlier for the WebRTC call.

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
