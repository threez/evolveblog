---
layout: post
title: Tmux is awesome
tags: unix tmux server-management
---

From time to time you have to manage a bunch of servers. Managing 30 servers, that can change their ip address or/and domain name can be quite hard. In a very dynamic setup it is hard to setup some sort of static connection to all of those hosts.

tmux to the resque is a great tool for this type of problem. Tmux is a terminal multiplexer (multiple virtual terminals in one real terminal) that makes scripting very easy. The main concept one needs to understand is the structure of tmux to really leverage its power.

The first entity is the session. Generally, the simplest case to create a session is like this:

    tmux

The created session usually has a very simple name, a number starting at 0 counting up. Since tmux has a client server infrastructure, the session doen't end when the connection of the client is lost. So if you close the terminal the session is still alive. To see all running sessions type:

    tmux ls

This command also tells you if there are any sessions that don't have a client. Sessions can be identified, by searching for lines that don't have the `(attached)` text at the end of the line. THe first name in the line is the session name. To attach to a session type:

    tmux attach -t 1

Where `1` is the name of the session.

In a session the next building block is the window. Usually the bottom line shows you all available windows. It behaves very much like a tab browser. Each window can have an identifier (the name), that is displayed in the bottom bar.

To create a new window type inside of a running tmux session:

    Ctrl-b c

All tmux commands begin with `Ctrl-b`, followed by another single character. The new session will normaly called after the shell that you are using. To rename the window type:

    Ctrl-b ,

The session can also be renamed later using `Ctrl-b &`.

To move between windows use either `Ctrl-b w` to select one from a list or use `Ctrl-b <n>` where n is the number of the window. Sometimes going to the next window `Ctrl-b n` or the previous window `Ctrl-b p` is very handy.

Once you have windows you can start using panes. Pane split a window into multiple sections. These sections can realy be everywhere. To create horizontal panes use `Ctrl-b %` to create vertical panes use `Ctrl-b "`. Switch between the different panes using `Ctrl-b o`.

Of course all these shortcuts can be changed and this is just the default configuration.

But the real fun begins, where the defaults end. Tmux has a really powerful and really nice way to automate things. Basically you can send commands to a session or the current session. Here a simple example, type the following in a already open tmux session:

    tmux new-window

To target a session use (in this example the session is calles 1):

    tmux new-window -t 1:

The colon seperates the session from the window. So in order to remove for example the 3 window in the session 1 we would type:

    tmux kill-window -t 1:3

And i guss you now start to understand the power of tmux. Because nearly every action is possible with a tmux command, it is highly scriptable.

To come back to the original problem. I solved this login using tmux and scripting. I wrote a simple linux script that works like this:

1. Create a new session
2. Create new windows in that session, with every server being one window (while creating name the windows accordingly
3. Attach the current process to the session

This way i can connect to a complete server farm using a simple script and switch between the different windows easily.

Now here the script essence:

{% highlight ruby %}
require 'securerandom'
session = ARGV.shift || "#{SecureRandom.hex(2)}"
system "tmux -2 new-session -d -s #{session}"
Instance.all do |instance|
  system "tmux new-window -t #{session}: -n '#{instance.name}'" +
         "'ssh -oStrictHostKeyChecking=no root@#{instance.ip}'"
end
exec "tmux -2 attach-session -t #{session}"
{% endhighlight %}

For a more static setup, for example my working environment, i use [teamocil](http://teamocil.com/). Which is very handy to create a rebuildable working environment.
