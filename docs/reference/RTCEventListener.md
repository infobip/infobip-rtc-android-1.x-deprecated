* [`void onConnected(ConnectedEvent connectedEvent)`](#onConnected)
* [`void onDisconnected(DisconnectedEvent disconnectedEvent)`](#onDisconnected)
* [`void onReconnecting(ReconnectingEvent reconnectingEvent)`](#onReconnecting)
* [`void onReconnected(ReconnectedEvent reconnectedEvent)`](#onReconnected)

<a name="onConnected"></a>
## `onConnected(connectedEvent)`

### Description
Method that fires when connection to Infobip WebRTC platform is successful.

### Arguments
* `connectedEvent`: [*ConnectedEvent*](./ConnectedEvent.md) - Event instance with all relevant data.

### Returns
none

<a name="onDisconnected"></a>
## `onDisconnected(disconnectedEvent)`

### Description
Method that fires when connection to Infobip WebRTC platform is lost completely.

### Arguments
* `disconnectedEvent`: [*DisconnectedEvent*](./DisconnectedEvent.md) - Event instance with all relevant data.

### Returns
none

<a name="onReconnecting"></a>
## `onReconnecting(reconnectingEvent)`

### Description
Method that fires when connection to Infobip WebRTC platform is lost, and SDK tries to reconnect back.

### Arguments
* `reconnectingEvent`: [*ReconnectingEvent*](./ReconnectingEvent.md) - Event instance with all relevant data.

### Returns
none

<a name="onReconnected"></a>
## `onReconnected(reconnectedEvent)`

### Description
Method that fires when connection to Infobip WebRTC platform is lost but then retried and connected again successfully.

### Arguments
* `reconnectedEvent`: [*ReconnectedEvent*](./ReconnectedEvent.md) - Event instance with all relevant data.

### Returns
none