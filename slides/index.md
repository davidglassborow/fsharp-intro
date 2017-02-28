- title : Introduction to F#
- description : Introduction to F#
- author : Dave Glassborow
- theme : night
- transition : default

***

### What is F#?

- ML / OCaml derivative (influenced by Haskell & Erlang)
- Influencing F*, Elm ( [Elmish](https://github.com/fable-compiler/fable-elmish) ), C# (Linq,async/await)
- Open source since 2005: [FSharp.org](http://fsharp.org) & [Github](https://github.com/fsharp/)
- Functional First but very pragmatic
- Supports Actor model & OO
- Infers Types
- Used a lot in finance (correctness + speed), but general purpose language

***

### Examples

    let simpleValue = 5
    let factorial x = [1..x] |> List.reduce (*)

    let reversePipe = factorial <| 1+2      // $ in haskell
    let composition = factorial >> ((+) 1)  // . in haskell

    let optional: int option = None         // Maybe in haskell


[F# cheatsheet](http://dungpa.github.io/fsharp-cheatsheet/)

***

### 5 things beyond FP

- It runs *everywhere*
- Linear dependancies
- Light weight actors
- Active Patterns
- Type providers

***

### Where can you run F# ?

- Mac, Linux, Windows, FreeBSD (mono or .net core)
- Android, iPhone
- GPU
- Cloud functions (AWS Lambda, Azure Functions)
- MBrace
- Juptier notebooks
- Spark
- Javascript

> **Atwood's Law**: any application that can be written in JavaScript, will eventually be written in JavaScript.

***


### Linear Dependancies


- Compiles top to bottom, left to right, single pass
- Annoying at First
- Grown to love the constraint
- Foundations first, type driven development
- Leads to less coupled code, less cyclic dependancies

---

#### Tickflow

[Shamlessly copied from FSFFAP](http://fsharpforfunandprofit.com/posts/cycles-and-modularity-in-the-wild/)
![Tickflow](http://fsharpforfunandprofit.com/assets/svg/tickSpec.all.dot.svg)

---

#### Specflow

![Specflow](http://fsharpforfunandprofit.com/assets/svg/specFlow.all.dot.svg)

***


### Light Weight Actors

- Good for state diagrams, thread safety
- Post and Receive, sync or async, timeouts
- Supports thousands of them
- Async pool

##### Examples

- Stateful
- Bounded queue
- Web Socket

---
    open System

    type Result = Even of int | Odd of int 

    let agentLogic (inbox:MailboxProcessor<AsyncReplyChannel<Result>>) =
        let rec odd(counter) = async {
            let! msg = inbox.Receive()
            msg.Reply(Odd counter)
            return! even(counter + 1)} 
            
        and even(counter) = async {
            let! msg = inbox.Receive()
            msg.Reply(Even counter)
            return! odd(counter + 1)}

        odd 1

    let agent = MailboxProcessor<_>.Start agentLogic
    let getNext () = agent.PostAndReply id

---

    type internal Token = int
    type internal ThrottlingAgentMessage<'t> = 
        | GetToken of AsyncReplyChannel<Token>
        | ReturnToken of Token

    type ThrottlingAgent(limit) = 
        let agent = MailboxProcessor<ThrottlingAgentMessage<_>>.Start(fun agent -> 
        
            /// Represents a state when the agent is blocked - uses 'Scan' to wait for completion of some work
            let rec blocked() = 
                agent.Scan(function ReturnToken t -> Some( hasCapacity (limit - 1)) | _ -> None) 
            
            /// Represents a state when the agent has capacity
            and hasCapacity inUse = async { 
                let! msg = agent.Receive()
                match msg with 
                | ReturnToken _  -> return! hasCapacity (inUse - 1))
                | GetToken resp  ->
                    resp.Reply(1)
                    if inUse < limit - 1 then return! hasCapacity (inUse + 1)
                    else return! blocked() }
            hasCapacity 0)          // Start in working state with zero running work items
        member x.GetToken() = agent.PostAndReply( GetToken )
        member x.ReturnToken(token) = agent.Post( ReturnToken token )


***


### Active Patterns

- Abstraction over types

#### Complete patterns

    let (|Even|Odd|) i = 
        if i % 2 = 0 then Even else Odd

    let testNumber i =
        match i with
        | Even -> printfn "%d is even" i
        | Odd -> printfn "%d is odd" i

---

#### Parametised

    let (|DivisibleBy|_|) by n = 
        if n % by = 0 then Some DivisibleBy else None

    let fizzBuzz = function 
        | DivisibleBy 3 & DivisibleBy 5 -> "FizzBuzz" 
        | DivisibleBy 3 -> "Fizz" 
        | DivisibleBy 5 -> "Buzz" 
        | i -> string i

***

### Type Providers

- Removing magic strings
- Irritating when the build breaks


CSV,JSON,XML,HTML: [FSharp.data](https://fsharp.github.io/FSharp.Data/index.html)


---

### Lots of them

- Regular Expressions
- AWS S3 / Azure storage
- Databases (ORMS, querys, sprocs)
- WMI
- OData
- Hadoop
- Slack
- [R](http://bluemountaincapital.github.io/FSharpRProvider/)

---

[Rogue One TP]( http://www.pinksquirrellabs.com/post/2017/01/19/Star-Wars-Rogue-One-Type-Provider-edition.aspx)

![silly](http://www.pinksquirrellabs.com/image.axd?picture=r11_thumb.png)
![silly2](http://www.pinksquirrellabs.com/image.axd?picture=r12_thumb.png)


***

### Other stuff

- Quotations: expression trees
- Computation Expressions: Monads + DSLs
- Units of measure


    [<Measure>] type m
    [<Measure>] type sec
    [<Measure>] type kg

    let distance = 1.0<m>    
    let time = 2.0<sec>    
    let speed = 2.0<m/sec>    
    let acceleration = 2.0<m/sec^2>    
    let force = 5.0<kg m/sec^2>
    let travel = time * speed
