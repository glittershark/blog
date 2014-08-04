---
layout: post
title: "An introduction to the Vim :substitute command"
date: 2014-08-04 14:08:20 -0400
comments: true
categories: vim
---

I know a lot of people who use Vim as a sort of glorified console-mode notepad. They
write a bunch of things twice, have a tendency to mash backspace when they made a typo
earlier on the line, and perform a lot of very repetitive actions to modify code. Even if
you’re being a “good vim user” and using `hjkl` instead of cursor keys, if you frequently
find yourself doing the same thing twice then you’re really not taking advantage of the
full power that Vim has to offer.

The `:substitute` command is an awesome way to get rid of a lot of that repetition. Though
it may seem like just a find and replace command it packs a full regex engine, which with
references to numbered groups means you can rearrange and reorganize lines of code to your
liking. Let’s take a look at an example of a use case I just encountered the other day.

<!--more-->

An Example
----------

Let’s say we’ve just run a `\d` (describe) command on a table in PostgreSQL, and we’ve
copied the following list of columns from the output:


```postgresql
psql=> \d users                                           
                               Table "public.users"
         Column         |            Type             |                     Modifiers
------------------------+-----------------------------+----------------------------------------------------
 id                     | integer                     | not null default nextval('users_id_seq'::regclass)
 email                  | character varying(255)      | not null default ''::character varying
 created_at             | timestamp without time zone |
 updated_at             | timestamp without time zone |
 encrypted_password     | character varying(255)      | not null default ''::character varying
 reset_password_token   | character varying(255)      |
 reset_password_sent_at | timestamp without time zone |
 remember_created_at    | timestamp without time zone |
 sign_in_count          | integer                     | not null default 0
 current_sign_in_at     | timestamp without time zone |
 last_sign_in_at        | timestamp without time zone |
 current_sign_in_ip     | character varying(255)      |
 last_sign_in_ip        | character varying(255)      |
 confirmation_token     | character varying(255)      |
 confirmed_at           | timestamp without time zone |
 confirmation_sent_at   | timestamp without time zone |
```


Let’s say we’re issuing a SQL UPDATE command on this table, and want to pass in values as
named parameters using the `:parameter` syntax. So that’d look like this:

```sql
UPDATE users SET
    id = :id     
    email = :email     
    created_at = :created_at     
    updated_at = :updated_at     
    encrypted_password = :encrypted_password     
    reset_password_token = :reset_password_token     
    reset_password_sent_at = :reset_password_sent_at     
    remember_created_at = :remember_created_at     
    sign_in_count = :sign_in_count     
    current_sign_in_at = :current_sign_in_at     
    last_sign_in_at = :last_sign_in_at     
    current_sign_in_ip = :current_sign_in_ip     
    last_sign_in_ip = :last_sign_in_ip     
    confirmation_token = :confirmation_token    
    confirmed_at = :confirmed_at    
    confirmation_sent_at = :confirmation_sent_at
;
```

We can take the list of columns from the postgres output and translate it into that
`UPDATE` command in just one `:substitute` command. Assuming we’ve selected the column
list using [visual-line mode](http://vimdoc.sourceforge.net/htmldoc/visual.html#V), here’s
that command:

```vim  linenos:false
:'<,'>s/ \(.\{-}\) .*$/\1 = :\1/
```


That looks pretty complicated, so let’s break it down step by step

---------------------------------------


```vim  linenos:false
:'<,'>s
```

This is just the command to start the `:substitute` command - as with all Ex commands,
`:substitute` has a short form and we’re using that (`:s`) here. You don’t need to worry
about typing the `'<,'>` bit, as Vim will enter that for you as long as you’re in Visual
mode.


```vim  linenos:false
/ \(
```

The `/` here denotes the beginning of a regular expression, which is going to make up our
find pattern. It’s not relevant to this example, but it’s worth noting that you can choose
a character other than `/` if you have a regular expression that has `/` in it a lot, for
example a file path.

Then there’s a space, which matches the space before the beginnning of the column name in
our list.

Then there’s `\(`, which starts an “atom”, also known in other regular expression engines
as a “match group”. This allows us to base our replace text on the original text.


```vim  linenos:false
.\{-}
```

Now we’re getting into some regular expression fun. The `.` will match any character, and
the `\{-}` will match any amount of that character, but **as few as possible**. Otherwise,
the regular expression will chomp up as many characters as possible as long as the rest of
the regular expression is still satisfied, which isn’t what we want. For those of you
familiar with Perl-style regular expressions, this is a “lazy quantifier”, and is the
equivalent of `.*?` in other languages.


```vim  linenos:false
\)
```

This closes the atom that was started above. All the text that’s matched between this and
the preceding `\)` can be referred in the second half of the `:substitute` command as `\1`
(and `\2` for the second atom, etc.)


```vim  linenos:false
.*$/
```

This closes out the matching regular expression - first we have a space to make sure the
`.\{-}` construct matches until the very end of the word, then `.*`, the regular
expression for any amount of any character **as much as possible**, then `$` to match the
end of the line. Finally, the `/` character tells `:substitute` we’re starting the
expression for text to replace this with


```vim  linenos:false
\1 = :\1
```

Remember above when we were making the atom, and I said it can be referenced in the
replaced text as `\1`? Well we can actually do that as many times and wherever we like.
Here we’re using it both for the column name, and for the name of the named parameter.

Finally, `/` closes out the whole thing


Try running it yourself and playing around with things to see what happens. What if you
replace the lazy `\{-}` with the greedy `*`? What if you remove the space after the atom?
