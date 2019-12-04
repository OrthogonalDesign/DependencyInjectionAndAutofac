## Dependency Injection & Autofac
### in the SyncTool

Note:

---
### Before vs After 
```CSharp
  PollingPublisher = new Publisher<DateTime>();
  var syncCommandPublisher = new Publisher<SyncCommand>();
  PollingPublisher.Subscribe(new GetAPISyncRequest(syncCommandPublisher));
  syncCommandPublisher.Subscribe(new SyncLegalEntityProxy());

```
```CSharp
    PollingPublisher = container.Resolve<Publisher<DateTime>>()
    builder.RegisterGeneric(typeof(Publisher<>));
    builder.RegisterType<GetAPISyncRequest>().AsImplementedInterfaces();
    builder.RegisterType<SyncLegalEntityProxy>().AsImplementedInterfaces();
```
---
### What is DDD
Not a technology or Methodology

But a set of principles and patterns for design

Note:
Actually DDD is just Object-Oriented Programing, 
but guides for design decision. Design is architect.

And sometimes the principles/patterns are
not only tell you what you supposed to do 
but also what you are NOT supposed to do.
So I prefer say DDD help you to make design decision.

---
### What is DDD
The business logic is the hard part of the software
But it is also the core value of the software

Domain Driven is Business Driven, 
so the full name is Business Domain Driven Design.

Note:
OK. So is that not what we doing now?
Yes. We always implement the business logic in our code. 
But there is different way to do that, 
different way is what I said design decision

+++
### “Talk is cheap. Show me the code.”
#### YearMonth
Lot of business involves monthly processing:
* Oil Royalty Product Period 
* Payroll Cycle 
...

Note:
Let look a very simple example.

There are lot of scenario which use the month as a unit period, 
the month here is different from the month part of a date, 
so I just named it as the YearMonth to make it clearer.

And most time our developer will simply use DateTime to represent this type, 
like this:

+++
### YearMonth by DateTime

```CSharp
public class Entity
{
    property DateTime Period {get; set;}
}

```

Note: 
Simple start and not problem at all, until more requirement coming

+++
### YearMonth requirement
* Get the next month of the given month
...

Note:
In the real project, there are more requirements for sure.
Here just list one as a example.

+++

### YearMonth Function Implementation
```CSharp
public class YearMonthUtilities 
{
    public static DateTime GetNextMonth(DateTime month)
    {
        ...
    }
    
    public static DateTime GetFirstDayOfTheMonth(DateTime date)
    {
        ...
    }
}

```

Note: 
I am confident any developer know how to implement these two methods 
and even did in the real project include myself.

Please look at the second method, which is not from the requirement at all, 
but I do need it, and it even used more often than other feature related functions.

But do you see the problem now? 

* Because we use DateTime to represent YearMonth which is not exactly same thing which 
bring some I called technical requirement (instead of business code) which is not actually
necessary. 
* But consider the potential bug prevention, it is better to always convert a give datetime
to the first day of the month, even it is already did before. Because from the snippet code 
you working on, you have no idea.

+++
### YearMonth 
#### Another Design Decision
```CSharp
public class YearMonth 
{
    public YearMonth(int year, int month)
    public YearMonth(DateTime date_time)
    public DateTime as_date_Time()
    public YearMonth get_next_month()
}
```

Note:
Instead of just implementing the feature (get_next_month), I abstracted a concept 
YearMonth in this solution, which is represented by the object. 
With this simple change, I am modeling, and modelling loudly. 

Actually that is DDD about, DDD is just the model driven engineering. And the term
Domain Model is actually the pattern name in 
Martin Fowler's book <Patterns of Enterprise Application Architecture>.

With modeling, now we have our first member 
in the *vocabulary* of our *ubiquitous* language now.

---
### What is DDD again
#### Benefit
* Ubiquitous language:  (YearMonth)
* A bridge which connects:
    * Business Analyst
    * Domain Expert
    * Developer

---
### Why DDD
#### Different Modelling Paradigms
 * Data(base) Modelling (Mathematical Paradigm)
 * Object(-Oriented) Modelling 
 * Service(-Oriented) Modelling
 
Note:
Most time when we say data modelling means database modelling which is mathematical paradigm,
but DDD is Object modelling which is engineering paradigm.
When we modelling in real project, we are kind of mixing the data modelling and object modelling
that introduce confusion.
For example, we use both one to many relationship and inheritance relationship on same diagram 
which indicate in our mind we actually use both paradigms.

Regarding this situation, the DDD just tell us that we should use object modelling without
affected by database model at all. That is call persistence ignorance. 

---
### Freedom for Modelling

Note:
After get rid of the burden from database modelling, yes, I consider it as a burden.
We are free to empower our model now.
I just skip the long story and show you final picture of my YearMonth modelling.

+++
#### Freedom for Modelling
### YearMonthRange
A period from a start yearmonth to a end yearmonth
Usage: Application effective period or similar qualification period
```CSharp
var yearMonthRange = 2019.year(1).to(2019.year(10));

//OR
Januray.year(2019).to(October.year(2019)
```

Note:
Based on the simple YearMonth object I showed before, I create another object YearMonthRange.

Another example is jetbrain licence valid period.

And its a function ==>

+++
#### Freedom for Modelling
### YearMonthRange Overlap
```CSharp
 2019.year(1).to(2019.year(6))
    .overlap(2019.year(4).to(2019.year(12)))
       .ShouldEqual(2019.year(4).to(2019.year(6))); //Assert
```

+++
#### Freedom for Modelling again
### YearMonthMap
A data/value on a period(YearMonthRange)

```CSharp
YearMonthMap<int> salary;
salary.put(2019.year(1).to(2019.year(12), 5000);

/*
$5000   |--------------...---|
        |                    |
        |                    |
    ----+----+----+----...---+---
       Jan                  Dec
*/
```

Note:
YearMonthRange is not the end but a start, based on it, I created another object YearMonthMap
The code above represent the data/diagram below.
Now you can see, the data model is so complicated , but the code still clean enough.
Image how we can reach this step by the utilities/method solution.
BTW: All the codes are real code, and can be run through unit test. 

--- 
### Value Object
>An object that describes some characteristic or attribute but carries no concept of identity.

--Eric Evans

>Objects that matter only as the combination of their attributes.
>Two value objects with the same values for all their attributes are considered equal.

--Martin Fowler

Note:
So far, we just peek one piece of DDD today: Value Object
But I think it is enough to open you mind.
When we come back to look at the definition of Value Object. ^

Although the second one from Martin Fowler is easier to understand, 
but still not enough to tell you when and how to use value object 
if I just show you at the beginning of the presentation.

---
### Why we did utilities way
* Reinvent wheel
* Hard/Longer time to implement
* Are you smarter than Microsoft?

Note: 
They just don't know what you need.

--- 
![Layer](assets/img/DDD-layered-architecture.png)

Note:
This is original Layered Architecture from Eric Evans.

Copied:
the UI layer interacts to business logic, and business logic talks to the data layer, 
and all the layers are mixed up and depend heavily on each other. 
In 3-tier and n-tier architectures, none of the layers are independent; 
this fact raises a separation of concerns. 
Such systems are very hard to und erstand and maintain. 
The drawback of this traditional architecture is unnecessary coupling.

---
#### Improved Onion Architecture
![Onion](assets/img/Onion_Architecture_1.png)

Note: 
Most importance factor in this diagram is that 
the database is not the center, it is external.
But because we still follow the Layered architecture,
so this architecture implemented heavily on the Dependency Inversion principle.

Copied:
Most of the traditional architectures raise fundamental issues of tight coupling and separation of concerns. 
Onion Architecture was introduced by Jeffrey Palermo 
to provide a better way to build applications in perspective of 
better testability, maintainability, and dependability. 
Onion Architecture addresses the challenges 
faced with 3-tier and n-tier architectures, and to provide a solution for common problems. 
Onion architecture layers interact to each other by using the Interfaces.

---
#### Another diagram
![Onion](assets/img/Onion_Architecture_2.png)

Note:
Start from the Onion architecture, we are ready for microservice architecture.

--- 
# Q & A

