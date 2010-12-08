---
layout: post
title: Properties for ruby
tags: ruby meta-programming properies
---

Recently i have worked on a Problem that requires default values in classes.
Guessing that we have a class like this:

{% highlight ruby %}
class MailAccount
  attr_accessor :host, :protocol, :port
  
  def initialize(host, protocol, port)
    @host, @protocol, @port = host, protocol, port
  end
end

imap = MailAccount.new("imap.example.com", :imap, 145)
pop = MailAccount.new("pop.example.com", :pop, 110)
{% endhighlight %}

There is an indirekt dependency between port and protocol. The protocol has a
dpendency to the corresponding port. So per default port *145* mapps to *IMAP*
and *110* to *POP3*. But these dependencies are not modeled. I was wondering if
i could setup a default port by creating a mapping, which looks like this:

{% highlight ruby %}
class MailAccount
  attr_accessor :host, :protocol, :port
  DEFAULT_PORT = {
    :imap => 145,
    :pop => 110
  }
  
  def initialize(host, protocol, port = nil)
    @host, @protocol, @port = host, protocol, (port || DEFAULT_PORT[protocol])
  end
end

imap = MailAccount.new("imap.example.com", :imap)
pop = MailAccount.new("pop.example.com", :pop)
{% endhighlight %}

I likeed the idea, but the problem was, that i can't and don't want do use the 
initializer for default values. I wanted them lazy evaluated, so that when i
change the protocol, and i did't set the port, i get the default. The second
solution:

{% highlight ruby %}
class MailAccount
  attr_accessor :host, :protocol
  attr_writer :port
  DEFAULT_PORT = {
    :imap => 145,
    :pop => 110
  }
  
  def initialize(host, protocol, port = nil)
    @host, @protocol, @port = host, protocol, port
  end
  
  def port
    @port || DEFAULT_PORT[@protocol]
  end
end

imap = MailAccount.new("imap.example.com", :imap)
pop = MailAccount.new("pop.example.com", :pop)
{% endhighlight %}

Next thing to do was, to handle the translation of values in the class. So that
stings are converted to integers or symbols.

{% highlight ruby %}
class MailAccount
  attr_accessor :host
  attr_reader :protocol
  DEFAULT_PORT = {
    :imap => 145,
    :pop => 110
  }
  
  def initialize(host, protocol, port = nil)
    @host = host
    self.protocol = protocol
    self.port = port
  end
  
  def protocol=(protocol)
    @protocol = protocol.to_sym
  end
  
  def port=(port)
    @port = port.to_i if port
  end
  
  def port
    @port || DEFAULT_PORT[@protocol]
  end
end

imap = MailAccount.new("imap.example.com", "imap")
pop = MailAccount.new("pop.example.com", "pop")
{% endhighlight %}

I think one can see, where this is leading. Things get bloated very fast and you
have to change the declaration of the attribute accessors every time. So i have
written a little meta class that is helping with this problem, because I'm facing
it with many classes.

{% highlight ruby %}
# simple version of with properties
module WithProperties
  def property(name, options = {})
    attribute_name = "@#{name}"
    property_get(name, attribute_name, options)
    property_set(name, attribute_name, options)
  end
  
  # define the getter, with default value as simple object or lamdba/proc
  def property_get(name, attribute_name, options = {})
    if dval = options[:default]
      define_method(name) do
        unless val = instance_variable_get(attribute_name)
          val = dval.respond_to?(:call) ? dval.call(self) : dval
          instance_variable_set(attribute_name, val)
        end
        val
      end
    else
      attr_reader name
    end
  end
  
  # define the setter, convert types if neccessary
  def property_set(name, attribute_name, options)
    case options[:type]
    when :string
      define_method("#{name}=") do |val|
        instance_variable_set(attribute_name, val.nil? ? val : val.to_s)
      end
    when :int
      define_method("#{name}=") do |val|
        instance_variable_set(attribute_name, val.nil? ? val : val.to_i)
      end
    when :symbol
      define_method("#{name}=") do |val|
        instance_variable_set(attribute_name, val.nil? ? val : val.to_sym)
      end
    else
      raise ArgumentError.new("The property type #{options[:type]} is unknown!")
    end
  end
end


class MailAccount
  extend WithProperties
  DEFAULT_PORT = {
    :imap => 145,
    :pop => 110
  }
  property :host, :type => :string
  property :port, :type => :int, :default => lambda { |r| DEFAULT_PORT[r.protocol] }
  property :protocol, :type => :symbol
  
  def initialize(attributes = {})
    attributes.each { |name, value| self.send("#{name}=", value) }
  end
end

imap = MailAccount.new(:host => "imap.example.com", :protocol => "imap")
pop = MailAccount.new(:host => "pop.example.com", :protocol => "pop")
{% endhighlight %}

Really what I was asking myself was: Is this overdone? I have around 15 classes
that all use this approach. Is it fast? The conversion and lambdas may take more
time than the original attribute accessors. Is is right? I mean you put in 
something of type A and get back something of type B. Is it better to always 
cast the value outside the class? For now i will stick to this solution. But 
i will blog if i encounter problems.

In the end there is much code for just three variables huh?
