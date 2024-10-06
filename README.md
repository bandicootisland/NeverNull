# NeverNull - TLDR
NeverNull contains methods, functions and patterns to eliminate null exceptions from C# code. The helper functions form a compact library, mostly static extension methods, that require only the addition of the function to an object. e.g Book.NeverNull() ( or the preferred short form, Book.NvN()).

For NeverNull to work best, it also requires a small programming philosphy change commitment - specifically: no nulls, ever!

And then, in the words of Napolean Dynamite (Pedro actually) - all your dreams will come true.

# NeverNull
Methods, Functions and Patterns to eliminate nulls from C# code

Out here in the fields - where we fight for our meals - we do not have the ability to handle nulls by using Options, monads,railway track programming, discriminated unions or the like in a huge rewrite of billions of line of existing code. We certainly cannot change language.
We are unsupported by existing libraries, such as Linq, which provide features we may use multiple times in a procedure, but which will return a null at the drop of a hat. 
There may be a million row in the database, but just one empty field in one row, perhaps entered a decade ago, will blow up the yearly reports.
The DotNet framework writes wonderfully concise and efficient code. But they too throw null exceptions as their code instantly gives up like a frightened rabbit if you say 'null' to it. Which means you are the dev trying to solve the issue, which could be 10, 20 layers deep or even more if a part of a set of async tasks.
If you are writing business applications, you know the call-centre cannot stop for a null value. The customer cannot be told to call back tomorrow. 
You have to code to protect from nulls, and be able to carry on when you do. You dont have the luxury to throw an exception.

And I am not sure if compiler and source code 'squiggles' help. Yes, yes, the value could be null - thats the problem! It could be null. It will be null. If you work in the real world, with real applications, you know every value everywhere, at some point now or in the future, will be a null. If you deserialise from a source, and only the most basic of applications do not, then you have a data source you cannot control, and it will have nulls in it one day.

This is not a rant about writing defensive code. I have no problem doing that (the defense I mean, rants are for free). 
In fact NeverNull itself is an entire library of defense. Built with one aim, expunging nulls out of existence.
And this is about the status of Software Engineering in general. No field of endevour with the word 'engineering' in its title can just throw up its metaphorical hands and refuse to work, just because of a special value. It is a ridiculous proposition, one that cannot stand. As an industry we have to solve it. We cannot continue to blame someone for a decision decades ago. That is a cop out.

It is now a Trillion dollar cost, and is fixable by us, if not now, then at the very least from now on.

Will NeverNull solve it? I am not sure I would be so bold as to say completely - but I have been using the ideas for years, and found it more than useful. It was the Crowd Strike massive fail that prompted me to finally document NeverNull. Yet again, a null reference point exception, which took down huge swathes of systems. Even after decades of trying the IT industry still seems unable to solve the issue.

So - every little bit helps. Have a read, and if there is food for thought here, or any ideas you can use to rid us all of the scourge of nulls, then have at it. 

The code here does have functions that have proved useful. So I think they provide a useful starting point. But it is not a panacea - what could be after decades of Industry coding?

Eliminating nulls from your own code is part philosophy, part approach, and part coding practice.

So if nothing else, this library hopefully will at the least provide food for thought for your solutions.

# The Problem
Lets get to the nub of the problem. Let me jot down a little object hierarchy, in C# - imagine we have the following:

Book.Publisher.FirstName

Lets says Publisher has not been assigned yet. It is null, perhaps the Book is newly created. So accessing any Publisher property it will throw the 'null pointer' exception. Fair enough - we dont want to access any areas of memory that are invalid.

But if I could paraphrase the memory issue, it would look like this:

Book.0x00000000.FirstName

Thats it! That is what apparently cannot be coped with, by decades of engineering. Yet a level out (prior to machine code) the compiler 'knows' that it is the Publisher object being referred to. And it 'knows' it has a First Name property. Otherwise your code will not compile. If you look at the disassembly, you can see type names and lookup tables through-out.
So the problem is with the '.' (dot) property accessor. Why cant it do more? But we are unlikely to ever get that change over the line, so lets solve it at the level we can effect: at code level.

#The Nub - 2
The issue we have is the exception being thrown (and of course it is good the compiler refused to access invalid memory). We should have written nested if statements, for each and every object, and even every property, depending on our deserialisation or mapping techniques.

But what if the object existed, even if it were empty. No error!

That is the essence of NeverNull: check for null and if null, return the object you first thought of. In my view - this is what the compiler should do. (Why would anyone anywhere want a null exception, ever? )

And make it convenient: so do it in a single function, that both provides the 'guard' lines of code, and also the null object protection by immediate creation of an object.

The essential functions is this:
[return: NotNull]
public static T NvN<T>(this T t) where T: new()
{
    if (t is object)
        return t;    
    return new();
}
(*The actual function is mildly different in the library, with some optimisations on the Create step. It is also necessary to have a 'new' constraint, discussed later.)

That is all we need to do to stop the exception. Ends up conceptually, as kind of like this (which will not compile of course):

Book.(publisherexists ?? new Publisher).FirstName.

Note: It is the generics in C# that allows this function to be used across objects, along with the static extension methods feature.


# Memory Objections
I can hear one objection already- that NvN creates objects that are in effect discards. Yes, I know: all solutions have to compromise. There is no get out of jail free card on this null monopoly board.
So a couple of points:
  You would have to write the defensive 'if null' check somewhere in your code. So that null validity check should exist anyway.
  NvN() will create an object only if it is null (to stop the error being thrown). If you are iterating through a large number of objects, with a significant number of null items, and you think the protection is a waste of memory, then dont do it! Simply write the check first:
    if (Book!=null)
      if (Book.Publisher!=null)
        if (Book.Publisher.FirstName !=null)
           if (Book.Publisher.FirstName.Length>0)
              //yay, we have a first name, just going to repeat all that for last name, and then concatenate the two. Also, middle name was just requested by business...so times 3
  (Yes, I know I can short form it with null coalescing operators; the logic is the point)
  
  or, within linq:
    foreach book in Books.Where(b=>b.Publisher!=null)
       etc.

 If a large number of properties on your domain models are always null, then perhaps the models are too wide and they should not be using up memory allocations.
The amount of memory used when a null exception is thrown, is enormous.
The amount of human resources is not measured in nano-seconds of GC - it is orders of magnitude more cost than an exception. We are talking hours of time, or with broad effects like Crowd-Strike, person-years of effort and loss.

However, if a lot of us use the NvN() technique, then the compiler will not stand still. The DotNet compilation team will see the code in GitHub, and could easily optimise away much of the cost. (inlining the method for example; probably already happening, or could be made to by the team easily)

# NeverNull - the name
One time I was updating the VS project xml, and working out the latest incarnation of 'Nullability' and if to 'enable' or not for the solution.
I realised the word I really wanted to write in that node was not enable, but Never:

<Nullable>Never</Nullable>

Never a null, never an exception, never an issue. Rock solid code, always working as intended.

What I intended needed to be a function (method call), and it would be used a lot, so I felt an abbreviation was acceptable in this case (I normally use descriptive method names).
But not too short, as that just obscures the intent. The most important letters in word recogition are the start and end, with a glance in the middle, so:

NvN  abbreviates well to 'Never Null'

Additionally, and obviously, it looks a little like NaN, which has been accepted in general (java) usage.

So NvN() is the name of a general null checking and coping set of functions.


# NeverNull - slight change to what the null checks means
Using the NvN() functions does mean empty objects can be returned in place of nulls. This can mean the logic and thinking behind a conditional has to be adjusted.

For instance, the following check for check:
  var isnull=Book.NvN().Publisher.NvN()!=null;
will never be true! 'isnull' will always be false. That is the point: NvN is a protector but does not replace other guards. (We cannot re-write the language here). 

To reiterate, this code would be pointless:
  Book.NvN().Publisher.NvN().FirstName="This assignment of a First Name will not error, but if Publisher was null, it wont persist anywhere.";

You use NvN() to handle nulls, but you still need some discrection and skill to interpret the result - like all programming.

You will at times have to walk down the object tree, especially when creating records. NvN() is not intended to be replacing the normal lines of code for significant code blocks, such as newing up the Publisher, or setting default properties. 

But for the vast majoirity of times, when you deserialised from the database, NvN() is going to prevent errors.

So, instead of using the 'not null', us a check like this:
  Book.NvN().Publisher.RecordId !=System.Guid.Empty
  
To that end, I have added the 'IsValid' concept as a check. IsValid() has a slighty higher bar than 'is not null' and higher than a simple new() object. It means something about the object proves it was retrieved from the database, or has its properties populated in some way

It does take some co-ordination with the default properties of an object.

Lets use a record id and the value type of Guid as an example. If a record is retrieved properly from a database, the primary key will be filled in. If it is a uniqueidentifier, then the Guid will not be empty (or put it this way, if you have one record with System.Guid.Empty as a value, then you probably should not)

So to check if a Publisher was deserialised:

if (Book.Publisher.PrimaryRecordId !=System.Guid.Empty)

i.e if it was new, perhaps created by NvN() on the fly, then a System.Guid type will default to an empty guid.

I put this kind of logic behind an 'IsValid()' function

if (Book.Publisher.PrimaryRecordId.IsValid())

or, if used a lot:
if (Book.Publisher.IsValid())
i.e: public static bool IsValid(this Publisher publisher){
  if (publisher is object && publisher.PrimaryRecordId!=System.Guid.Empty)
      return true;
   //else
   return false; 
}
You will find you cannot have too many of these IsValid checks.

What if it is not valid? then 
Book.Publisher=new Publisher(){
  FirstName="etc..."
};
before you proceed further.

In summary: with NvN() there is a slight change to conditionals: dont check for nulls, check for positive values. Lift the notion of an object existing, away from being simply 'not null' to one that has at least one core property with a valid value. Of course, you can check as many properties as you need, but I have found with database calls, if you have the primary guid, then it is sufficient to at least establish the call succeeded. This fact might even be viewed as more robust than 'not null'. (Of course, many other properties may be null, in the db)
 
# Never Null - never return Nulls
This one is not easy!
When first proving up Never Null for my use I finished a eight month project, coding solid for most of it as part of a team and using the Never Null approach, at least in my areas.
I thought I had done a good job. I searched for 'return null' from my methods, there were still hundred of times when I bailed out of a function early returning the classic 'return null;'.
We are so used to returning a null - it is an easy option. It has become ingrained.
Now a null return can be justified at times, but lets make those times special, and rare. As rare as hens teeth. 
Instead, return what the user expects. An object to work with.
If you are write extension methods, the first line will need to be thought about.
Lets run a builder class/function chain as an example:

public static Book BookBuilder(this Book book,Publisher publisher)

A little function to build up a book, with a publisher if known. What if the call to the database failed - for a myriad of reasons - and book were null.
Obviously we need a guard.
public static Book BookBuilder(this Book book,Publisher publisher)
{
    if (Book ==null){
       //now what? throw up our hands, and an error?
       }
    //also
    if (Publisher ==null){
      //less obvious. Might depend if a Publisher is optional, from a business perspective. What if it were unknown, i.e the Author is still unpublished? Do I have to propagate Optional everywhere now?
    }

We have done this thousands of times before. Bail and return nuill. But what if the builder looks like this, using a build concept:
    var book=  GetBook(_connection,matchontitle)//return a book
              .AddPublisher(_connection) //uses book
              .SetPageCount()              //works with book
              .AssignAuthor(newauthor)           
              ;
                   
That is what we want from our code: concise, builder pattern, chaining functions, functional composition.

But I would hazard there is hesitation to deploy such code, because there are nulls and exceptions are lurking within that set chained functions.
But we will never progress if we return nulls from functions. That simply delays the checking, and offloads the problem to a calling routines. It is what is giving rise to 'railway-track' programming, and quieting compiler alerts with 'NotNull' attributes and yet more code branching. 
The complexity will never end, in fact it is getting materially worse. It is a sysephean task - you want to break your code into discrete function blocks, but every function call means more null checking guard clauses, and more branching on each result.

Unless, we use NvN().
Here is the proposition - as one of the core principles to expunging nulls from our code:
No function can return a null, or throw a null exception. We must decide anew what it means to return early from a function.
And returning an empty, but instantiated object is the best alternative, if not the only alternative, to returning a null.
It is not easy to change this mindset, but it can be done. For the example above, GetBook takes a connection and a title, then passes a Book object along to further builder functions.
The only correct thing for GetBook to do, is to always return a Book object. In the happy path, it is retrieved.

Never a Null

public Book GetBook(DBConnection connection,string matchontitle)
{
if (connection==null)
  return new Book();
if (string.IsNullOrEmpty(matchontitle))//cant match
  return new Book()
//call database..No match? guess what...
  return new Book()

}

That way, the rest of the builder chain will work. All calls should be robust, and still need to do the null checking, as they should not assume they are called in sequence
Take CountPages() next:
public Book SetPageCount(this Book book)//function extends book
{
//classic, necessary check:
if (book==null)
   return new Book(); //never return null, return an empty object
//etc
}

If we take this approach, we will always have an object at the end. The chain will finish
The objection to this is that, what is the validity of the ending object? We still need to check it, and an IsValid() test is appropriate
var book=GetBook(...etc)
if (!book.RecordId.IsValid())
  //not valid: calls to the database did not work. We have an empty book with no pages.
  book=CreateNewBook(){
    title=matchontitle
  };














