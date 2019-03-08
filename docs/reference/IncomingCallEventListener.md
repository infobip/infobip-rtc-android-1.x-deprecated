* [`void onIncomingCall(IncomingCallEvent incomingCallEvent)`](#onIncomingCall)
* [`void onMissedCall(MissedCallEvent missedCallEvent)`](#onMissedCall)

<a name="onIncomingCall"></a>
## `onIncomingCall(incomingCallEvent)`

### Description
Method that fires when new incoming call is received.

### Arguments
* `incomingCallEvent`: [*IncomingCallEvent*](./IncomingCallEvent.md) - Event instance with all relevant data.

### Returns
none

<a name="onMissedCall"></a>
## `onMissedCall(missedCallEvent)`

### Description
Method that fires when user is in the middle of call and someone else is trying to establish call.

### Arguments
* `missedCallEvent`: [*MissedCallEvent*](./MissedCallEvent.md) - Event instance with all relevant data.

### Returns
none