* [`void hangup()`](#hangup)
* [`void addEventListener(CallEventListener callEventListener)`](#addEventListener)
* [`void mute(boolean shouldMute)`](#mute)
* [`void sendDTMF(String dtmf) throws ActionFailedException`](#sendDTMF)

<a name="hangup"></a>
## `hangup()`

### Description
Hangs up call, which ends up in both parties receiving hangup event, after hangup is processed by Infobip WebRTC platform.

### Arguments
none

### Returns
none

### Example
```
InfobipRTC infobipRTC = new DefaultInfobipRTC(
    "2e29c3a0-730a-4526-93ce-cda44556dab5",
    RTCOptions.builder().build(),
    getApplicationContext()
);
infobipRTC.connect();

OutgoingCall call = infobipRTC.call("Alice", CallOptions.builder().build());
call.addEventListener(new CallEventListener() {
    @Override
    public void onEstablished(CallEstablishedEvent callEstablishedEvent) {
        call.hangup();
    }
    ...
});
```

<a name="addEventListener"></a>
## `addEventListener(callEventListener)`

### Description
Configures event handler for call events. There are 2 event types in Infobip RTC API, ones that are configured globally, per connection basis (RTC events), and ones that are configured per call basis (call events). This one handles call events.

### Arguments
* `callEventListener`: [*CallEventListener*](./CallEventListener.md) - Interface with event methods that should be implemented, method per call event to be handled.

### Returns
none

### Example
```
InfobipRTC infobipRTC = new DefaultInfobipRTC(
    "2e29c3a0-730a-4526-93ce-cda44556dab5",
    RTCOptions.builder().build(),
    getApplicationContext()
);
infobipRTC.connect();

OutgoingCall call = infobipRTC.call("Alice", CallOptions.builder().build());
call.addEventListener(new CallEventListener() {
    @Override
    public void onEstablished(CallEstablishedEvent callEstablishedEvent) {
        Toast.makeText(getApplicationContext(), "Established!", Toast.LENGTH_LONG);
    }

    @Override
    public void onHangup(CallHangupEvent callHangupEvent) {
        Toast.makeText(getApplicationContext(), "Hangup!", Toast.LENGTH_LONG);
    }

    @Override
    public void onError(CallErrorEvent callErrorEvent) {
        Toast.makeText(getApplicationContext(), "Error!", Toast.LENGTH_LONG);
    }

    @Override
    public void onRinging(CallRingingEvent callRingingEvent) {
        Toast.makeText(getApplicationContext(), "Ringing!", Toast.LENGTH_LONG);
    }
});
```

<a name="mute"></a>
## `mute(shouldMute)`

### Description
Toggles mute option.

### Arguments
`shouldMute`: *boolean* - Whether call should be muted after action or not.

### Returns
none

### Example
```
InfobipRTC infobipRTC = new DefaultInfobipRTC(
    "2e29c3a0-730a-4526-93ce-cda44556dab5",
    RTCOptions.builder().build(),
    getApplicationContext()
);
infobipRTC.connect();

OutgoingCall call = infobipRTC.call("Alice", CallOptions.builder().build());
call.addEventListener(new CallEventListener() {
    @Override
    public void onEstablished(CallEstablishedEvent callEstablishedEvent) {
        call.mute(true);
    }
    ...
});
```

<a name="sendDTMF"></a>
## `sendDTMF(dtmf) throws ActionFailedException`

### Description
Simulates key-press by sending DTMF (Dual-Tone Multi-Frequency) entry.

### Arguments
* `dtmf`: [*String*](https://developer.android.com/reference/java/lang/String) - One of the allowed DTMF characters (digits from `0` to `9`, `*` and `#`).

### Returns
none

### Throws
`ActionFailedException` - if sending fails for any reason.

### Example
```
InfobipRTC infobipRTC = new DefaultInfobipRTC(
    "2e29c3a0-730a-4526-93ce-cda44556dab5",
    RTCOptions.builder().build(),
    getApplicationContext()
);
infobipRTC.connect();

OutgoingCall call = infobipRTC.call("Alice", CallOptions.builder().build());
call.addEventListener(new CallEventListener() {
    @Override
    public void onEstablished(CallEstablishedEvent callEstablishedEvent) {
        call.sendDTMF("1");
    }
    ...
});
```