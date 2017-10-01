+++
title = "Building a MUD with F# and Akka.NET - Part One"
date = "2017-03-12"
tags = ["fsharp", "akka.net"]
aliases = ["/building-a-mud-with-f-sharp-and-akka-net-part-one"]

[extra]
github = "https://github.com/17cupsofcoffee/AkkaMUD"
+++

I feel like no matter how many languages I try, [I always keep getting drawn back to F#](https://www.youtube.com/watch?v=UPw-3e_pzqU). It's got just about everything I love about functional languages like Elm, Haskell and OCaml, while still giving me access to the mountain of great open source libraries that are available in the .NET ecosystem.

This week, I've been taking a look at [Akka.NET](http://getakka.net/), an unofficial (but extremely polished) port of the popular Java/Scala actor framework. It's primarily designed for C#, but they provide an add-on package that exposes a really nice, idiomatic F# API. 

As a little exercise for myself (and so my new blog wouldn't be quite so empty!), I thought it'd be fun to try to write a [MUD](https://en.wikipedia.org/wiki/MUD) using this API. Given that there's quite a lot of [prior art](http://www.kotancode.com/2012/01/29/building-a-mud-in-scala-revisited/) for this, it shouldn't be too hard to get started with, but time will tell if this is a little over-ambitious for a beginner project.

If you're following along at home, make sure you have the [Akka](https://www.nuget.org/packages/Akka) and [Akka.FSharp](https://www.nuget.org/packages/Akka.FSharp) packages installed into your project using Nuget or Paket. I'll be [committing to GitHub](https://github.com/17cupsofcoffee/AkkaMUD) as I go.
## The Basics

<small>[<i class="fa fa-github" aria-hidden="true"></i> View the code for this section](https://github.com/17cupsofcoffee/AkkaMUD/blob/part1a-the-basics/AkkaMUD/Program.fs)</small>

To begin with, let's take a quick look at how to use the Akka.NET F# API by creating a simple actor - all it'll do is say hello or goodbye when it receives a message. If you're already familiar with how to do this, feel free to [skip ahead to the next section](#networkio)!

First, we need to create a 'system' - I'm not going to go into too much detail on the actual concepts behind the Akka actor model here ([the docs](http://getakka.net/docs/)  explain it much better than I probably would), but at the most simple level, the system is the structure which co-ordinates all of your actors, managing the thread pool that allows them to do their work.

```fs
let system = System.create "system" (Configuration.defaultConfig())
```

Next, we'll define the type of message that our actor will respond to. F#'s discriminated unions are perfect for this:
```fs
type GreeterMsg =
    | Hello of string
    | Goodbye of string
```

Now here's the fun part - let's define our actor, and spawn it into our system. If you're coming from the C# or Java versions of Akka, you may be used to defining actors as classes - it's certainly possible to do things that way in F# too, but we can be much more functional:

```fs
let greeter = spawn system "greeter" <| fun mailbox ->
    let rec loop() = actor {
        let! msg = mailbox.Receive()

        match msg with
        | Hello name -> printf "Hello, %s!\n" name
        | Goodbye name -> printf "Goodbye, %s!\n" name

        return! loop()
    }
    loop()
```

Most of this code will seem familiar if you've written much F# - we define an anonymous function that takes in the actor's mailbox and uses tail recursion to loop, then pass it into the spawning function to create a new instance of the actor. But wait - if this just loops forever, how does it not hang the program?

This is where the [actor computation expression](http://getakka.net/docs/FSharp%20API#creating-actors-with-actor-computation-expression) comes in - `loop` may look like a normal synchronous function, but the `actor` wrapper converts it into a [continuation-based](https://en.wikipedia.org/wiki/Continuation) form that can be suspended when no messages need to be processed. This is similar in nature to the async expressions in the F# standard library, or async/await in C#.

Let's test this out! To send a message to a actor in Akka.NET, you use the `.Tell` method - however, as this is such a common operation, the F# API provides the handy `<!` operator, which  does the same thing.

```fs
greeter <! Hello "Joe"       // or greeter.Tell(Hello "Joe") 
greeter <! Goodbye "Joe"     // or greeter.Tell(Goodbye "Joe") 

System.Console.ReadLine() |> ignore
```

Note that we have to call a blocking function like `ReadLine` at the end of our program - otherwise it will terminate before our actor gets a chance to respond! There's almost certainly more elegant ways of stopping this from happening, but this is the simplest for now.

All being well, you should get output along the lines of this:

```bash
Hello, Joe!
Goodbye, Joe!
```

## Network I/O

Okay, that's pretty cool, but it's not exactly a MUD, is it? Let's make a start on hooking our actor up to the network. Akka provides an I/O API out of the box; to hook into it, we'll need to define two new actors.

### The Server

<small>[<i class="fa fa-github" aria-hidden="true"></i> View the code for this section](https://github.com/17cupsofcoffee/AkkaMUD/blob/part1b-the-server/AkkaMUD/Program.fs)</small>

Let's define another simple actor below our greeter. This time, however, we'll add some code to be run when it gets initialised:

```fs
let listener = spawn system "listener" <| fun mailbox ->
    let rec loop() = actor {
        let! msg = mailbox.Receive()
        
        return! loop()
    }

    mailbox.Context.System.Tcp() <! Tcp.Bind(mailbox.Self, IPEndPoint(IPAddress.Any, 9090))
    loop()
```

We send a message to the built-in TCP manager, passing it a reference to our actor and a port to bind to. When it's done, it'll reply with a message of its own - so let's get our actor to handle it.

Unfortunately, this is where Akka.NET's C# roots start to show a little bit - rather than using a single discriminated union type for its messages, the TCP manager has a separate class for each. We can work around this without making our code too ugly by using the [type test pattern](https://docs.microsoft.com/en-us/dotnet/articles/fsharp/language-reference/pattern-matching#type-test-pattern) syntax, which allows you to match against types instead of values at runtime.

```fs
let server = spawn system "listener" <| fun (mailbox: Actor<obj>) ->
    let rec loop() = actor {
        let! msg = mailbox.Receive()
        
        match msg with
        | :? Tcp.Bound as bound ->
            printf "Listening on %O\n" bound.LocalAddress
        | _ -> mailbox.Unhandled()

        return! loop()
    }

    mailbox.Context.System.Tcp() <! Tcp.Bind(mailbox.Self, IPEndPoint(IPAddress.Any, 9090))
    loop()
```

Note that we have to provide a type annotation for the mailbox, otherwise we get a compile error. Also, since our match expression isn't exhaustive, it's good practice to have a catch-all arm that signals if a message wasn't handled. We'll be able to log this out later.

If you run the program now, you should see the socket get bound successfully! Next, we need to define what happens when someone connects.

### The Handler

<small>[<i class="fa fa-github" aria-hidden="true"></i> View the code for this section](https://github.com/17cupsofcoffee/AkkaMUD/blob/part1c-the-handler/AkkaMUD/Program.fs)</small>

Above the server actor, let's start to define our connection handler.

```fs
let handler connection (mailbox: Actor<obj>) =
    let rec loop connection = actor {
        let! msg = mailbox.Receive()

        match msg with
        | :? Tcp.Received as received -> ()
        | _ -> mailbox.Unhandled()

        return! loop connection
    }

    loop connection
```

There's a couple of things to note here:

* We're not spawning this actor yet - we're going to have our server actor make a copy as a child once for every connection.
* Rather than storing the connection as mutable state, we thread it through the loop. You could use a mutable variable here, but this feels more functional to me.

So, how are we going to translate incoming data into something our greeter actor can understand? We'll need to come up with a protocol - I'm going to keep this as basic as possible and just use this for now:

```bash
<command> <name>
```

And here's our code updated to parse data into a `GreeterMsg`, passing it on to the greeter actor:

```fs
let handler connection (mailbox: Actor<obj>) =
    let rec loop connection = actor {
        let! msg = mailbox.Receive()

        match msg with
        | :? Tcp.Received as received ->
            let data = (Encoding.ASCII.GetString (received.Data.ToArray())).Trim().Split([|' '|], 2)

            match data with
            | [| "hello"; name |] -> greeter <! Hello (name.Trim())
            | [| "goodbye"; name |] -> greeter <! Goodbye (name.Trim())
            | _ -> ()
        | _ -> mailbox.Unhandled()

        return! loop connection
    }

    loop connection
```

If the input was more complex, you'd probably be better served using something like [FParsec](http://www.quanttec.com/fparsec/) to parse it - if this code starts getting too unwieldy in later posts, I might take a detour into that, but for now, this'll do.

Now we need to update our server actor to create handlers for each new connection. Rather than adding them directly to the root system, we can create a hierarchy by passing the server's mailbox to the spawn function.

```fs
let server = spawn system "server" <| fun (mailbox: Actor<obj>) ->
    let rec loop() = actor {
        let! msg = mailbox.Receive()
        let sender = mailbox.Sender()
        
        match msg with
        | :? Tcp.Bound as bound ->
            printf "Listening on %O\n" bound.LocalAddress
        | :? Tcp.Connected as connected -> 
            printf "%O connected to the server\n" connected.RemoteAddress
            let handlerName = "handler_" + connected.RemoteAddress.ToString().Replace("[", "").Replace("]", "")
            let handlerRef = spawn mailbox handlerName (handler sender)
            sender <! Tcp.Register handlerRef
        | _ -> ()

        return! loop()
    }

    mailbox.Context.System.Tcp() <! Tcp.Bind(mailbox.Self, IPEndPoint(IPAddress.Any, 9090))
    loop()
```

Note that we have to give each actor a unique name - here, I'm using the user's IP address and port to generate one. Akka doesn't like the square brackets in IPv6 addresses, so don't forget to strip those out!

With that, all the pieces are in place - if you run your program, and then use `telnet` to send some messages, you should see the greeter printing out messages to the server console!

## Responding

<small>[<i class="fa fa-github" aria-hidden="true"></i> View the code for this section](https://github.com/17cupsofcoffee/AkkaMUD/blob/part1d-responding/AkkaMUD/Program.fs)</small>

But that's no fun for our user, is it? Let's make a few more changes so our greeter can talk back to them.

```fs
let greeter = spawn system "greeter" <| fun mailbox ->
    let rec loop() = actor {
        let! msg = mailbox.Receive()
        let sender = mailbox.Sender()

        match msg with
        | Hello name -> sender <! sprintf "Hello, %s!\n" name
        | Goodbye name -> sender <! sprintf "Goodbye, %s!\n" name

        return! loop()
    }
    loop()
```

Pretty simple - instead of printing the text to stdout, we send it back to the actor which sent us the message. If you recall, that's our connection handler, so we need to teach that how to handle the new message type:

```fs
let handler connection (mailbox: Actor<obj>) =
    let rec loop connection = actor {
        let! msg = mailbox.Receive()

        match msg with
        | :? Tcp.Received as received ->
            let data = (Encoding.ASCII.GetString (received.Data.ToArray())).Trim().Split([|' '|], 2)

            match data with
            | [| "hello"; name |] -> greeter <! Hello (name.Trim())
            | [| "goodbye"; name |] -> greeter <! Goodbye (name.Trim())
            | _ -> connection <! Tcp.Write.Create (ByteString.FromString "Invalid request.\n")
        | :? string as response ->
            connection <! Tcp.Write.Create (ByteString.FromString response)
        | _ -> mailbox.Unhandled()

        return! loop connection
    }

    loop connection
```

Again, a really simple change! We just write any strings we receive out to the socket. I also added an error message for invalid commands, for completeness' sake.

If you try accessing your server through `telnet` again, you should now get the greetings returned straight to your console! It's hardly a MUD, but it's a start.

## Wrapping Up

We've now effectively got the 'gateway' into our actor system set up - the server listens for connections, the handlers parse data into strongly-typed messages, and then our greeter handles the actual logic. The important thing to note is that the greeter is totally unaware that it's part of a networked application - if we wanted, we could write a WebSockets endpoint or a unit test that sends messages into it, and we wouldn't have to modify the greeter code at all.

Next time, we'll start implementing some of the actual game on top of these foundations. I look forward to hearing people's feedback - I'm still very much a beginner with Akka, so don't hesitate to let me know if you think bits of the code could have been done better!