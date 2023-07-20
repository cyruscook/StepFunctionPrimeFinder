# Step Function Prime Finder
A demonstration of a an AWS Step Functions State Machine that can check whether a number is prime, and another state machine which can check a range of numbers against that state machine.

## How?
Step Functions state machines cannot do much math - the only mathematical operation available is addition.
We can however, use another method available - the [`States.ArrayPartition`](https://docs.aws.amazon.com/step-functions/latest/dg/amazon-states-language-intrinsic-functions.html#asl-intrsc-func-arrays) method which takes an array and a partition length and splits up the original array into an array of arrays of a length not exceeding the partition length.
This actually reveals important information to us - the length of the final sub-array will be either the partition length (if the array length is divisible by the partition length) or the remainder of dividing the array length by the partition length.

Using this, we can calculate whether a number is prime using a trial division algorithm - (where `x > 1`) `x` is prime if, for `2` up to `x/2` as `y`, `x mod y != 0`

Unfortunately, there's a slight limitation - the `States.ArrayRange` method needed to create the array that is partitioned can only return an array of 1000 items or less.
This means that this cannot check whether any numbers over 1000 are prime.
There may be a way around this by using an API call to retrieve an array with more elements, but you would want to avoid calls that charge.

## Why?
Step Functions are designed to orchestrate other services and not really to do any work themselves, which causes them to be a lot cheaper than other services like lambda for execution time.
This is just finding ways to do more with step functions than intended (even if this paticular use case isn't very useful).

## Architecture

### IsPrime state machine
![IsPrime state machine diagram](IsPrime.svg)

### PrimeFinder state machine
![PrimeFinder state machine diagram](PrimeFinder.svg)

# Deployment
Create a state machine from `IsPrime.asl.json`, and `PrimeFinder.asl.json`, replacing the `{account_id}` and `{isprime_sm_name}` in PrimeFinder with the id of the account the IsPrime state machine is in, and the name of the IsPrime state machine in that account.

Call IsPrime with an input like so:
```json
{
  "x": 10
}
```
And it will return an output like so:
```json
{
  "prime": false,
  "x": 10
}
```

Call PrimeFinder with an input like so:
```json
{
  "low": 2,
  "high": 10
}
```
And it will return an output like so:
```json
{
  "prime": [
    3,
    5,
    7
  ]
}
```
