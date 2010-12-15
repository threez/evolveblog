---
layout: post
title: Hello World
tags: jekyll ruby git sass scss blog disqus
---

Two years ago i have written my own static page generator in ruby. At that time 
i have used a very big rake task that knows how to transform and compose 
textile and all other parts of the page. It was fully driven by easiness and
usability so that a user just has to edit files and folders. 

Versioning was not a problem, because everything was organized in files and
folders. Using git for this task was very easy to do. Just like this:

{% highlight bash %}
git commit -a -m "changes from #{Time.now}"
git push
{% endhighlight %}

After things are uploaded a [post receive](http://www.kernel.org/pub/software/scm/git/docs/githooks.html#post-receive) hook will compose the page on the
server (assuming that ruby and [redcloth](http://redcloth.org/) are installed).

There where actually some erb templates for things like for loops around some
of the data. These for loops where pre filled with functions from the rake task.

I was thinking of using this old rake task to build my first blog page. But
i faced some major issues:

- No dynamic data. I would have to write
  ruby functions to provide every data i need during the page composition.
- The lack of sass support
- The predefined folder structure for layouts and pages
- The compile overhead (execute rake all the time)
- The page gets recompiled in the whole (no partial updates)
- Syntax highlighting for ruby, html, js, ...
- No meta data support
- Commenting would be not very easy on static sites 
  (spam, comment voting, comments on comments, ...)

I decided to not fix the old site generator. I wanted something fresh new
and compatible with my new work flows. Implementing a full blown application
stack with [sinatra](www.sinatrarb.com) or [rails](http://www.rubyonrails.org) 
seemed like a little overkill. So i search for static site generators on google.
I compared them and found [Jekyll](https://github.com/mojombo/jekyll) to be a 
perfect fit for my Problem.

But Jekyll has two issues that it is sharing with my old solution:

- Commenting
- Lack of Sass support

Both have a sweet solution. The commenting part can be easily done using
[disqus](http://disqus.com/). This is a platform for commenting and getting
feedback from your readers. It has some need features like [openid](http://openid.net) login and
such. It is very easy to integrate. For me it was just as easy as putting these
lines at the end of my post:

{% highlight html %}
<div class="comments">
  <a href="http://toevolve.org{{ page.url }}#disqus_thread">
    View Comments
  </a>
  <div id="disqus_thread"></div>
</div>
<script type="text/javascript" 
        src="http://disqus.com/forums/vilandgr/embed.js"></script>
{% endhighlight %}

The second problem can be easily addressed using the plugin infrastructure.
There are three types of [plugins](https://github.com/mojombo/jekyll/wiki/Plugins) 
in Jekyll. I used the *Converter* to make it aware of scss files. One has to 
keep in mind, that the scss file has to have a YAML Front Matter in the first lines
of the scss file. The YAML Matter is the meta data YAML structure at the head
of the document.

{% highlight ruby %}
require "sass"

module Jekyll
  class ScssConverter < Converter
    safe true
    priority :low

    def matches(ext)
      ext =~ /scss/i
    end 

    def output_ext(ext)
      ".css"
    end

    def convert(content)
      engine = Sass::Engine.new(content, :syntax => :scss)
      engine.render
    end
  end
end
{% endhighlight %}

My scss file starts like this:

{% highlight css %}
---
layout: nil
---

* {
  margin: 0px;
  padding: 0px;
}
{% endhighlight %}

Adding posts is easy. I just commit them in my local git repo and push the changes
to the server where a post commit hook will compose the page. Therefore the remote
repository can't be +bare+ and must have the current contents (that's why i am 
reseting to the head after adding new content). If you are interested in my git 
post receive hook, here it is:

{% highlight bash %}
cd ~/evolveblog.git/
GIT_DIR=~/evolveblog.git/.git git reset --hard HEAD
jekyll ../www/
{% endhighlight %}

So if there are new issues with Jekyll that i face while im blogging, i will tell you.
But for now this is enough about my setup.