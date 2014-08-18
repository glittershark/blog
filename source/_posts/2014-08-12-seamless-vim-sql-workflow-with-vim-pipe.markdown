---
layout: post
title: "Seamless vim SQL workflow with vim-pipe"
date: 2014-08-15 22:21:30 -0400
comments: true
categories: vim

---

In case it's not obvious at this point, I really, really like vim. I like it so
much, in fact, that any time I'm editing text and I can't use vim I end up
spending about half my time pulling my hair out and cursing loudly at the
computer. To get around this, I've managed to inject it into about everything I
do. I use [i3](http://i3wm.org/) as my window manager, configured so that I can
move, resize, and switch between all my windows using Vim cursor keys. I use
[pentadactyl](http://5digits.org/pentadactyl/) in firefox, which revamps the
whole user interface so that everything can be done using Vim commands and
shortcuts. And I `set -o vi` in my shell so that I can edit command line
commands using Vim keys.

Another thing I do to make it so I never have to use anything that isn't vim is
edit all temporary SQL commands in vim, and run them all from there. This allows
me to run SQL statements directly from code, and also to use the output to help
me write the code itself (see [my post about the vim substitute
command](http://blog.griffinsmith.me/post/2014/08/04/an-introduction-to-the-vim-substitute-command/)
for an example). To accomplish this, I use the
[vim-pipe](https://github.com/krisajenkins/vim-pipe) plugin by krisajenkins.
Here's how to get that set up, and also some tips and tricks on how to use it
best.

<!--more-->

Firstly, you should really be using
[Vundle](https://github.com/gmarik/vundle.vim) to manage your Vim plugins, and
not Pathogen. There are a lot of reasons for this, but that's a topic for
another post.

If you are using Vundle, installing vim-pipe is as simple as adding the
following line to the **Plugins** section of your `vimrc`:

```vim
Plugin 'krisajenkins/vim-pipe'
```

and then running the `:PluginInstall` command from within Vim


If you're intent on using Pathogen anyway, you can install vim-pipe using the
following command:

```bash
$ git clone 'git://github.com/krisajenkins/vim-pipe.git' ~/.vim/bundle/vim-pipe
```

If you're using PostgreSQL like me, there's one more plugin you can install
that'll further improve this process.
[vim-postgresql-syntax](https://github.com/krisajenkins/vim-postgresql-syntax),
by the same author as vim-pipe, provides Vim syntax highlighting for
Postgresql's output, which is very nice.

As above, if you're using Vundle you can install this plugin by adding the
following line to your vim plugin list:

```vim
Plugin 'krisajenkins/vim-postgresql-syntax'
```

and then running `:PluginInstall` from within vim.

Or, if you're on pathogen, you can install it by running the following command:

```bash
$ git clone https://github.com/krisajenkins/vim-postgresql-syntax ~/.vim/bundle/vim-postgresql-syntax
```

--------

Once you've got everything installed, you need to tell vim-pipe how to run SQL
files. With vim-pipe, you can do this by putting the executable to run in the
`b:vimpipe_command` variable. So, if you have a postgresql server running
locally, a user 'myuser' and a database 'mydb', you can accomplish this by
putting the following line in your `.vimrc`:

```vim
autocmd FileType sql let b:vimpipe_command="psql mydb myuser"
```

To configure vim-pipe to set the `filetype` for the output buffer to
`postgresql` so it can be handled by the syntax highlighting script we installed
previously, you have to set the `b:vimpipe_filetype` variable. This can be
accomplished with another autocmd, like so:

```vim
autocmd FileType sql let b:vimpipe_filetype="postgresql"
```

Usually, however, you won't have a postgres server running locally on your
machine. Usually it'll be on a server or inside a dev machine. This approach
will still work for that, though, by using the following line in your vimrc:

```vim
autocmd FileType sql let b:vimpipe_command="ssh you@yourserver.com 'psql mydb myuser'"
```

It can also be helpful to be able to send SQL scripts to multiple different
servers, for example one on a development virtual machine and one on a
production server. I accomplish this by making commands to switch between them.
They look like this:

```vim
comamnd! SqlDevel let b:vimpipe_filetype="ssh me@virtualmachine 'psql mydb myserver'"
comamnd! SqlProd  let b:vimpipe_filetype="ssh me@myserver.com   'psql mydb myserver'"
```

I find this in combination with the autocmds for the default filetype to provide
an excellent workflow for selecting some data from production, using that data
to modify it on devel, and then running the same query on production. In
addition, this entire approach can be especially useful when copying the output
of the postgres `COPY TO STDOUT` command to the input of the `COPY FROM STDIN`
command. The possibilities are numerous.

Have fun!

