---
title: Building some Ruby source code
date: "2020-07-10T12:40:32.169Z"
template: "post"
draft: false
slug: "building-some-ruby-source-code"
category: "Ruby"
tags:
  - "Ruby"
description: "Where we add a pointless method to the Ruby language!"
---

In [my last post](reading-some-ruby-source-code), I fumbled around in the Ruby source code and took a look at an internal function that is not exposed via the public Ruby API. It's called `ary_make_hash` and turns an array into a hash where each key-value pair has the same value.

In imaginary Ruby, it would work like this:

```ruby
irb> [1, 2, 3].make_hash #(Doesn't exist)
  => {1=>1, 2=>2, 3=>3}
```

As a learning exercise, let's alter the Ruby source so that the example above would actually work!

### Building Ruby

First, we gotta be able to build Ruby. There's a [guide](https://github.com/ruby/ruby/blob/master/README.md#how-to-compile-and-install) in the repo, but that just resulted in a wall of C compiler errors sprinkled with complaints about the openssl lib. I was just about to desperately try it on a virtual Linux box instead (I'm running macOS), when I found [this](https://github.com/ko1/rubyhackchallenge/blob/master/EN/2_mri_structure.md#exercise-build-mri-and-install-built-binaries).

A detailed guide for getting started on hacking the MRI (the standard Ruby C implementation)! This worked like a charm on the latest stable branch (ruby_2_7). I feel like this should be on the official README. Possible pull request coming up!

### Hacking Ruby

As we found out last time, the mapping of Ruby method name to C function is located at the bottom of each relevant source file. For example, here's the line for [Array.to_h](https://github.com/ruby/ruby/blob/master/array.c#L8052). So, let's pop in a new line and map it to `ary_make_hash`:

```c
...
rb_define_method(rb_cArray, "make_hash", ary_make_hash, 0);
...
```

I'm not sure what the last argument does, but the regular `to_h` passes `0`, so we'll probably be fine using that as well.

Try to run it and.. 💥 BAM 💥

```ruby
irb> [1, 2, 3].make_hash

[BUG] Segmentation fault at 0x0000000000000020

(Followed by a million lines of traces and error messages)
```

A segfault pops up! Not really used to these, but it seems like we're trying to read a part of the memory that we shouldn't.

However, if we assign the expression to a variable, it works just fine.

```c
irb> hash = [1, 2, 3].make_hash
irb>
```

Of course, if we try to do anything with the `hash` variable, everything blows up again. Let's look at the source for `ary_make_hash`:

```c
static VALUE
ary_make_hash(VALUE ary)
{
    VALUE hash = ary_tmp_hash_new(ary);
    return ary_add_hash(hash, ary);
}
```

Make a temp hash, you say? That sounds suspicious. Let's dig deeper:

```c
ary_tmp_hash_new(VALUE ary)
{
    long size = RARRAY_LEN(ary);
    VALUE hash = rb_hash_new_with_size(size);

    RBASIC_CLEAR_CLASS(hash);
    return hash;
}
```

It looks like we should do fine if we just use the call to `rb_hash_new_with_size` without calling the clear function afterward. Let's make a new function, and call it `rb_ary_make_hash` to indicate that it's visible to Ruby (with the `rb_` prefix):

```c
static VALUE
rb_ary_make_hash(VALUE ary)
{
    VALUE hash = rb_hash_new_with_size(RARRAY_LEN(ary));
    return ary_add_hash(hash, ary);
}
```

And swap the method mapping for our new function:

```c
rb_define_method(rb_cArray, "make_hash", rb_ary_make_hash, 0);
```

### Running Ruby

Moment of truth coming up. Will it work?

```ruby
irb> [1, 2, 3].make_hash
  => {1=>1, 2=>2, 3=>3}
```

It actually does!

This is of course pretty pointless, but we figured out how to build, hack and run our own version of Ruby! Next time, we might be able to try something useful!
