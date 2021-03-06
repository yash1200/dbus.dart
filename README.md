[![Pub Package](https://img.shields.io/pub/v/dbus.svg)](https://pub.dev/packages/dbus)

A native Dart client implementation of [D-Bus](https://www.freedesktop.org/wiki/Software/dbus/).

## Accessing a remote object using *dart-dbus*

The easist way to get started is to use *dart-dbus* to generate Dart objects to access existing D-Bus services.
Start with a D-Bus interface definition:

```xml
<node name="/org/freedesktop/hostname1">
  <interface name="org.freedesktop.hostname1">
    <property name="Hostname" type="s" access="read"/>
  </interface>
</node>
```

The *dart-dbus* tool processes this interface to generate a Dart source file:

```shell
$ dart-dbus generate-remote-object hostname1.xml -o hostname1.dart
```

You can then use the generated `hostname1.dart` to access that remote
object:

```dart
import 'package:dbus/dbus.dart';
import 'hostname1.dart';

var client = DBusClient.system();
var hostname1 = OrgFreeDesktopHostname1(client, 'org.freedesktop.hostname1');
var hostname = await hostname1.getHostname();
print('hostname: ${hostname}')
await client.close();
```

The code generated by *dart-dbus* may not be the cleanest API you want
to expose for your service. It is recommended that you use the
generated code as a starting point and then modifiy it as necessary.

## Accessing a remote object manually

You can access remote objects without using *dart-dbus* if you want.
This requires you to handle error cases yourself.
The equivalent of the above example is:

```dart
import 'package:dbus/dbus.dart';

var client = DBusClient.system();
var object = DBusRemoteObject(client, 'org.freedesktop.hostname1', DBusObjectPath('/org/freedesktop/hostname1'));
var hostname = await object.getProperty('org.freedesktop.hostname1', 'Hostname');
print('hostname: ${hostname.toNative()}');
await client.close();
```

## Exporting an object on the bus

```dart
import 'package:dbus/dbus.dart';

class TestObject extends DBusObject {
  @override
  DBusObjectPath get path {
    return DBusObjectPath('/com/example/Test');
  }

  @override
  Future<MethodResponse> handleMethodCall(String sender, String interface, String member, List<DBusValue> values) async {
    if (interface == 'com.example.Test') {
      if (member == 'Test') {
        return DBusMethodSuccessResponse([DBusString('Hello World!')]);
      } else {
        return DBusMethodErrorResponse.unknownMethod();
      }
    } else {
      return DBusMethodErrorResponse.unknownInterface();
    }
  }
}

var client = DBusClient.session();
client.registerObject(TestObject());
```

## Contributing to dbus.dart

We welcome contributions! See the [contribution guide](CONTRIBUTING.md) for more details.
