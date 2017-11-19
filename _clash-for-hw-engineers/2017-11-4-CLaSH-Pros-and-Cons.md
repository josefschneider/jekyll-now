---
layout: post
comments: true
title: CLaSH Pros and Cons
---

After years of being disappointed by the lofty promises of High-Level-Synthesis tools, it is only natural for every hardware engineer to be sceptical of what high level HDL languages may have to offer. Here is my incomplete list of what I have determined so far:

#### Pro - It's not an HLS language!

The language does not abstract away how the design is implemented in low-level hardware. As a CLaSH developer, you are still in control of every single register/signal/mux/etc. A circuit produced from CLaSH should be just as efficient as a circuit implemented in VHDL or Verilog.

#### Con (sort of) - It's not an HLS language

This is not a gateway language that will suddenly turn software engineers into hardware developers. CLaSH describes a digital circuit in Haskell code using the CLaSH library together with a subset of the Haskell programming language. It is not possible to take some arbitrary piece of Haskell code and to just turn it into hardware.

#### Pro - Compact, expressive code

It goes without saying that VHDL, and to a lesser extent Verilog, are very verbose. In CLaSH, interfaces, records and signal assignments are elegantly handled, and I'm frequently surprised by how easy it is to connect various bits of complex logic in just a couple of lines.

#### Con - The language is complex

VHDL and Verilog are also brutally simple, whereas Haskell is not, making it a lot harder to learn. For a beginner, getting even just a couple of lines to compile can take a few hours. Having said that, once you do get it to compile it generally _just works_.

#### Pro - Simulation

Because code written in CLaSH is valid Haskell you can perform functional tests using test-benches written in native Haskell. Result: insanely fast testing (if you know what you're doing) and amazing randomised testing capabilities provided by the Haskell QuickCheck library.

#### Con - There's an intermediate language before synthesis

The CLaSH code is compiled into verbose VHDL or Verilog. This makes it a bit harder to track down signals that are causing the design to not meet timing.

#### Pro - Haskell maps really nicely to hardware

Consider this example:

```vhdl
process(clk)
begin
    if rising_edge(clk) then
        if conditionA then
            [.. heaps of code ..]
            if conditionB then
                s <= '0';
            end if;
        elif conditionC then
            [.. even more code ..]
            s <= '1';
        end if;
    end if;
end process;
```

Now imagine this as a complex 1000 line process. It can be quite difficult to keep track of when `s` is set and cleared, and when it is left unchanged. This makes it easier to accidentally overwrite `s` later on in the process, or to forget conditions where `s` should be set to a certain value, but isn't. Haskell forces you to do this (every "\|" can be interpreted as an "if"):

```haskell
s
    | conditionA && conditionB = 0
    | conditionC = 1
    | otherwise = s'
```

Immediately it's clear what value is assigned to `s` under which conditions, and the circuit this code will create is also obvious. (Yes, I know... you can write something similar in VHDL, but VHDL doesn't force you to and in my experience code looks more like the process statement above). Note also that you can't overwrite signals, which is exactly how it works in hardware! This is why we can't assign `s` to `s`, we need to create the intermediate signal `s'` which we connect to the output of the register.

#### Con - No recursion

Oh the irony - you can't use recursion in CLaSH. 99% of the time that's okay though as the main type you would want to recurse over, CLaSH.Sized.Vector, comes with a lot of built in support for higher-order functions. The lack of recursion was an issue for me once when I wanted to implement a binary tree, the generic solution for which is still beyond my current Haskell capabilities.

#### Pro - Types

Haskell's fantastic type system ensures that every type is always exactly what you expect it to be.

#### Con - Types are sometimes annoyingly strict

Sometimes converting one type to another type, even though they are very similar, can lead a beginner to despair, e.g., what's the difference between Nat, SNat, Int and Integer, and why does it seem to be so difficult to convert one to the other?

#### Pro - A "Maybe" solves all your "valid signal" woes

In Haskell, a `Maybe x` is an optional value of type `x`. It either exists and has a value or it doesn't exist and will cause an error (in simulation) if you try to access it. This is useful as it's too easy to forget to check a valid signal in VHDL, causing us to read a default or common state which may be correct most of the time, but wrong sometimes. Wrapping a type in a Maybe is a very elegant solution to this problem.

#### Pro - Safe and robust complex designs

Many large complex circuits can't be split up into smaller parts (e.g., large state machines, or even our 1000 line process from above) and debugging and testing them can be extremely tedious. I have found that CLaSH is perfect for making such designs safe and robust because the language itself forces you to by being strictly typed and because signals can only be assigned a value once per clock cycle. Thanks to the constrained randomised testing capabilities I have been able to tease out corner cases I would never have thought of otherwise.

Even though I have listed nearly as many cons as pros, most of the cons are mere beginner's frustrations rather than actual shortcomings of the CLaSH language. The pros however offer significant advantages over conventional HDL languages in terms of expressiveness, safety, testing and conciseness.


{% if page.comments %}
<div id="disqus_thread"></div>
<script>

/**
*  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
*  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/

var disqus_config = function () {
this.page.url = "https://josefschneider.github.io/clash-for-hw-engineers/2017-11-4-CLaSH-Pros-and-Cons/";  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = "CLaSH Pros and Cons"; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
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
