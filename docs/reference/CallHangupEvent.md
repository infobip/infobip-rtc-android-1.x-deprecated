* [`CallHangupEvent(ErrorCode errorCode)`](#CallHangupEvent)
* [`ErrorCode getErrorCode()`](#getErrorCode)

<a name="CallHangupEvent"></a>
## `CallHangupEvent(errorCode)`

### Description
Creates new instance of `CallHangupEvent`.

### Arguments
* `errorCode`: [*ErrorCode*](./ErrorCode.md) - Error code that describes why call hung up.

### Returns
[`CallHangupEvent`](./CallHangupEvent.md) - Created instance.

<a name="getErrorCode"></a>
## `getErrorCode()`

### Description
Getter for `errorCode` field.

### Arguments
none

### Returns
[`ErrorCode`](./ErrorCode.md) - Value of `errorCode` field.