---
layout: post
title: Writing Good Shell Scripts
date: 2019-03-19
---

One of the typical tasks in system automation is writing scripts. Using the
shell is a popular choice, given you're likely to already know it (since you
interact with it every time you log in!) and also given its apparent simplicity.

However, the simplicity of the shell is quite misleading, since it's very easy
to get things just **wrong**. So what can you do about it?

## When should I write shell scripts?

**NEVER!** Ideally, you should avoid shell scripts. They are error-prone. It's
hard to test them. It's easy to break them. It's easy for things to go wrong in
unintended ways.

Prefer using a higher level scripting language, one that would give you more
structure and tools to help manage the complexity. While some scripting
languages are better than others, any of them is better than shell! My personal
choice is Python, but other very nice choices are Lua or Ruby. Even JavaScript
is possible! Perl is a bit dated these days... But if you're proficient with it,
well it can still make for better code than shell.

Some argue that there's nothing wrong with a short, 10 to 20 line shell
script... But, inevitably, requirements change and scripts start growing to take
on new roles. At some point, it becomes obvious that that old script needs a
rewrite in a better scripting language. But at that point, the beast has already
grown to be unmaintainable and has enough complexity that a rewrite becomes
quite costly. (Particularly since shell scripts are typically hard to test,
rewriting them becomes hard since you don't have automated test cases to
validate the rewrite!)

So the best advice here is to avoid shell scripts from the start! Pick a nice
scripting language, learn it well enough to get some proficiency and use it for
your scripting needs. You won't regret it!

## Ok, but I really want/need to write shell scripts... How can I make them better?

Ok, so here is some advice you can use to improve the quality of your shell
scripts. Beware, even if you adopt these, there are many dark corners where bugs
will hide. You will eventually regret not taking the previous advice... Having
said that, here we go!

### Break on errors

Make your scripts break when an error is encountered.

By default, if something goes wrong, the shell will simply print an error
message and **keep running the next command**. That is terrible, you really want
to stop running when something goes wrong, otherwise it's really hard to figure
out what happened.

In order to enforce that, start every shell script with the following three
commands:

```shell
set -o errexit
set -o nounset
set -o pipefail
```

The first one, `errexit`, breaks the script execution whenever a command exits
with a "false" (i.e. non-zero) return code. Programs typically return non-zero
when there's an error condition, so in most cases you **want** to break when
that happens.

The second one, `nounset`, breaks the script if a variable hasn't been declared.
This helps you find typos, where you think you're referring to a variable using
an incorrect name. By default, the shell will just replace the non-existing
variable with an empty string. Setting `nounset` makes it break execution, which
should help you figure out that you have a typo on that place.

These first two help catch errors, but on the other hand you might need to work
around some of their side-effects...

For instance, some commands are expected to return non-zero even on conditions
that are not supposed to cause the program to break. For instance, `grep`
returns 1 whenever it does not find the expression that it is looking for.

One way to prevent the script from breaking in those cases is to use the command
inside a conditional expression, such as `if`. For example:

```shell
if grep "^${username}:" /etc/passwd ; then
    echo "Found ${username}, it's a valid user."
fi
```

A shortcut for conditional expressions is the `||` operator, which can be used
for when a command returns non-zero, so it can make a good error handler,
especially combined with the `{ }` construct:

```shell
grep "^${username}:" /etc/passwd || {
    echo "No such user ${username}." >&2
    username=${fallback_username}
}
```

Another good shortcut is using `||` together with `:`, in cases where the
non-zero return is to be silently ignored:

```shell
# Print an entry for ${username} in the user database, if one exists.
grep "^${username}:" /etc/passwd || :
```

The `:` command is a command that does nothing, and always succeeds.

The use of `nounset` helps prevent typos, but on the other hand it can make it
harder to use variables that you're not really setting... For instance, many
scripts depend on variables coming from the _external_ environment, which is a
perfectly valid use case.

In order to cope with those, the recommendation is to set them to a default
value (possibly empty) at the start of your script. That way, when you refer to
them later, they will have been defined and `nounset` won't trigger.

A good way to accomplish that is to use the `:` command with variables as
arguments, and then using the `${var=value}` form, which sets `${var}` to the
value if it's unset at that point. Also possible is the `${var:=value}` form,
which sets `${var}` to the value if it's set *or* empty.

```shell
# Use vi as the default editor.
: ${EDITOR:=vim}

# Set ${DEBUG_LOG} in the environment to a non-empty value to enable debugging.
: ${DEBUG_LOG=}
```

The third one, `pipefail` makes execution break when a command that is a part of
a pipeline fails, which is usually desirable since there's not really anything
special about those commands that you'd want to ignore when they fail!

The `pipefail` setting also covers command substitution (running a command under
`$(...)` and using its output, the backticks syntax is also possible), which is
nice!

It is rare that `pipefail` causes unintended side-effects, but if it does, the
same techniques used for `errexit` can typically be employed here too.

There is one shortcoming of `pipefail` that it is good to know!

While it covers errors in execution of command substitution while running other
commands, it does not cover command substitution on variable assignment that is
using a modifier such as `local`.

The command below will not break when the user is not found, even under
`errexit` and `pipefail`:

```shell
find_user() {
    # Wrong!
    local pwent=$(grep "^$1:" /etc/passwd)
    echo "${pwent}"
}
```

Instead, break it down into a declaration and a separate assignment.

```shell
find_user() {
    local pwent
    # This correctly breaks (returns from find_user) if the user is not found.
    pwent=$(grep "^$1:" /etc/passwd)
    echo "${pwent}"
}
```

I hope you now have a better understanding of the error settings of shell
scripts. Employing them in your scripts will help make them slightly more
robust... Though, definitely not as much as they could be if they were written
in a better scripting language!

Next time we'll discuss quoting!

