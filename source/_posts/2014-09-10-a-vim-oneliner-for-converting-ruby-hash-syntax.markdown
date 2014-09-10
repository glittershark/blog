---
layout: post
title: "A vim oneliner for converting ruby hash syntax"
date: 2014-09-10 13:07:59 -0400
comments: true
categories: vim,ruby
---

As of ruby 2.1, hashes that previously looked like:

```ruby
{
    :one => '1',
    :two => '2'
}
```

can now use the following syntax:
```ruby
{
    one: '1',
    two: '2'
}
```

I like this syntax a lot; it's cleaner, more concise, and more consistent with
other languages (javascript, python...). However, many code generators (Rails
generators et al.) will still generate code that uses the old syntax. I came
upon such an occasion today and wrote the following one-liner in Vim to convert
between the two:

```vim
%s/:\(\S\{-}\)\(\s\{-}\)=> /\1:\2/
```

We can further convert this into a command that we can use with a custom range
with the following line:

```vim
command! -range ConvertHashSyntax <line1>,<line2>s/:\(\S\{-}\)\(\s\{-}\)=> /\1:\2/
```

This will preserve spacing and ignore cases where hash keys aren't symbols. For
more fun with the vim `:substitute` command, check out my post that contains an
introduction and another sample use case
[here](/post/2014/08/04/an-introduction-to-the-vim-substitute-command/).

