# C++ Event Loop Provider
C++ library providing the functionality of an event loop to process instantaneous and scheduled events to be used by event driven applications  
The event loop processes events to notify registered receivers optionally with data throughout the current user session   
You may use the event loop to block the current thread for event processing or as independent thread leaving the current thread free for other tasks  

### 1. Blocking event loop `EventLoop::Mode::BLOCK`
This mode of event loop will block the current thread (usually main thread) where the `EventLoop::Run()` call has been issued  
The current thread will then continue processing events as long as the loop is running   

### 2. Non-blocking event loop `EventLoop::Mode::NON_BLOCK`
This mode will run the event loop on another thread which will prevent the current thread where the `EventLoop::Run()` call has been issued from getting blocked  
This can prove useful where the current thread has to either get to some execution after starting the event loop on the same thread or if it already uses a blocking event loop of its own. An example of the latter can be the Qt framework's `QGuiApplication::exec()` call which blocks the main thread and starts processing events  
That raises an important question: *Why would I need a secondary event loop when a primary one like Qt's `exec()` already exists?*

### `EventLoop` value proposition 
I don't know about you but I badly missed some features in Qt's event loop system like the ability to broadcast a signal to all receivers who have registered/subscribed to it. This was my primary motivating factor behind developing this event loop from scratch, besides my insatiable curiosity for how things work and an itch for dogfooding!
Broadcasting is quintessential when you want multiple components of the application to act upon a single stimulus without having to connect each receiver with the sender explicity. Good luck passing around sender instances to multiple recievers and creating a signal slot connection for each to enable broadcast reception in Qt!    
Another solid intention was to create an event driven system which is completely application agnostic in which it runs independently without needing any connections or objects to receive or trigger an event. Thus, `EventLoop` is a standalone static class with static methods enabling the developer to issue any API call from absolutely any thread, anywhere in your code. You just need to register a callback with the `EventLoop::RegisterEvent()` call, start the event loop with `EventLoop::Run()` and you are good to go!

## Detailed Usage
As a user, you can dynamically link the event loop library to your application and include the `EventLoop.hpp` header in your application code to access the library methods. Since we are using `Event` as a custom type to deliver and receive events with name and data, `Event.hpp` header will be required in the source files fetching those details from an incoming event  

**Note:** DO NOT include any other headers from the *include* directory of this project apart from ones mentioned above, while using the library in your application  

Detailed API documentation can be found in the `EventLoop.hpp` header. The following steps demonstrate possible usage of event loop in your application:  
- **[Optional step]** In the `main()` function of your program, call `EventLoop::SetMode()` method if you want the loop to be non-blocking. No need to call this explicitly for blocking mode because that's the default mode  
- **[Mandatory step]** Call the `EventLoop::Run()` method in `main()` where you wish the event loop to start and/or block  
- Register callbacks accepting `Event*` in any class or source file with `EventLoop::RegisterEvent()` where you wish to get notified for an event. Usually registrations are done in class constructors with either lambdas or class member functions as handlers  
- Trigger events anytime in any thread of the application with `EventLoop::TriggerEvent()` and the corresponding handlers which registered for the particular event name will be invoked either instantenously or after a timeout depending on the type of overload used  
- Retrieve information from a received event in a registered handler like name and data using `Event` type's `Event::getName()` and `Event::getData()` methods  
- **[Optional but highly recommended step]** Enable the event loop to exit gracefully through the explicit call of `EventLoop::Halt()` method. This method can be called from any thread in either modes. If called inside a registered handler, it will take effect only on completion of the handler scope  

**Tip:** If you're planning to use `EventLoop` on top of Qt's `exec()` event loop, set up non-blocking `EventLoop::Run()` before the `exec()` call and `EventLoop::Halt()` to be invoked on the `QGuiApplication::aboutToQuit` signal as follows for a graceful exit
```
QObject::connect(&app, &QGuiApplication::aboutToQuit, []{ EventLoop::Halt(); });
```
**Note:** Any code just after a blocking `EventLoop::Run()` call on the same thread will not execute until the event loop is halted because `EventLoop::Run()` blocks the current thread to process events by design.  

Shopping cart app (https://github.com/amoldhamale1105/ShoppingCart) serves as a usage reference of this library and API. In case of any questions or clarifications, you can reach out to me at amoldhamale1105@gmail.com  

## Build instructions
Check for latest stable releases of the library under `Releases` but if you wish to build a library from source with the current source code version, run the `build.sh` script  

The script can be run without any options. Defaults for each option will appear in `{}` in the usage instruction. Learn about the script usage by running the following command
```
./build.sh -h
```
As an example, if you want to use the script to build for release mode with `Unix Makefiles` cmake generator, the script can be executed as follows
```
./build.sh -a -r -g "Unix Makefiles"
```
Build artifacts will be generated in the `build` directory  
Output artifact will be present in `lib` directory as `libEventLoop.so` which can be further linked or moved to your target destination  

## Contribution
I don't want to sound cliche but this was a hobby project conceived out of curiousity and exasperation of missing features in popular existing event loop implementations like mentioned above.  
Initially it seemed insignificant but when I saw the solution's immense potential in event driven systems and state machines which could use custom event loops, I decided to open it to the public domain for use, distribution and contribution. I consider myself a 'good' programmer but definitely not an 'expert' or 'perfect' one which always leaves room for improvements and an opportunity to learn from other programmers and users. I invite you to contribute to this library implementation with your valuable insights for the benefit of everyone since this is open source software.  
Feel free to open pull requests, raise issues or contact me at amoldhamale1105@gmail.com for any ideas of modifications or feature additions  
This library is licenced under LGPL v2.1 instead of any other fully permissive license because I care a about sharing improvements and making it available publicly. I would love to have work based on this library accessible to everyone and at the same time want people to be able to use the shared library without modifications for their own non-LGPL projects, without the obligation of releasing their work under the same license  