# `extends` [`Call`](./Call)

* [`void accept()`](#accept)
* [`void decline()`](#decline)
* [`String source()`](#source)

<a name="hangup"></a>
## `accept()`

### Description
Accepts incoming call, which ends up in both parties receiving established event, after accept is processed by Infobip WebRTC platform.

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
    call.accept();
  }
  ...
});
```

<a name="decline"></a>
## `decline()`

### Description
Declines incoming call, which ends up in both parties receiving hangup event with proper status, after decline is processed by Infobip WebRTC platform.

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
    call.decline();
  }
  ...
});
```

<a name="source"></a>
## `source()`

### Description
Returns identity of user who originated incoming call.

### Arguments
none

### Returns
[`String`](https://developer.android.com/reference/java/lang/String) - Caller's identity.

### Example
```
InfobipRTC infobipRTC = new DefaultInfobipRTC(
    "2e29c3a0-730a-4526-93ce-cda44556dab5",
    RTCOptions.builder().build(),
    getApplicationContext()
);

infobipRTC.addIncomingCallEventListener(new IncomingCallEventListener() {
    @Override
    public void onIncomingCall(IncomingCallEvent incomingCallEvent) {
        String source = incomingCallEvent.getIncomingCall().source();
        Toast.makeText(getApplicationContext(), "Incoming call from " + source, Toast.LENGTH_LONG).show();
    }

    ...
});

infobipRTC.connect();
```