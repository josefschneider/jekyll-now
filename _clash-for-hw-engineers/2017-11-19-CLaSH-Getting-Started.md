---
layout: post
comments: true
title: Getting Started
---

I found the following resources essential when getting started:
1. [Learn You A Haskell](http://learnyouahaskell.com/chapters), at least up to chapter 9 to start off (for CLaSH you don't really need monads until you start writing tests). It's a fantastic beginner's guide that goes over most of what you'll need.
2. [Functors, Applicatives, And Monads In Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html) which really helped me understand functors and applicatives which are used a lot in CLaSH.
3. The CLaSH getting [started tutorial](http://hackage.haskell.org/package/clash-prelude-0.11.2/docs/CLaSH-Tutorial.html), which is more a tutorial on how to use the tools rather than teach the language.

Great! So you've done the tutorials, have installed your CLaSH tools and have a rough understanding of Haskell, but how to write hardware? I hope to cover some basics here, especially things that I found confusing. Make sure to have the [CLaSH language reference](http://hackage.haskell.org/package/clash-prelude-0.11.2) and [Stackage](https://www.stackage.org/) open for your respective CLaSH and Haskell needs. When it comes to experimenting with different commands nothing beats the `clash --interactive` shell.


## Signal
A Signal is a functor that wraps a type, turning it into a clocked synchronous signal that can be used as a top-level input/output. A variable also needs to be wrapped by a Signal if it is to be registered or used by a state machine. Take the following two statements:

```haskell
s :: Signal Int = signal 40
b :: Int = 122
```

`s` is an integer signal variable that can be connected to a register, state machine etc. while `b` represents a time-independent variable, or in electronic terms, a simple wire. Luckily there's a function to turn our variable into a signal variable, `signal`:

```haskell
b :: Int = 122
sb :: Signal Int = signal b
```

This function was also used above to turn the integer 40 into a signal before assigning it to `s`. What about turning a signal variable into a simple variable? You can't! It's a functor. You need to perform an operation on the Signal through `fmap`, the result of which is another Signal:

```haskell
s' :: Signal Int = fmap (+10) s
```

or

```haskell
s' = (+10) <$> s    -- The ":: Signal Int" is implicit
```

In other words, once a signal is clocked, it can't be "un-clocked", which for most synchronous digital circuits makes perfect sense.

Having to constantly deal with functors through fmap gets tedious though, which is why I have taken to getting out of the clocked domain as quickly as possible. This allows me to write the asynchronous logic between the registers without having to worry about signals and the associated fmap and applicatives. It also makes for a very clear distinction between the sequential and the combinatorial parts of the circuit.

```haskell
-- Synch and asynch mixed in the same line of code
s' = register 0 $ (*30) <$> (\x -> x - 10) <$> s
```

vs

```haskell
-- Clear separation between synch and asynch code
s' = register 0 $ my_asynchronous_function <$> s

my_asynchronous_function :: Int -> Int
my_asynchronous_function in = (in - 10) * 30
```

`my_asynchronous_function` has an Int as an input, an Int as an output and can perform operations directly on those Ints. Though the sequential/combinatorial separation might be more verbose it tends to be infinitely easier to read and understand in more complex circuits.

Which brings me to my favourite CLaSH function: `mealy`. In one fell swoop mealy deals with input, output and registered signals while letting you focus on the combinatorial logic without having to unwrap Signal functors because `mealy` unwraps them for you!

```haskell
s' = (*2) <$> register 0 $ (*30) <$> (\x -> x - 10) <$> s
```

vs

```haskell
s' = mealy my_asynchronous_function 0 s

my_asynchronous_function :: Int -- Current state
    -> Int                      -- Input
    -> (Int, Int)               -- (Updated state, output)
my_asynchronous_function regOut s = ((s - 10) * 30, regOut * 2)
```

With a mealy function it is obvious what happens in a clock cycle, which values are written to the registers (state) and which values are written to the output. As you may have guessed, if the output is registered the `moore` function may be more appropriate.

Finally, signals can be bundled/unbundled. If a tuple is wrapped in a Signal and needs to be separated, `unbundle` does the trick:

```haskell
(a, b) :: (Signal Int, Signal Int) = unbundle $ signal (3, 12)
```

and `bundle` does the opposite:

```haskell
b :: Signal (Int, Int) = bundle $ (signal 3, signal 12)
```


## Bit array types
* `BitVector n`: a single-dimension vector of n bits, much like VHDL's std_logic_vector.
* `Unsigned n`: an unsigned integer of size n bits
* `Signed n`: a signed integer of size n bits

Creating a variable and assigning a value to it works as expected, though it does truncate the result if it is too big:

```haskell
bv :: BitVector 12 = 100    -- the value of bv is 100
u :: Unsigned 8 = 0x10F     -- u contains 15 as only the lowest 8 bits are kept
```

Turns out performing direct assignments like this is syntactic sugar for `fromIntegral x`, where `x` is our number. The `bv` assignment above is identical to the one below:

```haskell
bv :: BitVector 12 = fromIntegral 100
```

This is how one type can be converted to the other:

```haskell
u :: Unsigned 12 = fromIntegral (22 :: BitVector 12)
```

Care must be taken when converting types to make sure no data is lost to truncating! Assigning a bit array of one size to another can also be done through resize. The `bitSizeMaybe` function returns the size of the bit array (wrapped in a Maybe):

```haskell
bv :: BitVector 12 = resize (100 :: BitVector 10)
size :: Int = bitSizeMaybe bv
```

Addition, subtraction, multiplication, shiftR and shiftL are supported by all types:

```haskell
bv :: BitVector 12 = 100
bv2 = bv + 1
bv3 = bv * 100
bv4 = bv `shiftL` 2
```

And BitVector supports concatenation:

```haskell
bv = (0x55 :: BitVector 8) ++# (0xAA :: BitVector 8)
```

Then there's the `minBound` and `maxBound` functions that give the max and min value of the given type. Note also that the `Bit` type is simply an alias for `BitVector 1`. And that's about it! Bit array types aren't very powerful, but they don't need to be as we can rely on Vectors to do all the heavy lifting.


## [Vectors](https://www.stackage.org/haddock/lts-9.10/clash-prelude-0.11.2/CLaSH-Sized-Vector.html)

Vectors are CLaSH compatible, fixed-size arrays of a user-defined type, and that can be nested to create multi-dimensional arrays. Apart from some standard Haskell array functions (`tail`, `init`, `!!`, `findIndex`, `length`) they also come with fancy functions to perform repetivitve operations over the array.

Assigning a value directly to a vector is similar to creating a standard Haskell array:

```haskell
v :: Vec 4 Int = 1 :> 2 :> 3 :> 4 :> Nil
```

A commonly used vector type is `Vec n Bit`, which is similar to a `BitVector` but with a lot more capabilities. To turn one into the other the `v2bv` and `bv2v` functions can be used:

```haskell
v :: Vec 8 Bit = bv2v (255 :: BitVector 8)                      -- 1D vector
bv :: BitVector 4 = v2bv (1 :> 0 :> 1 :> 0 :> Nil)
```

These functions only work for 1D vectors though. For conversion to and from a vector of 1 or more dimensions use the `unpack` and `pack` functions:

```haskell
v2d :: Vec 4 (Vec 8 Bit) = unpack (0x12345678 :: BitVector 32)  -- 2D vector
bv2d :: BitVector 4 = pack (((1 :> 0 :> Nil) :> (0 :> 1 :> Nil) :> Nil) :: Vec 2 (Vec 2 Bit))
```

A potential source of confusion is that indexing is ascending within the vector:

```haskell
v :: Vec 4 (Vec 8 Bit) = bv2v (0x12345678 :: BitVector 32)
elem0 = v !! 0  -- elem0 contains 0x12, not 0x78!
```

Now that our bits are in Vector form we can do all kinds of things to them such as iterating, merging, concatenating, zipping, rotating, shifting and folding. I won't go through them all here as [the documentation](https://www.stackage.org/haddock/lts-9.10/clash-prelude-0.11.2/CLaSH-Sized-Vector.html) is very good at showing what these functions can do.

The few Vector functions I will mention are the ones that I found extremely useful and that didn't jump out at me:

* `splitAt` and `splitAtI` should be used to split a vector in two, based on an index or a destination type respectively.
* `replace` creates another array, replacing the value of one element at a given index in the array
* `reverse` to take care of all your endianness needs, amongst others
* `windows1d` for when you need to access successive windows within your vector. Say we need to access 4 consecutive bits from an array of 8 bits based on an index stored in variable `i`:

```haskell
--               |-- window 0 --|
--                    |-- window 1 --|
--                         |-- window 2 --|
--                              |-- window 3 --|
--                                   |-- window 4 --|
v :: Vec 8 Bit = 1 :> 0 :> 1 :> 0 :> 1 :> 1 :> 1 :> 1 :> Nil
w = windows1d d4 v  -- d4 is an SNat
i = 3
output = w !! i     -- The result is <0, 1, 1, 1>
```

This should mostly make sense apart from `d4`, which is covered next.


## [SNat](https://www.stackage.org/haddock/lts-9.10/clash-prelude-0.11.2/CLaSH-Promoted-Nat.html#t:SNat)

Singleton natural numbers are constants that are used to parametrise functions, i.e., they represent values. They are different from standard natural numbers which are used to represent types. For example, when defining `BitVector 32` the `32` is of type `Nat`. This subtle difference makes for one of the only real downsides I can think of of CLaSH when compared to VHDL. A VHDL generic can be used as a constant, can define bit widths, loop iterations etc. while in CLaSH all these values are represented by a different type, and it's sometimes not possible to convert one to the other.

Defining an SNat the shorthand way can be done by adding a `d` prefix to the number:

```haskell
sn :: Snat 3 = d3
```

The range of `d` SNats is limited, maxing out at 1024:

```haskell
sn = d1024  -- No problem
sn = d1025  -- Error!
```

This has to do with some funky Haskell type stuff, but luckily there's a way around this limitation though it's a bit more verbose:

```haskell
sn = SNat @ 20000000 -- No problem
```

It is possible to do some arithmetic operations on SNats using the provided functions and, most usefully, to convert an SNat into an integer:

```haskell
i = snatToInteger d23
```


## [Index](https://www.stackage.org/haddock/lts-9.10/clash-prelude-0.11.2/CLaSH-Sized-Index.html)

An Index is an Integer that starts at 0 and has an arbitrary upper bound (unlike standard `Int` integers that always have a bit-width of 64 bits). If the value of the index is beyond the upper bound an exception is thrown when the variable is read. There is not much to the Index type, apart from the `bv2i` function that turns a BitVector into an Index:

```haskell
i :: Index 8 = 7
```


{% if page.comments %}
<div id="disqus_thread"></div>
<script>

/**
*  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
*  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/

var disqus_config = function () {
this.page.url = "https://josefschneider.github.io/clash-for-hw-engineers/2017-11-19-CLaSH-Getting-Started/";  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = "Getting Started"; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};

(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = 'https://https-josefschneider-github-io.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
{% endif %}
