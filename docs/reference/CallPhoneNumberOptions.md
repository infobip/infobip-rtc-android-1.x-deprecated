* [`static CallPhoneNumberOptions.Builder builder()`](#builder)
* [`String getFrom()`](#getFrom)

<a name="builder"></a>
## `static builder()`

### Description
Creates builder instance used to build new instance of `CallPhoneNumberOptions`.

### Arguments
none

### Returns
[`CallPhoneNumberOptions.Builder`](./CallPhoneNumberOptions.md) - instance of builder.

### Example
```
CallPhoneNumberOptions.Builder builder = CallPhoneNumberOptions.builder();
CallPhoneNumberOptions callPhoneNumberOptions = builder.from("41793026731").build();
```

<a name="getFrom"></a>
## `getFrom()`

### Description
Getter for `from` field.

### Arguments
none

### Returns
[`String`](https://developer.android.com/reference/java/lang/String) - value of `from` field.

### Example
```
String from = CallPhoneNumberOptions.builder().from("41793026731").build().getFrom();
```