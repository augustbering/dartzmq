[![Discord](https://img.shields.io/discord/781219798931603527.svg?label=enwi&logo=discord&logoColor=ffffff&color=7389D8&labelColor=6A7EC2)](https://discord.gg/YxVyJWX62h)
[![GitHub tag (latest by date)](https://img.shields.io/github/v/tag/enwi/dartzmq?label=release)](https://github.com/enwi/dartzmq/releases)

# dartzmq
A simple dart zeromq implementation/wrapper around the libzmq C++ library


## Features
Currently supported:
- Creating sockets (`pair`,  `pub`,  `sub`,  `req`,  `rep`,  `dealer`,  `router`,  `pull`,  `push`,  `xPub`,  `xSub`,  `stream`)
    - Depending on the zeromq library also `server`, `client`, `radio`, `dish`, `channel`, `peer`, `raw`, `scatter` and `gather`
- Sending messages (of type `List<int>`, `String`, `ZFrame` or `ZMessage`)
- Bind (`bind(String address)`)
- Connect (`connect(String address)`)
- Curve (`setCurvePublicKey(String key)`, `setCurveSecretKey(String key)` and `setCurveServerKey(String key)`)
- Socket options (`setOption(int option, String value)`)
- Receiving multipart messages (`ZMessage`)
- Topic subscription for `sub` sockets (`subscribe(String topic)` and `unsubscribe(String topic)`)
- Monitoring socket states (`ZMonitor` and `MonitoredZSocket`)
- Asynchronous (`ZSocket`) and Synchronous (`ZSyncSocket`) sockets


## Getting started
Currently Windows and Android are officially supported as platforms.
Basically there a no extra setup steps unless the provided zeromq binary does not work for your platform.
I have tested this on Windows and Android, which both work. 
Other platforms have not been tested, but should work. 
If you have tested this plugin on another platform and got it to work, please share your steps and create an issue so I can add it to the list below.

## Usage
Create context
```dart
final ZContext context = ZContext();
```

Create asynchronous socket
```dart
final ZSocket socket = context.createSocket(SocketType.req);
```

Create synchronous socket
```dart
final ZSyncSocket socket = context.createSocket(SocketType.req);
```

Connect socket
```dart
socket.connect("tcp://localhost:5566");
```

Send message
```dart
socket.send([1, 2, 3, 4, 5]);

socket.sendString('My Message');

socket.sendFrame(ZFrame([1, 2, 3, 4, 5]));

var message = ZMessage();
message.add(ZFrame([1, 2, 3, 4, 5]));
message.add(ZFrame([6, 7, 8, 9, 10]));
socket.sendMessage(message, flags: ZMQ_DONTWAIT);
```

Receive `ZMessage`s
```dart
socket.messages.listen((message) {
    // Do something with message
});
```

Receive `ZFrame`s
```dart
socket.frames.listen((frame) {
    // Do something with frame
});
```


Receive payloads (`Uint8List`)
```dart
socket.payloads.listen((payload) {
    // Do something with payload
});
```

> NOTE: If you're trying to use req/rep pattern, consider using either a `ZSyncSocket` and call `socket.recv()` or a `ZSocket` of type `SocketType.dealer` instead.
> For the latter you need to add an empty ZFrame at the beginning of your ZMessage as identification.

Receive socket events
```dart
final MonitoredZSocket socket = context.createMonitoredSocket(SocketType.req);
socket.events.listen((event) {
    log('Received event ${event.event} with value ${event.value}');
});
```
```dart
final ZSocket socket = context.createSocket(SocketType.req);
final ZMonitor monitor = ZMonitor(
    context: context,
    socket: socket,
    event: ZMQ_EVENT_CONNECTED | ZMQ_EVENT_CLOSED, // Only listen for connected and closed events
);
monitor.events.listen((event) {
    log('Received event ${event.event} with value ${event.value}');
});
```

Destroy socket
```dart
socket.close();
```

Destroy monitor
```dart
monitor.close();
```

Destroy context
```dart
context.stop();
```

### Using a custom `libzmq` binary

#### Windows
Place a shared library of `libzmq` next to your executable (for example place `libzmq-v142-mt-4_3_5.dll` in the folder `yourproject/build/windows/runner/Debug/`)

> You can use a shared library of `libzmq` or compile it yourself. 
> Especially when using this on windows you need to make sure that `libzmq` is compiled using `MSVC-2019` if you are using `clang` it will not work ([more info](https://flutterforum.co/t/windows-desktop-flutter-ffi-and-loading-the-clang-library/3842))

#### Android
> Note that you need to use Android NDK version r21d, 23 or 24. Newer versions might not work as expected (see https://github.com/zeromq/libzmq/issues/4276)

1. Follow [these steps](https://github.com/zeromq/libzmq/tree/master/builds/android) to build a `libzmq.so` for different platforms
   - If you need `curve` support make sure to set the environment variable `CURVE` either to `export CURVE=libsodium` or `export CURVE=tweetnacl` before running the build command
2. Include these in your project following [these steps](https://github.com/truongsinh/flutter-ffi-samples/blob/master/packages/sqlite/docs/android.md#update-gradle-script)
3. Include the compiled standard c++ library `libc++_shared.so` files located inside the Android NDK as in step 2 ([reference](https://developer.android.com/ndk/guides/cpp-support#cs))
   - You can find these inside the Android NDK for example under this path `ndk\21.4.7075529\toolchains\llvm\prebuilt\windows-x86_64\sysroot\usr\lib`

#### IOS
> Note IOS arm64 architecture supported so it will run from iPhone5s to the latest Iphone, Ipads devices
1. To make it Run on simulator you have to modify the podspec file in the ios directory of plugin
2. modify this   - s.pod_target_xcconfig = { "OTHER_LDFLAGS" => "$(inherited) -force_load $(PODS_TARGET_SRCROOT)/Frameworks/libzmq.a -lstdc++" } 
  - From
  s.pod_target_xcconfig = { "OTHER_LDFLAGS" => "$(inherited) -force_load $(PODS_TARGET_SRCROOT)/Frameworks/libzmq.a -lstdc++" }
  - To
    s.pod_target_xcconfig = { "OTHER_LDFLAGS" => "$(inherited) -force_load $(PODS_TARGET_SRCROOT)/Frameworks/libzmq_simulator.a -lstdc++" }

<!-- ## Additional information
Tell users more about the package: where to find more information, how to 
contribute to the package, how to file issues, what response they can expect 
from the package authors, and more. -->
