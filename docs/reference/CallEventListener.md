* [`void onEstablished(CallEstablishedEvent callEstablishedEvent)`](#onEstablished)
* [`void onHangup(CallHangupEvent callHangupEvent)`](#onHangup)
* [`void onError(CallErrorEvent callErrorEvent)`](#onError)
* [`void onRinging(CallRingingEvent callRingingEvent)`](#onRinging)

<a name="onEstablished"></a>
## `onEstablished(callEstablishedEvent)`

### Description
Method that fires when call between two parties is established and media starts to flow.

### Arguments
* `callEstablishedEvent`: [*CallEstablishedEvent*](./CallEstablishedEvent.md) - Event instance with all relevant data.

### Returns
none

<a name="onHangup"></a>
## `onHangup(callHangupEvent)`

### Description
Method that fires when call that was established hangs up successfully.

### Arguments
* `callHangupEvent`: [*CallHangupEvent*](./CallHangupEvent.md) - Event instance with all relevant data.

### Returns
none

<a name="onError"></a>
## `onError(callErrorEvent)`

### Description
Method that fires when call fails with an error before it was established.

### Arguments
* `callErrorEvent`: [*CallErrorEvent*](./CallErrorEvent.md) - Event instance with all relevant data.

### Returns
none

<a name="onRinging"></a>
## `onRinging(callRingingEvent)`

### Description
Method that fires when outgoing call starts ringing at other party's device.

### Arguments
* `callRingingEvent`: [*CallRingingEvent*](./CallRingingEvent.md) - Event instance with all relevant data.

### Returns
none