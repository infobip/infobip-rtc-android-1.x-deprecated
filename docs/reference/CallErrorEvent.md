* [`CallErrorEvent(ErrorCode errorCode)`](#CallErrorEvent)
* [`ErrorCode getReason()`](#getReason)

<a name="CallErrorEvent"></a>
## `CallErrorEvent(errorCode)`

### Description
Creates new instance of `CallErrorEvent`.

### Arguments
* `errorCode`: [*ErrorCode*](./ErrorCode.md) - Error code that describes what caused call to fail.

### Returns
[`CallErrorEvent`](./CallErrorEvent.md) - Created instance.

<a name="getReason"></a>
## `getReason()`

### Description
Getter for `errorCode` field.

### Arguments
none

### Returns
[`ErrorCode`](./ErrorCode.md) - Value of `errorCode` field.