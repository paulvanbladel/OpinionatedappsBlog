+++
date = "2016-06-20T15:48:27+00:00"
draft = false
title = "On the exceptional nature of .Net Exceptions"
slug = "on-the-exceptional-nature-of-net-exceptions"

+++

## Exceptions are meant for exceptional circumstances.
Non thoughtful usage of exceptions is as evil as using global variables and 
goto statements.

Basically an exception is an instrument meant for programmers in the sense that a programmer can use it to inform another programmer (or herself) that 'a bug is in the air'. An exception is not something to be used to inform a user of your software about a certain (failing) condition.
In practise there are only a few use cases where exceptions should be raised and consumed.
For the cases where traditionally a lot of programmers abuse exceptions, we will elaborate here a cleaner and more functional oriented alternative.

## What's the precise problem with exceptions?

In (functional) programming method signatures must be pure and trustworthy and methods should be treatable as real mathematical functions. A mathematical function maps values between a domain (the set of allowed input values) into a codomain (the output of the mathematical function) in a 100% deterministic way. So there is no ambiguity about the input and output values: the same input always leads to the same output.

We want to only deal with pure functions because only then we can treat them as black boxes. 

## Why do we prefer black box methods?

When looking with more epistemological eyes to software architecture we might discover following mechanism: sometimes it can be useful to simplify certain sub aspects of a system not with the intention of simplifying the overall system, but with the sole purpose of *being ready to handle more complexity*. 

The same idea applies to methods as black boxes, they give room for more complexity because it becomes more easy to reason about the software because we don't need to bother about the internals of every piece in the overall system.

## Why do exceptions violates the pure nature of a (mathematical) function?

1. Exceptions pollute the codomain with an additional pseudo return type. Image a method that return integers, but the internals of that method can raise under specific conditions also an exception.  How do we reason then about the return type of that function? How do we now that 
there is kind of hidden part, namely the exception? The pure nature of the return type is jeopardized.

2. An exception complicates the natural direct flow of a function call and reminds us to the 'goto hell' of the late eighties of the previous century (DLL hell happened a decade later).
When a function raises an exception, there is no guarantee on the exact location in the call stack hierarchy where the exception will be 'consumed' via a try catch statement. Therefor, an exception is even worse than a goto statement because a goto statement doesn't travel around in the call stack.

3. The producer and consumer of the exception might not be on the same page about the 'exceptional nature' of the exception. Sometimes a specific condition can be an exception for the initial author of the method, but for someone else simply an 'edge case'. As consumer of a method I would prefer that these assumptions are not made for me, I would prefer to decide myself.

## So, should my code base be completely exception free?

No ! 

In order to give a thoughtful answer we need to make distinction between 
1. the activity of throwing an exception and consuming an exception via a try catch block and
2. exceptions raised by your own code and code from third party libraries (e.g. entity framework)

### Only throw exceptions on method contract signature violations
This are the classic ArgumentNullException and InvalidOperationException.

Consider following constructor for a Result class where a the semantics of a Result require that a succesful results can't contain an ErrorType and a non-succesful result needs an ErrorType:

```language-csharp
protected Result(bool isSuccess, ErrorType? errorType)
        {
            if (isSuccess && errorType != null)
                throw new InvalidOperationException();
            if (!isSuccess && errorType == null)
                throw new InvalidOperationException();

            IsSuccess = isSuccess;
            ErrorType = errorType;
        }
``` 

Notice that the semantics of the method needs to be rich enough to express this kind of non-atomic pre-conditions. A positive Results with attached errors is from semantic perspective nonsense. This need for decent semantics in functional programming touches nicely the notion of an ubiquitous language in Domain driven design or is at least a gentle invitation to think about semantics.

The exceptions used in this example are meant for the programmer/consumer of this method to tell them they introduced a bug in the code base. So it has nothing to do with validating user input, it's about the violation of the pre conditions of a code contract and non recoverable and thus a real exceptional situation.

### Appropriate consumption of third party library exceptions
What is exceptional for the third party library may very well be non-exceptional for you as consumer of that library. So, catch these exceptions on a fine grained level as close as possible to the call to the third party library and transform them into a non exceptional handling. Don't handle all other exceptions from the third party library for which you cannot transform to a non exceptional situation. What happens a lot is that these unexpected exceptions are swallowed in one way or another. Don't do this and fail fast as it will in the long run increase the quality of your code base. Remember, desperate diseases need desperate remedies.

Where will these 2 types of exceptions be consumed?  The next paragraph will introduce the notion of a *apology call center*.

### Every app needs an *apology call center*. 
Treat your users with respect and make an apology when your app crashes.

Every application architecture has kind of highest level point like a void main or in case of an asp.net application: the typical global.asax. That's the place where you should install what I call deliberately an *apology call center*, a simple try catch around the initial bootstrapping of your application with the sole purpose of making an apology for the bug that the user encountered and some logging statements.

That try catch block should never be used to try to recover from the exception. If post-mortem analysis would reveal that indeed recovering from the exception would have been possible, apply then that logic for a fix in the place where the exception was initially raised. 

### Use a Result class to be able to return data together with expected failures

So, we know now that we only raise exceptions in our own code when the signature contract is violated, but how can we deal with the ambivalent nature of a method's return type. Indeed we must be able to either return a value or fail with an expected error reason.

We need something close to the code example I provided above but we need also to be able to return
 data of type T:

```language-csharp
public class Result<T> : Result
    {
        private readonly T _value;
        public T Value
        {
            get
            {
                if (!IsSuccess)
                    throw new InvalidOperationException();

                return _value;
            }
        }

        protected internal Result(T value, bool isSuccess, ErrorTypebase errorType)
            : base(isSuccess, errorType)
        {
            _value = value;
        }
    }
```
The ErrorType is a kind of extended enum class which can be populated as follows:
```language-csharp
public class ErrorType : ErrorTypebase
    {
        public static readonly ErrorType DatabaseCanCauseALotOfTrouble
            = new ErrorType(1, "Database error of some sort", "some addition value I might use when dealing with error");
        public static readonly ErrorType NoXinNameAllowed
            = new ErrorType(2, "No X in Name Allowed", "some addition value I might use when dealing with error");

        private ErrorType(int value, string displayName, string myAdditionalValue)
           : base(value, displayName, myAdditionalValue) { }
    }
```
The advantage of this approach for the ErrorType class over a classic enum that there is more room for expressivity when providing the return text. (an enum is basically just an integer value, the ErrorType class provided here additional room for a string value containing the text of the excepted error condition)
ErrorType derives from ErrorTypebase which could reside in a more framework related assembly. ErrorTypebase contains the necessary plumbing for the hybrid enum class. I borrowed the approach of the Return class from Vladimir Khorikov (http://enterprisecraftsmanship.com/)

Following unit tests provide some simple examples on how to work with the Result class:
```language-csharp
 public class ResultClassDemoTests
    {
        [Fact]
        public void CorrectCallToPaymentGatewayGivesPositiveResult()
        {
            Result<PaymentResponse> result = new PaymentGateway().Pay(DateTime.Now, "paulus");
            Assert.Equal(true, result.IsSuccess);
            Assert.NotNull(result.Value.EffectivePaymentDate);
        }

        [Fact]
        public void IncorrectCallToPaymentGatewayGivesNegateResultWithReason()
        {
            Result result = new PaymentGateway().Pay(DateTime.Now, "xpaulus");
            Assert.Equal(false, result.IsSuccess);
            Assert.Equal(ErrorType.NoXinNameAllowed.Value, result.ErrorType.Value);
        }
    }

```
The PaymentGateway is kind of mini 'anti-corruption layer' towards the real thirdparty lib:

```language-csharp
public class PaymentGateway
    {
        public Result<PaymentResponse> Pay(DateTime date, string customerName)
        {
            try
            {
                var client = new ThirdPartyApiClient();
                PaymentResponse result = client.DoPayment(date, customerName);

                return Result.Ok(result);
            }
            catch (NoXinNameException)
            {
                return Result.Fail<PaymentResponse>(ErrorType.NoXinNameAllowed);
            }
            //don't handle further unknown execption, let it go :)
        }
    }
```
Note how we catch the potential exceptions from the third party library and transform them
to a Return class. The ThirdPartyApiClient is nothing more than:
```language-csharp
 //this class mimics a 3rd party library returning an exception in the error case
    internal class ThirdPartyApiClient
    {
        public ThirdPartyApiClient()
        {
        }

        internal PaymentResponse DoPayment(DateTime date, string customerName)
        {
            if (customerName.StartsWith("x"))
            {
                throw new NoXinNameException("name starts with x, not allowed");
            }
            return new PaymentResponse
            {
                effectivePaymentDate = DateTime.Now,
            };
        }
    }
```
I'm working on a small 'library' containing some useful classes to make your code more
functional. The sources can be found here:
https://github.com/paulvanbladel/DotNetFuncToolBelt
The full code of the above example including the unit tests can be found there as well.

















