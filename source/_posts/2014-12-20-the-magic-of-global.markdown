---
layout: post
title: "The magic of :global"
date: 2014-12-20 00:05:32 -0500
comments: true
categories: vim

---

One of my non-technical coworkers recently came by my desk with a problem - she
had [this list][1] of zip codes in Long Island, and needed me to write a SQL
query to fetch a list of users who had zip codes in that list.

<!--more-->

The problem at hand, then, was that I needed to translate the HTML table source
of that page to a comma-separated list of single-quoted values to be passed to a
SQL IN statement, like so:

```sql
('11930', '11701', '11708', -- ...
```

I briefly considered whipping out a [pry][2] shell and [nokogiri][3], which is
my normal go-to solution for this sort of one-time web scraping task. However, I
decided an exclusively vim-based solution was better suited to this particular
problem.

I used `curl` to download the full source of the page, then opened it up in vim.
Fortunately, the HTML was poorly-minified, so every `<tr>` containing a single
zip code entry was on its own line. This kind of task is where the vim
[`:global`][4], and its counterpart `:vglobal` (the v is mnemonic for
in**v**erse) commands really shine.

So let's get to the solution. The following is my first version of a vim regular
expression for one of these zip codes:

```vim
/[0-9]\{5\}/
```

Which just matches exactly five digits in a row. With that regular expression,
this command:

```vim
:v/[0-9]\{5\}/d
```

Will delete every line that does **not** contain a string matching the regular
expression. The `v` tells Vim to operate on every line not matching the
following regular expression (`g` would operate on every line that **did**), and
the `d` tells Vim to delete those lines. This isn't quite the solution, however,
since on this particular web page it'll still leave several lines that have long
strings of numbers, such as the following Google Publisher Tag line:

```html
<div id='div-gpt-ad-1326570816893-0' style='width:300px; height:250px;' class="display_ad">
```

in addition to several others. The second version of the regular expression
addresses this problem:

```vim
/\<[0-9]\{5\}\>/
```

The `\<` and `\>` sequences in a Vim regular expression are signifiers for the
beginning and end of a [word][6], which is a special concept in Vim and refers
to the same text objects that big-case [`W`][7] and [`B`][8] in Normal-mode move
over. With that regular expression, this command:

```vim
:v/\<[0-9]\{5\}\>/d
```

will get the desired output. After that, it's just a matter of clearing the
whitespace at the beginning of the line and the close `</td>` at the end of the
line with

```vim
:%norm dwf<d$
```

Keep in mind that the [`norm`][9] command executes its arguments as if they
were run in normal-mode, and the `%` at the beginning will do that to every
line.

Now we have a file containing a zip code on every line, with some lines
containing a comma-separated list of zip codes. We can split those lines such
that every line has only one zip code with

```vim
:%s/, /\r
```

(in vim, the escape sequence for a newline is `\r`)

The next step is to surround each zip code with single quotes. The vanilla vim
solution would be to use a [`:substitute`][10] [command][12], but I much prefer
to use [surround.vim][11] for this - then, we can just use

```vim
:%norm ys$'
```

Then, we put a comma after every line with either

```vim
:%s/$/,/
```

or

```vim
:%norm A,
```

and join all the lines together with

```vim
:%j
```


[1]:  http://www.longisland.com/zip-codes.html
[2]:  https://github.com/pry/pry
[3]:  https://github.com/sparklemotion/nokogiri
[4]:  http://vimdoc.sourceforge.net/htmldoc/repeat.html#multi-repeat
[6]:  http://vimdoc.sourceforge.net/htmldoc/motion.html#word
[7]:  http://vimdoc.sourceforge.net/htmldoc/motion.html#W
[8]:  http://vimdoc.sourceforge.net/htmldoc/motion.html#B
[9]:  http://vimdoc.sourceforge.net/htmldoc/various.html#:normal
[10]: http://vimdoc.sourceforge.net/htmldoc/change.html#:substitute
[11]: https://github.com/tpope/vim-surround
[12]: http://blog.griffinsmith.me/post/2014/08/04/an-introduction-to-the-vim-substitute-command/

