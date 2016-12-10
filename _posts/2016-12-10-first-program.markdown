---
layout: post
title:  "First Program"
subtitle: "A simple input string analyzer"
date:   2016-12-10 10:46:46
categories: [rust]
---
Ok so I didn't quite know where to start as I have already read through some of the [Rust Manual](https://doc.rust-lang.org/stable/book/) and the  [Rust-by-example](http://rustbyexample.com/) website. Which I recommend you do as well if you are going on this journey into rust alongside me.
In the end, after this small program I made yesterday, I feel like rust isn't all that different to C as people make it out to be. Of course there is the concept of Ownership, the concept of Lifetimes and whatnot, but in the end it isn't all that different, at least to me for now.

So let's talk about what I learned yesterday. So first of all, let me explain what I wanted to create: I wanted to make a [string analyzer](https://github.com/fabiovitalba/string_analyzer) that gives you the length of a string and the most used character.

This turned out to be more difficult than I expected. Since I cannot figure out how to do this with a two-dimensional array and I cannot figure out how to do it with a [LinkedList](https://doc.rust-lang.org/std/collections/struct.LinkedList.html). So i had no choice but to make a very memory-inefficient one-dimensional array that stores the count of each character that I can find in the string of input.
Of course I know that I may not cover all cases and I know that there is probably a better way to do this, but I frankly didn't care. I wanted to get coding and I wanted to learn the quirks and features of rust.

Now, shall we get to the code? I started out by using a [BufReader](https://doc.rust-lang.org/std/io/trait.BufRead.html) to read the input lines from the user.

{% highlight rust %}
  let reader = io::stdin();

  // each line the user types in the terminal will be evaluated seperately
  for line in reader.lock().lines()   {
      let to_analyze = line.unwrap();
      [...]
  }
{% endhighlight %}

Along these lines you can see that I use a unwrap to get the content of the line I get from the BufReader. If you check out my source code on my repository, you can see that I also use the unwrap() function for the conversion between char and u32.
This is needed because what we get is actually not always a string when reading a line or a char when converting from u32. What we actually get is a value of the type of [Option<T>](https://doc.rust-lang.org/std/option/enum.Option.html). This Option-variable is kind of like a safety-net for your Value. If you have a valid value as a result, you can [unwrap()](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap) it to get the value. In this case, if you get an invalid value or an error, it will return with an error. But if you use unwrap_or() like I did, you can actually set a standard value to default to, in case your unwrapping fails.

So as we go on you can see that I used a function to get the most used character. It's conveniently called:

{% highlight rust %}
fn get_most_used_char(new_string: &str) -> (char, u32) {
  [...]
}
{% endhighlight %}

... and as you can see, it's actually returning a [tuple](https://doc.rust-lang.org/nightly/std/primitive.tuple.html). Which is great, since I couldn't figure out what the advantages of tuples are before getting to this very point. Because I have this tuple as a return value I can actually return the most used character as well as it's occurrence in the input string. This is actually pretty nice.
And while I'm actually thinking about it, I could have probably made the whole procedure a lot more memory efficient by using a tuple array. Maybe I will come back to this and change that about it.

So this is probably everything I learned by doing this exercise. I invite you to stay tuned for more exercises and posts about rust. If you have any good advice on my coding or any exercise that might be very helpful to learn a new language, you're very welcome to send me a mail, twitter or anything else really.
