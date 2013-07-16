---
layout: post
title: Sharding with DataMapper
tags: sharding database ruby datamapper rails
---

I have recently worked on a project, were i need to shard around multiple databases. The setting was *rails3* with *DataMapper*. Luckily *DataMapper* plays well with multiple repositories out of the box. But during the implementation i faced some caveats. 

First of all lets shortly speak about the architecture and a main use cases. In this example we have a master database with a users table and this user table contains the user id, password, etc. Most importantly this db stores the shard that will be used for the user as well. In our case this is just a symbol or name. One important fact is, that the number of shards is unknown to us, which will lead to more flexibility in the operations part. Some examples for rails are about notes and this is no exception. We assume 300 Mio. different users of our application and therefore we shard the notes on different database hosts.

The code we want to write in our *NotesController* should be familiar. We don't want do do any extra work to use a different database. Absolutely this is not a complete example. I just focus on the sharding parts.

{% highlight ruby %}
class NotesController < ApplicationController
  def index
    @posts = Note.find(:user_id => @user.id)
  end
end
{% endhighlight %}

Therefore we have to connect or use an existing connection to the shard of the user. I assume that the user is already logged in and and has a session which has stored the shard location and we can recover a user object using *user_from_session*. If there is a user we can wrap our call in the database repository context of *DataMapper*. But we have to make sure, that the connection to the shard is there, therefore we will introduce a simple helper module *Database*.

{% highlight ruby %}
class ApplicationController < ActionController::Base
  around_filter :shard_connection

private

  def shard_connection(&block)
    if @user = user_from_session
      Database.shard(@user.shard, &block)
    else
      block.call
    end
  end
end
{% endhighlight %}

Okay now *DataMapper* has multiple problems we have to avoid. First of all it assumes to define all definitions like *properties*, *validations*, *relations* in the default repository (which is in this example the master repository were the user lives in). But the second thing is, that it wants you to specify the name of a storage ahead of time. Since we don't know how many and which stores we have, we have to use a little trick to come around this. We will define de table name using our own custom lookup table *Database::TABLES*. The rest can be simple and common *DataMapper* stuff. The resulting model class looks like this:

{% highlight ruby %}
class Note
  include DataMapper::Resource
  Database::TABLES[default_storage_name] = "user_posts"
  repository(:default) do
    property :id,       Serial
    property :user_id,  Integer
    property :title,    String, :length => 100
    property :content,  Text,   :length => 10.kilobyte
    # more ...
    
    has 1, :user 
    # more ...
    
    validates_presence_of :title, :content, :user_id
    # more ...
  end
end
{% endhighlight %}

Now we can have a look at the heart of the sharing. It's a simple module that has multiple tasks.

- Do the table name resolution for each new repository since we don't know them in advance
- create new connections
- only connect once to a shard per rails instance
- look up the shards connection string

To do the table name trick we use the *resource naming conventions* one can attach to an adapter. Since the adapter will be returned after setting up a connection we can simply set the new convention there. Our convention is called *NAMING_CONVENTION* and will use the name of the mapping table if it is defined,  otherwise it will fallback to the *DataMapper* defaults convention.

The *url* method takes care of creating the connection string, i have used a template in which the shard name will be passed. But you can use what ever you want to lookup the connection string.

The shard method implementes the connection setup and uses the passed *block* to execute the real application logic in the context of the users shard.

{% highlight ruby %}
module Database
  ADAPTERS = {}
  TABLES = {}
  # this is the datamapper default
  DEFAULT_NAMING = DataMapper::NamingConventions::Resource::UnderscoredAndPluralized
  # the naming convention will be cached per repo so don't bother using a lambda
  NAMING_CONVENTION = lamdba do |name|
    TABLES[name] ? TABLES[name] : DEFAULT_NAMING.call(name)
  end
  
  def self.shard(name, &block)
    unless ADAPTERS[name]
      # connect with the new repository
      ADAPTERS[name] = DataMapper.setup(name, url(name))
      ADAPTERS[name].resource_naming_convention = NAMING_CONVENTION
    end
    
    # use the existing connection to the db shard
    DataMapper.repository(name, &block)
  end
  
  def self.url(name)
    # get the connection url on demand ...
  end
end
{% endhighlight %}

Rspec testing is easy too. Just setup an *around(:each)* proc and use the shard connection there. So thats it for know. I hope you find this useful.
