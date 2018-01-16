- title : Intro to Functional programming via F#
- description : Why F# is awesome and how to start using it right now
- author : Alexander Prooks 
- theme : night
- transition : default

***

## F# |> LV

<br />
<br />

<br />
Alexander Prooks - [@aprooks](http://www.twitter.com/aprooks)

***

### OOP vs Functinal


``` C#

public class Calculator
{
    //mutable state
    double amount = 0;
    
    //incapsulation
    public void Add(double value){
        amount += value;
    }
}
```
---

linear function

    F(x) = x + 1 

F#

    let f x = x + 1


two arguments

    F(x,y) = x + 2*y 

F#

    let f x y = x + 2*y

---

    let add x y = x + y

    let result = add 2 5

***

### Data structures


#### Record types

    type User = {
        Phone: string
        Name: string
        LastName: string
    }

---

[Full comparison with C#](https://fsharpforfunandprofit.com/posts/fsharp-decompiled/)

---
### Using records

    let dto = {
        Phone= "79062190016"
        Name= "Alexander"
        LastName= "Prooks"
    }

---

### Syntax sugar

    let copy = {
        Phone= "79062190016"
        Name= "Alexander"
        LastName= "Prooks"
    }
    copy = dto //values comparison
    //true
    
    let b = {a with Name="Test2"}
    //!!!!

    b = a //false
---

### Multiple case type (DU)

    type Gender = 
    | Male
    | Female
    | Other of string
    
    type CustomerWithGender = {
        // ... 
        Gender: Gender
    }
    
    let a = Male
    let b = Other "111"

---

### Pattern matching

    let toString gender= 
        match gender with 
        | Male -> "male"
        | Female -> "female"
        | Other s -> s

    //same as:
    let toString =
        function 
        | Male -> "male"
        | Female -> "female"
        | Other s -> s

---

### other way round

    let fromString = function
        | "male" -> Male
        | "female" -> Female
        | other -> Other other


***

### Basic Functional calculation

    type Operator =
    | Add
    | Deduct
    | Multiply
    | Divide

    type CalculationTask = {
        Operator: Operator
        Left: double
        Right: double
    }

    let calculate op = 
        match op.Operator with
        | Add -> op.Left + op.Right

---
    //example

    let input = {
        Operator: Add
        Left: 10
        Right: 20
    }

    let result = calculate input

---

### Memory (handling state)?

    type State = {
        Amount: double
    }
    with static member Zero() = {
            Amount = 0.
        }

    type Operation =
    | Add of double
    | Substract of double
    | Multiply of double
    | Divide of double

    let calc state op =
        match op with
        | Add a -> { Amount = state.Amount + a}
        //..V
---
### full 

    let res1 = calc (State.Zero()) (Add 10.) 
    // db.Save res1

    //let res1 = db.Load
    let res2 = calc res1 (Add 20.)
---

### advanced

    let opsList = [
        Add 10.
        Substract 20.
        Add 50.
        Multiply 2.
    ]

    let resFull zero = 
        (calc 
            (calc (calc zero (Add 10.))
                (Substract 20.))
                    (Add 50.))

    let folded zero ops = 
        Seq.fold (fun acc el -> calc acc el) zero ops

***

# F is for Functions!

---

## Reading signatures

    // string -> string
    let append (tail:string) string = "Hello " + tail
    
    // infered types:
    let append tail = "Hello " + tail
    
    // append 10 //compile error
    append "world" //"Hello world"

    // string -> string -> string
    let concat a b = a + b

    // unit -> int
    let answer() = 42

    // string -> unit
    let devnull _ = ignore() 

---

## Function as params

    // (string -> unit) -> (unit->'a)
    let sample logger f = 
        logger "started"
        let res = f()
        logger "ended"
        res
    
    let consoleLogger output =
        printfn "%s: %s" (System.DateTime.Now.ToString("HH:mm:ss.f")) output
    
    let result = sample consoleLogger 
                        (
                            fun () -> 
                                System.Threading.Thread.Sleep(500)
                                42
                        )

---

## 'a -> 'a -> 'a

    // int -> int -> int
    let sumInts (a:int) (b:int) = a+b     

    // static int sum(decimal a, decimal b) = { return a+b}
    // etc 

    // 'a -> 'b -> 'c
    //           when ( ^a or  ^b) : (static member ( + ) :  ^a *  ^b ->  ^c)
    let inline sum a b = a + b   //WAT??

    let d = 10m + 10m
    let c = "test" + "passed"
    let d = 100 + "test" //error

***
# let (|>) x f = f x

---

## Currying

    // string -> string -> string
    let concat x y = string.Concat(x,y)

    // <=>

    // string -> (string -> string)

    // string -> string
    let greet = concat "Hello"
    // <=>
    let greetVerbose w = concat "Hello" w

---

## DI F# way


    module Persistence = 

        let saveToDb connString (id,obj) = 
            // blah blah
            Success 

    module CompositionRoot =
        
        let connectionString = loadFromConfig("database")
        let persist = saveToDb connectionString

    let result = persist ("123",customer)
    
---

## Data pipe [0]

    let evenOnly = Seq.filter (fun n -> n%2=0) {0..1000}
    let doubled = Seq.map ((*) 2) evenOnly
    let stringified = Seq.map (fun d-> d.ToString()) doubled
    let greeted = Seq.map greet stringified

    // ["Hello 0","Hello 4", ...]
---

## Data pipes [1]

    let inline (|>) f x = x f   
    
    let evenF = (|>) ( {0..1000} ) ( Seq.filter (fun n -> n%2=0) )
    
    let evenInfix = {0..1000} |> ( Seq.filter (fun n -> n%2=0) )

---

## Piped data!

    {0..1000}
    |> Seq.filter (fun n -> n%2=0) //numbers
    |> Seq.map ((*) 2) //evenOnly
    |> Seq.map (fun d-> d.ToString()) //doubled
    |> Seq.map greet //stringified

---

## Real world like

    let handlingWrapper myHandler request = 
        request
        |> Log "Handling {request}"
        |> Validator.EnsureIsValid
        |> Deduplicator.EnsureNotDuplicate
        |> Throttle (Times 5) myHandler
        |> Log "Handling finished with {result}"

***

### F# in UI

<img src="images/hotloading.gif" style="background: transparent; border-style: none;"  />

---

## F#/OCaml ecosystem

* https://facebook.github.io/reason/
* https://github.com/alpaca-lang/alpaca
* http://elm-lang.org/
* http://fable.io/
* https://ionide.io

---


***

# Q?