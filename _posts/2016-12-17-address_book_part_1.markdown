---
layout: post
title:  "Address Book - Part 1"
subtitle: "Handling an Address Book based on a File"
date:   2016-12-17 17:57:30
categories: [rust]
---
I have been a bit busy over this week and therefore couldn't get this far earlier. But let's dive in anyway.

I can't remember if I found this exercise online or if I just came up with it after reading something similar. But here comes my description of what I want to achieve. I wanted to create an [Address Book Manager](https://github.com/fabiovitalba/address_book) that reads the addresses you have in a file and lets you modify, delete, add and show these addresses. Maybe I can even get some understanding of encryption or compression with this to create a super-small-weight or secure Address Book Manager. But that will be Step 2 somewhen in the future! Hell, I might even add a README.md file this time!

But let's get to the interesting bits. I had to start by setting a File Format. I just decided to take a new extension ".ab" and use fixed spacing for my layout. I might be able to make it dynamic later on. I probably should put most of this stuff in some kind of config file, so you may even be able to use this (if you don't mind having no GUI).
Therefore I started setting up some constants. These are pretty straight forward for someone who has programmed before, thus I won't be explaining them. I'll just show you the examples to get an idea for the syntax:

<pre><code class="rust">
{% highlight Rust linenos %}
  static INSTANCE_SEPERATOR: &'static str = "****";
  static DEFAULT_FILE_NAME: &'static str = "addr_book.ab";
{% endhighlight %}
</code></pre>

I'm using these for my basic setup. After I had this set up, I had to get working on the Writing & Reading part. I was so overwhelmed with the errors I got, that I had to ask some people on the [Rust IRC for #Rust-Beginners](https://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust-beginners) for help. And I was helped almost instantly. I have to give the user kazagistar special thanks, as he gave me an explanation that I - as someone who is coming from Java, C and C/AL - can understand and so I wanted to share it with you:

> String is like a Vector of expandable buffer full of characters on the heap
> &str is a pointer to an immutable string somewhere

Now this is no exact quote, but almost perfectly close. Anyway, after this revelation I was ready to keep going and I got to a point where everything is working like I want it to.

Writing to File
===============

So I decided to use a LinkedList to save all my AddressBooks after I loaded them from a File. Turns out that writing to file is a lot easier than I thought it would be at the beginning. The current state looks like this:

<pre><code class="rust">
{% highlight Rust linenos %}
  fn save_address_list(curr_list: &LinkedList<AddressBook>) {
    let fname = DEFAULT_FILE_NAME;

    let mut file = match File::create(fname) {
      Ok(file) => file,
      Err(e) => panic!("File error: {}", e)
    };

    let mut writer = BufWriter::new(&file);
    for addr_book in curr_list {
      let mut linebuffer = INSTANCE_SEPERATOR.to_string() + "\n";
      writer.write(&linebuffer.into_bytes());

      linebuffer = "id      : ".to_string() + &(addr_book.id.to_string()) + "\n";
      writer.write(&linebuffer.into_bytes());

      [...]

      linebuffer = INSTANCE_SEPERATOR.to_string() + "\n";
      writer.write(&linebuffer.into_bytes());
    }

    writer.flush();
  }
{% endhighlight %}
</code></pre>

Lots of Stuff happening here, but let's break it down. I have created a Function save_address_list that takes a LinkedList of AddressBooks as a parameter. You can see that it's a borrowed version of that LinkedList as I only need to read the values from it to write them to file and so I don't really need a mutable reference.
Then I open a [File](https://doc.rust-lang.org/std/fs/struct.File.html) named after the DEFAULT_FILE_NAME constant that I defined earlier. As you can see I am again using the Option<T> type I had to use last week.
If this file was successfully opened, I use a [BufWriter](https://doc.rust-lang.org/std/io/struct.BufWriter.html) to write to it. The BufWriter should be pretty straight forward as well, as almost every Programming Language I came across handles this in a similar way.
And since I am using a LinkedList, I can iterate very nicely with a "foreach"-similar for [...] in [...] loop. And then I just simply write each information I have about this Address to file. There really isn't much else to say about this. And so I won't.

Reading from File
=================

Next up is the read function. This one is a bit trickier, but when boiled down, it's pretty simple as well. Here it goes:

<pre><code class="rust">
{% highlight Rust linenos %}
  fn load_existing_addressbook() -> LinkedList<AddressBook> {
    let mut addr_book_list: LinkedList<AddressBook> = LinkedList::new();

    [...]

    let reader = BufReader::new(file);

    let mut is_new_instance = false;
    let mut curr_addr_book = AddressBook::new(0,format!(""),format!(""),format!(""),
                                                format!(""),format!(""));

    [...]
{% endhighlight %}
</code></pre>

Here I am creating a new LinkedList<AddressBook> to store all the addresses from the file in. Done that I just open the File and open a [BufReader](https://doc.rust-lang.org/std/io/struct.BufReader.html) on this file. This works pretty similar to the writing part, so I won't go into detail.
The variable is_new_instance is needed for me, because I have an information for each address book on a line-by-line setting. Meaning I have the contact name on one line, the address on another, and so on. Therefore I must know if I have to switch to the next address or not, and this is handled by this variable.
Then I set up a new AddressBook instance called curr_addr_book and create it using a Constructor function I wrote for it. I may cover these Structure specific informations in my following posts. Once I created this instance I go ahead and read every single line of the file.

Now let's see how I handle this separator string I declared earlier:
<pre><code class="rust">
{% highlight Rust linenos %}
  [...]
  for line in reader.lines() {
    let mut linebuffer = line.unwrap_or("".to_string());
    if linebuffer.len() >= INSTANCE_SEPERATOR.len() {
      if &linebuffer[..INSTANCE_SEPERATOR.len()] == INSTANCE_SEPERATOR {
        if is_new_instance {
          is_new_instance = false;
          addr_book_list.push_back(curr_addr_book.clone());
        } else {
          is_new_instance = true;
          curr_addr_book.reset();
        }
      }
    }
    [...]
{% endhighlight %}
</code></pre>

For each line I receive from the BufReader I create a linebuffer. This linebuffer contains the value of the read line from the file if it was successfully read, or an empty string if it broke on an error.
Then I go ahead and check if this linebuffer is my instance seperator which I defined earlier. If this is the case, I set the variable is_new_instance to true, so that I know that the next time I pass by one line of a INSTANCE_SEPERATOR I can push the AddressBook into my LinkedList.
After I set is_new_instance to true, I make sure that every value in the AddressBook was reset, to avoid having old values copied into the wrong address.

If this line is no instance separator, I check if it's actually a value from my AddressBook. This is done similar to the separator check, as you can tell by looking at the source code:
<pre><code class="rust">
{% highlight Rust linenos %}
  [...]
  if linebuffer.len() >= 10 {
    match &linebuffer[..10] {
      "id      : "    => {
        match (&linebuffer[10..]).parse::<u32>() {
          Ok(n) => curr_addr_book.id = n,
          Err(e) => curr_addr_book.id = 0,
        }
      },
      "name    : "    => {
        curr_addr_book.name = (&linebuffer[10..]).to_string();
      },
      [...]
{% endhighlight %}
</code></pre>

And this is really all there is to these Write & Read functions I created. In my next Post I will be talking about the Struct AddressBook that I created and then there will be a third one explaining my LinkedList operations on Adding, Modifying and Deleting Addresses from the List. But I still have to code them as well, so this is probably going to wait a while.
