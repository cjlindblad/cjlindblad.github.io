---
title: Reading some Ruby source code
date: "2020-07-09T12:40:32.169Z"
template: "post"
draft: false
slug: "reading-some-ruby-source-code"
category: "Ruby"
tags:
  - "Ruby"
description: "Where my quest to avoid learning to write proper hash functions made me dive into the Ruby source."
---

I'm a Ruby newbie, learning the language by doing some old Advent of Code puzzles. In [one of them](https://adventofcode.com/2015/day/16), I found the Array `-` (minus) method to be very useful. In short, it returns the difference between two arrays. My particular use case was to check if every element in the first array was included in the second one, which would return an empty array:

```ruby
irb> [2, 4] - [1, 2, 3, 4, 5]
  => []
```

However, my elements weren't integers but rather custom types. As per [the docs](https://ruby-doc.org/core-2.7.1/Array.html#method-i-2D), we need to override `eql?` and `hash` to make it work.

That got me thinking about how a proper hash function is supposed to work. Some prime number times the hash of every member? Not sure. And that got me thinking even more — when would the hash function be used in my case? Under what conditions? Why isn't `eql?` enough?

And then I realized that the actual C source code is listed right there in the docs. Let's dive in!

```c
static VALUE
rb_ary_diff(VALUE ary1, VALUE ary2)
{
    VALUE ary3;
    VALUE hash;
    long i;

    ary2 = to_ary(ary2);
    ary3 = rb_ary_new();

    if (RARRAY_LEN(ary1) <= SMALL_ARRAY_LEN || RARRAY_LEN(ary2) <= SMALL_ARRAY_LEN) {
        for (i=0; i<RARRAY_LEN(ary1); i++) {
            VALUE elt = rb_ary_elt(ary1, i);
            if (rb_ary_includes_by_eql(ary2, elt)) continue;
            rb_ary_push(ary3, elt);
        }
        return ary3;
    }

    hash = ary_make_hash(ary2);
    for (i=0; i<RARRAY_LEN(ary1); i++) {
        if (rb_hash_stlike_lookup(hash, RARRAY_AREF(ary1, i), NULL)) continue;
        rb_ary_push(ary3, rb_ary_elt(ary1, i));
    }
    ary_recycle_hash(hash);
    return ary3;
}
```

I'm not what you would call a fluent C reader, but we should be able to sort this one out.

The broad strokes are that we supply our two arrays (named `ary1` and `ary2`), diff them, and store the result in `ary3` which is returned.

The first bit of conditional logic seems to check if either array is smaller than some constant named `SMALL_ARRAY_LEN` (if we poke around in the [repo](https://github.com/ruby/ruby/blob/master/array.c#L47), we find that it's defined to `16`). If so, it will iterate over every element in `ary1` and push it into `ary3` if it is not included in `ary2`. The inclusion check is performed by `rb_ary_includes_by_eql(ary2, elt)`, which we probably can assume runs in O(n) time.

Okay, so if the number of elements to check is within reason, we do it the brute force way. So, what if it isn't? Then we make a hash (Ruby for [dictionary](https://docs.python.org/3/tutorial/datastructures.html#dictionaries) or [map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)) of `ary2` and check inclusion in O(1) time!

### <🐰 Rabbit hole>

This raises a question — which Ruby method does `ary_make_hash` correspond to? It certainly isn't `to_h`:

```c
irb> [1, 2, 3].to_h

TypeError (wrong element type Integer at 0 (expected array))
```

That method [expects one array of keys and one array of values](https://ruby-doc.org/core-2.7.1/Array.html#method-i-to_h). That isn't what we're doing here.

If we search for `ary_make_hash` in the repo, we only get matches in the `array.c` file. Could this mean that it's an internal function? Let's compare it to a method we know is in the public API, like `unshift`. Near the bottom of `array.c`, we find [this](https://github.com/ruby/ruby/blob/master/array.c#L8074). Seems like the mapping of Ruby method name to C function. We also notice that every C function exposed this way is prefixed with `rb_`, like `rb_ary_unshift`. Now we can be pretty sure that `ary_make_hash` only is used internally. Let's take a look at it!

```c
static VALUE
ary_make_hash(VALUE ary)
{
    VALUE hash = ary_tmp_hash_new(ary);
    return ary_add_hash(hash, ary);
}

static inline VALUE
ary_tmp_hash_new(VALUE ary)
{
    long size = RARRAY_LEN(ary);
    VALUE hash = rb_hash_new_with_size(size);

    RBASIC_CLEAR_CLASS(hash);
    return hash;
}

static VALUE
ary_add_hash(VALUE hash, VALUE ary)
{
    long i;

    for (i=0; i<RARRAY_LEN(ary); i++) {
			VALUE elt = RARRAY_AREF(ary, i);
			rb_hash_add_new_element(hash, elt, elt);
    }
    return hash;
}
```

`ary_make_hash` uses two other functions, `ary_tmp_hash_new` and `ary_add_hash`. They're all listed above, and hopefully, it's enough to figure out what's going on. `ary_tmp_hash_new` just creates an empty hash with a size matching the passed array. The solution lies in the line `rb_hash_add_new_element(hash, elt, elt);` from `ary_add_hash` — we get a hash where the keys and values are the same, e.g:

```ruby
irb> [1, 2, 3].make_hash #(Doesn't exist)
  => {1=>1, 2=>2, 3=>3}
```

We probably could have guessed it, but where's the fun in that?

### </ 🐰 Rabbit hole>

Let's return to `rb_ary_diff`.

```ruby
static VALUE
rb_ary_diff(VALUE ary1, VALUE ary2)
{
    # other stuff excluded..

    hash = ary_make_hash(ary2);
    for (i=0; i<RARRAY_LEN(ary1); i++) {
        if (rb_hash_stlike_lookup(hash, RARRAY_AREF(ary1, i), NULL)) continue;
        rb_ary_push(ary3, rb_ary_elt(ary1, i));
    }
    ary_recycle_hash(hash);
    return ary3;
}
```

We make a hash out of `ary2`. Then we'll iterate over `ary1` and look the elements up in the hash. If they're not found, we push them onto `ary3`. And since we're in C land, we got to return the memory for the hash!

This all means that the requested `hash` override is only ever used if either array is longer than 16 elements, and even then it's a performance optimization (a really bad hash function turns a hash into a list). In my case, no array is ever longer than 10 elements, so I can safely ignore overriding it without risking any penalties!

### Conclusion

In the quest of avoiding learning how to write a proper hash function, I poked around in the Ruby source code and actually learned a thing or two about how things are organized — specifically the naming convention of functions belonging to the public Ruby API. This might be the beginning of a whole lot of Ruby source code spelunking!
