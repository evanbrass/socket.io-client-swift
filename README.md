Socket.IO-Client-Swift
======================

Socket.IO-client for Swift. Supports ws/wss connections and binary

Installation
============
1. Requires linking [SocketRocket](https://github.com/square/SocketRocket) against your xcode project. (Be sure to link the [frameworks](https://github.com/square/SocketRocket#framework-dependencies) required by SocketRocket)
2. Create a bridging header for SocketRocket
3. Copy the SwiftIO folder into your xcode project

Use
===
Methods
-------
1. `socket.on(name:String, callback:((data:AnyObject?) -> Void))` - Adds a handler for an event.
2. `socket.onMultipleArgs(name:String, callback:((data:[AnyObject]) -> Void))` - Adds a handler for an event that           can have multiple items. Items are stored in an array.
3. `socket.emit(event:String, args:AnyObject? = nil)` - Sends an event with optional arg.
4. `socket.emitMultiple(event:String, args:AnyObject...)` - Sends an event with multiple args.
5. `socket.connect()` - Establishes a connection to the server. A "connect" event is fired upon successful connection.
6. `socket.close()` - Closes the socket. Once a socket is closed it should not be reopened. 

```swift
// opts can be omitted, will use default values
let socket = SocketIOClient(socketURL: "https://localhost:8080", opts: [
    "reconnects": true, // default true
    "reconnectAttempts": 5, // default -1 (infinite tries)
    "reconnectWait": 5 // default 10
])

socket.on("connect") {data in
    println("socket connected")
    socket.emit("testEcho")
    socket.emit("testObject", args: [
        "data": true
        ])
    socket.emit("arrayTest", args: [1, true, "test", ["test": "test"], data, data])
    socket.emit("stringTest", args: "stringTest")
    socket.emit("intTest", args: 1)
    socket.emit("doubleTest", args: 2.3)
    socket.emit("boolTest", args: true)
    socket.emit("dataTest", args: data)

    // Sending multiple args
    socket.emitMultiple("multTest", args: [data], 1.4, 1, "true", 
        true, ["test": data], data)
}

socket.on("disconnect") {data in
    if let reason = data as? String {
        println("Socket disconnected: \(reason)")
    }
}

socket.on("reconnect") {data in
    if let reason = data as? String {
        println("Socket reconnecting: \(reason)")
    }
}

socket.on("reconnectAttempt") {data in
    if let triesLeft = data as? Int {
        println(triesLeft)
    }
}

socket.on("jsonTest") {data in
    if let json = data as? NSDictionary {
       println(json["test"]!) // foo bar
    }
}

socket.on("boolTest") {data in
    if let bool = data as? Bool {
        println(bool) // true
    }
}

socket.on("arrayTest") {data in
    if let array = data as? NSArray {
        println(array[0]) // 2
        println(array[1]) // "test"
    }
}

socket.on("intTest") {data in
    if let intData = data as? Int {
        println(intData)
    }
}

// Messages that have multiple items are passed
// by an array
socket.onMultipleArgs("multipleItems") {datas in
    if let str = datas[0] as? String {
        println(str)
    }

    if let arr = datas[1] as? [Int] {
        println(arr)
    }

    if let obj = datas[4] as? NSDictionary {
        println(obj["test"])
    }
}
        
// Recieving binary
socket.on("dataTest") {data in
    if let data = data as? NSData {
        println("data is binary")
    }
}

socket.on("objectDataTest") {data in
    if let dict = data as? NSDictionary {
        if let data = dict["data"] as? NSData {
            let string = NSString(data: data, encoding: NSUTF8StringEncoding)
            println("Got data: \(string!)")
        }
    }
}

socket.connect()

// Sending binary
socket.emit("testData", args: [
        "data": "Hello World".dataUsingEncoding(NSUTF8StringEncoding,
            allowLossyConversion: false)!,
        "test": true])
```
License
=======
MIT
