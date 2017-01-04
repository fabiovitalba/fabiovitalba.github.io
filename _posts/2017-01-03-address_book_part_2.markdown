---
layout: post
title:  "Address Book - Part 2"
subtitle: "Structs and where to find them"
date:   2017-01-03 17:05:14
categories: [rust]
---
Since the holidays are almost over I thought I should add another blogpost about the struct I used in my program. Let's dive in.

For reference we're talking about this program: [Address Book Manager](https://github.com/fabiovitalba/address_book)
The current version works, but has some sub-optimal usages. This is because I used LinkedList instead of Vectors. I will change my code accordingly over the next days, so stay tuned!

Now let's dive right in. The creation of a struct in Rust is pretty straight forward. Just forget about lifetimes and ownership for a moment:

{% highlight rust linenos %}
  // Struct to hold each Address in the Address Book
  #[derive(Clone)]    
  struct AddressBook {
    id: u32,
    name: String,
    address: String,
    postcode: String,
    city: String,
    country: String
  }
{% endhighlight %}

I may be renaming this struct when I'm reworking my code to use Vectors instead of LinkedLists, but for now we'll just keep calling it &quot;AddressBook&quot; even though it's more of a &quot;Address&quot; struct. Anyway I used classic fields or attributes to store the information about the address. I also used an id-field to store the id of the address. This field won't be ported over to my Vector-version, because Vectors allow direct indexing and therefore I can just use those indexes (indices?) to know which address is which.
However I'm deriving the [Clone Trait](https://doc.rust-lang.org/std/clone/trait.Clone.html) which allows me to create an exact copy of the struct. This could be done by creating a new Function for the struct (as we will see later on), but since Rust has already implemented this Generic Trait for all Data-Types we can just go ahead and derive it and use it for our AddressBook struct. This allows us to use the function {% highlight rust %}.clone(){% endhighlight %} as we will see later as well.

Since this was all pretty straight forward for now, we can continue by my implementation of certain functions for my Struct. Now there are two ways of doing this. Either we want to just implement some functions for our struct (like I did in the following example) or we create a new Trait that we implement either for generic-types or specifically for our struct or enum. I will go over the second way later in a different program or maybe even in this after I rewrote the program to use Vectors.
Anyway, here you can see how I implemented the function new for my struct:

{% highlight rust linenos %}
  // Implement various Functions for the Struct AddressBook
  impl AddressBook {
    // Pseudo-Constructor
    fn new(n_id: u32, n_name: String, n_address: String,
           n_postcode: String, n_city: String,
           n_country: String) -> AddressBook
    {
        // If no Return is stated, the last line is returned
        AddressBook {
            id: n_id,
            name: n_name,
            address: n_address,
            postcode: n_postcode,
            city: n_city,
            country: n_country
        }
    }

    [...]
  }
{% endhighlight %}

This function <span class="hljs-function">new()</span> works in a static-context (you will see a relative-context-function as the next example). This is because we just have some Parameters that give you a return value. This function just takes the parameters and creates a new instance of the struct AddressBook from those parameters. As you can see, there is no &quot;return&quot; keyword. This is because in rust, when there is no return-keyword and you have a return value, the last value in the function will be returned (I'm not quite sure whether it's the last line or the last value of the return type, but I don't think thats really relevant here).

Now let's look at a relative-context-function as I called it earlier:

{% highlight rust linenos %}
  // Implement various Functions for the Struct AddressBook
  impl AddressBook {
    [...]

    // Reset all values in this Struct.
    fn reset(&mut self) {
        self.id = 0;
        self.name = "".to_string();
        self.address = "".to_string();
        self.postcode = "".to_string();
        self.city = "".to_string();
        self.country = "".to_string();
    }

    [...]
  }
{% endhighlight %}

As you can see here we have a parameter that is a mutable reference to the instance of the Struct AddressBook itself. If you want to create relative-context-functions you can just add a self-reference as a first parameter. Then you can use the keyword &quot;self&quot; to access all the available functions and attributes that you can use in a AddressBook.
In this case the reference is mutable, because I want to reset all the values in the struct. The really isn't much else about this but it's pretty cool to know that you can implement functions for structs just like you can implement functions in classes in object oriented languages like Java.

And for usage it's pretty close to how you would use it in Java or C#:

{% highlight rust linenos %}
  // Loads the addressbook from the current directory into a Linked List.
  fn load_existing_addressbook() -> LinkedList<AddressBook> {
    [...]

    let mut curr_addr_book = AddressBook::new(0,format!(""),
                                                format!(""),
                                                format!(""),
                                                format!(""),
                                                format!(""));

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
    }
    [...]
  }
{% endhighlight %}

As you can see we use both functions we covered earlier in this function &quot;<span class="hljs-function">load_existing_addressbook</span>&quot;. To use the static function we call it trough a :: and the AddressBook struct. On the other hand, if we want to use the relative-context-function we just simply use it by calling it over the instance of the AddressBook we created.
In this function I use both of my struct-functions since the &quot;<span class="hljs-function">load_existing_addressbook</span>&quot; function creates the AddressBook-entries by reading each line from the file where they are saved and create a new AddressBook instance from the content of the file. Those instances then get added to the LinkedList (which will be changed into a Vector) and the List is complete.
To avoid lifetime-errors in this iteration we use the <span class="hljs-function">.clone()</span> function and just simply add a copy of the current address into the LinkedList.
