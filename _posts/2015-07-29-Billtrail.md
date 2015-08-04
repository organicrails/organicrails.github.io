---
layout: post
title: Project 2 - BillTrail. You may be my friend, but hey, there's no such thing as a free lunch!
subtitle: MongoDB Integration w/ Ruby 2.1.3 and Rails 4.2.3
---

# [BillTrail REPO](https://github.com/organicrails/billtrail)

One thing that seems to occur among every social circle is that no one can keep track of who paid for how much when splitting the bill. Whether it's a simple restaurant check, or a grand total sum of a week-long road trip, keeping track of every individual spendings is pretty irritating. The most troublesome part, however, is definitely trying to sum up all the totals and to figure out just exactly how much each person owes. 

With that said, for Project 2, I've decided to create a simple application that can keep track of the total amount spent on a given bill by person. **This application will keep track of how much everyone pitch in on a given bill, sums up everyone's spending, and return how much each person should have paid when the bill is evenly splitted among the group.** 

And guess what? We're going to experiment using MongoDB for this project. 

If you have never heard of MongoDB, or have no idea what NoSQL means, [This article](http://searchdatamanagement.techtarget.com/definition/MongoDB) breaks down what MongoDB is, and [this other article](http://kerrizor.com/blog/2014/04/02/quick-intro-to-mongodb-in-rails) explains what NoSQL is. I suggest give both of them a quick read, as they are prtty informative. 

If you are looking for a [TLDR](http://kerrizor.com/blog/2014/04/02/quick-intro-to-mongodb-in-rails), MongoDB basically allows us to work with _collections_ and _documents_ rather than the standard database tables and rows. Documents are stored in JSON hashes, so anything that can be represented in JSON is valid. Furthermore, MongoDB is _schemaless_, which means there is no need to call a migration every time a new column is added to the database. With MongoDB, there are no requirements for the data structure within the documents. 

Feeling Ready? Let's get started.


### Installing HomeBrew and MongoDB

Before we start creating a new rails project, we need to install MongoDB on the system first. To make the steps as painless as possible, let's utilize [HomeBrew](http://brew.sh/) if you are on OSX, an insanely awesome package manager that takes care of installation for you. (if you're on Windows, [the official guide](http://docs.mongodb.org/manual/tutorial/install-mongodb-on-windows/) is the best resource I found for installing MongoDB on your systems.) Following the HomeBrew page to install HomeBrew.

After installing HomeBrew, we can then proceed with installing MongoDB. Open up your terminal and enter the following:

`brew install mongodb`

And you're done. Was that not ridiculously easy?

Based on the [same article as the TLDR above](http://kerrizor.com/blog/2014/04/02/quick-intro-to-mongodb-in-rails), we don't need to allow MongoDB to start whenever your system starts. Let's take control of our lives and simply ignore all the post installation messages from MongoDB. Instead use the following to start MongoDB at your will

`brew services start mongodb`

and the following to stop MongoDB 

`brew services stop mongodb`

**If your bew services command does not work**, please head to [this SO post](http://apple.stackexchange.com/questions/150300/need-help-using-homebrew-services-command) and following the SECOND (aka the non-accepted) answer by _kbrock_. I ran into the same issue and his solution fixed it for me. 

When everything looks safe and sound, let's start up our mongoDB server. You should see the following message "_==> Successfully started `mongodb` (label: homebrew.mxcl.mongodb)_"

Now we are ready to play around with MongoDB.

### Just The Basics

Now we are ready to create a new rails project. For this application, let's name it BillTrail. For BillTrail, we want to install everything but the active record portion of the application. Active Record is the Model in the MVC, but since we are using MongoDB as our primary database, we don't need the default SQLite3 database. If you are not familiar with the term, [check out this link](http://guides.rubyonrails.org/active_record_basics.html). Because of that, we are going to skip the active record when creating BillTrail. 

`rails new billtrail --skip-active-record`

Take a look at the _GEMFILE_, and you'll see the _sqlite_ is not part of the gem this time, which is what we wanted. For this application, we really only need one gem, _mongoid_, to use MongoDB in our app, but I also love throwing _HAML_ in there. Let's add both into our _GEMFILE_.

[Mongoid](https://github.com/mongodb/mongoid)is "an ODM (Object-Document-Mapper) framework for MongoDB in Ruby", which essentially allow us to use MongoDB in our ruby projects. [HAML](http://haml.info/) just makes everything looks **much** nicer, which I always appreciate. 

{% highlight ruby %}
gem 'mongoid', '~> 4.0.2' 
gem 'haml', '~> 4.0.6'    
{% endhighlight %}

After installing the gems above, run `rails generate mongoid:config` and Mongoid will create a _config/mongoid.yml_. Feel free to take a quick look, and take note of _database: billtrail_development_, which we will run across later. But we do not need to change anything about it at this point. 

Let's think about what our application needs first. Our most basic premise is probably a list of names, and how much money that person spent. To showcase MongoDB and its schemaless protocols, let's simply create a _Bill_ scaffold with the names first. 

`rails g scaffold Bill name:string`

Take a look at **_app/model/bill.rb_**

{% highlight ruby %}

class Bill
  include Mongoid::Document
  field :name, type: String
end

{% endhighlight%}

Notice anything different? If we were not using MongoDB, but just the default SQLite, it might look something like this instead

{% highlight ruby %}
class Bill < ActiveRecord::Base
  ...
end
{% endhighlight%}

Notice how with MongoDB, there is no need to call any migrations, because we did not create any table rows or columns. Instead, we created a **Bill Collection**, and within the collection we currently have a **name document**. To make sure everything is working correctly, let's redirect the root page and create a simple record. If you want a more detailed explanation, [check out this article](http://www.w3resource.com/mongodb/databases-documents-collections.php)

**_routes.rb_**
{% highlight ruby %}
Rails.application.routes.draw do
  resources :bills
  root 'bills#index'
end
{% endhighlight %}

Try to create a new Bill, enter a name, and everything should work flawlessly. After creating a new bill though, take a look at the URL 

`http://localhost:3000/bills/55bbe3f46a61638630000000`

As oppose to the usual

`http://localhost:3000/bills/1`

MongoDB does not keep track of the documents with ID in numerical increment orders. Instead, every document field within MongoDB has its own unique [ObjectID](http://docs.mongodb.org/manual/reference/object-id/), and in my case, my object ID for my name is "55bbe3f46a61638630000000". We'll do something pretty interesting with the ObjectIDs later, but for now, let's add the other fields to our Bill. To add additional documents to our Bill collection, all we need to do add the following two lines. 

{% highlight ruby %}

class Bill
  include Mongoid::Document
  field :name, type: String
  field :dollar, type: Integer
  field :cent, type: Integer
end

{% endhighlight%}

And that is it. Once again, no migration or anything is required. Mongoid will automatically the map newly added fields to their respective documents. 

You may be wondering why I separated the monetary values to its dollar and cent component. The reason is because [MongoDB does not have decimal support](http://stackoverflow.com/questions/11541939/mongodb-what-about-decimal-type-of-value). So if we want to store something in the form of "$DDD.CC", that is not allowed. Instead, let's just break it down to dollars and cents, and store them both as integers. 

To check that this is indeed all we need to do to add more fields to the Bill collections, let's fix up the following...
[Here is a great tool to convert ERB to HAML](http://html2haml.herokuapp.com/)

**_bills_controller.rb_**

{% highlight ruby %}

....

# add field to strong parameter

# Never trust parameters from the scary internet, only allow the white list through.
def bill_params
  params.require(:bill).permit(:name, :dollar, :cent)
end


{% endhighlight%}

**_views/bills/_form.html.haml_**

{% highlight ruby %}
= form_for(@bill) do |f|
  - if @bill.errors.any?
    #error_explanation
      %h2
        = pluralize(@bill.errors.count, "error")
        prohibited this bill from being saved:
      %ul
        - @bill.errors.full_messages.each do |message|
          %li= message
  .field
    = f.label :name
    %br/
    = f.text_field :name
  .field
    = f.label :dollar
    %br/
    = f.text_field :dollar
  .field
    = f.label :cent
    %br/
    = f.text_field :cent
  .actions
    = f.submit

{% endhighlight%}

**_views/bills/show.html.haml_**
{% highlight ruby %}
%p#notice= notice
%p
  %strong Name:
  = @bill.name
%p
  %strong Dollar:
  = @bill.dollar
%p
  %strong Cent:
  = @bill.cent
= link_to 'Edit', edit_bill_path(@bill)
|
\#{link_to 'Back', bills_path}

{% endhighlight%}

Now head over to `locahost:3000`, create a new bill, enter the name, dollar, and cent values, and you should see everything working fine. Awesome huh.

### MongoDB Shell
Similar to how rails have the _rails console_, it is not surprising that mongoDB has something similar. To utilize the mongoDB "shell", let's first take a look this line `database: billtrail_development` in our **_config/mongoid.yml_**. This line tells you how to access the MongoDB shell, which we will now call by entering

`mongo billtrail_development` 

in the command line. You should see `MongoDB shell version: 3.0.4 connecting to: billtrail_development` if it's successfully connected.

First thing first, let's check out our collections.

{% highlight ruby %}
  > show collections
  bills
  system.indexes
{% endhighlight %}

Great, seems like bills is indeed one of our collections. Next, let's take a look at our documents. The syntax is `db.[collection].find()`, but we will append the `.pretty()` method to make it format much nicer.

{% highlight ruby %}
  >db.bills.find().pretty()
  { "_id" : ObjectId("55bbe3f46a61638630000000"), "name" : "Jack" }
  {
    "_id" : ObjectId("55bbe71f6a61638630000001"),
    "name" : "Bob",
    "dollar" : 100,
    "cent" : 10
  }
  {
    "_id" : ObjectId("55bbe9286a61638630000002"),
    "name" : "Lucy",
    "dollar" : 98,
    "cent" : 23
  }
{% endhighlight%}

As shown above, I got three documents within my Bills collection. First one with name "Jack" is created before we added the dollar and cent fields, therefore it does not have those two fields. "Bob" and "Lucy" were both created after the addition of those two fields. Great, seems like everything is working like it should.

To exit the shell, call **CTRL+C**

### A Pivot in the Application

Now that we have a basic understand of MongoDB and how it plays in our rails application, let's rethink our application a bit. At this point, I realized that the documents within Bill doesn't make much sense structurally. Right now, we are simply throwing names and numbers inside the Bill collection without any purpose. What we want is perhaps something along the lines of a relationship model. We can have a _Bill_ with perhaps a _EventName_, and within that _Bill_ exists many _Transactions_ that keep track of the payer and the values. Ultimately, this is what we want to achieve 


{% highlight ruby %}
 {
    "_id" : ObjectId("55bbe71f6a61638630000001"),
    "Event_name" : "Dinner Bill",
    "Transactions" [{ 
                     "payer": "Bob",
                     "dollar": 10
                     "cent": 50
                     },
                     {
                      "payer": "Lucy",
                     "dollar": 32
                     "cent": 50
                 	 }
                   ]
 }

{% endhighlight%}

Let's refractor the code we have so far to fit the more ideal relationships above.


### Relationships in MongoDB

First thing first, let's simplify our index page. We don't need all that extra information. Let's just make it so that all we see is a *New Bill* option

**_views/bill/index.html.haml_**
{% highlight ruby %}
%p#notice= notice
= link_to 'New Bill', new_bill_path
{% endhighlight%}


Let's take advantage of how flexible MongoDB is at adjusting fields. In a RDBMS, there would be a need to update the `Bill.rb` model with new tables and columns and all that, but not with a NoSQL DB such as MongoDB. To update `Bill.rb`, let's first add an _event_name_ field, along with its validation for presence. Afterwards, let's embed a [1-N relationship for mongoid](http://mongoid.org/en/mongoid/docs/relations.html) for a new _transactions_ model. 

Note two things. One is how how similar the syntax is for defining relationships. In regular rails, we would use _has_many_, and in mongoid we use _embeds_many_. Second is that the validation for presence is the exact syntax. This is because Mongoid is awesome enough to [includes ActiveModel::Validations to supply the basic validation](http://mongoid.org/en/mongoid/docs/validation.html), so developers would feel a lot more comfortable working with it rather than adapting a whole new systems of syntax. 

**_app/models/bill.rb_**

{% highlight ruby%}
class Bill
  include Mongoid::Document
  field :event_name, type: String

  # 1-N relationships for transactions
  embeds_many :transactions

  validates_presence_of :event_name
end
{% endhighlight %}

Before we proceed with the creation of a **Transactions** model, remember to update your _bills_controller_, _view/bills/show.html.haml_, _view/bills/index.html.haml_, and _view/bills/_form.html.haml_ with the newly udpated *event_name* field


Now, with the declaration of `embeds_many :transactions`, let's create a transactions model with 
`rails g model Transaction` and proceed with the following

**_models/transactions.rb_**
{% highlight ruby %}
class Transaction
  include Mongoid::Document

  embedded_in :bill

  field :payer, type: String
  field :dollar, type: Integer
  field :cent, type: Integer

  validates_presence_of :payer, :dollar, :cent
  validates_length_of :cent, :maximum => 2
end

{% endhighlight%}

In the `Transaction.rb` model, we first establish a relationship with the `Bill.rb` model by declaring `embedded_in :bill`. Three new fields are then added: *payer*, *dollar*, and *cent*. Furthermore, we also valdiated the presence of those three fields, and limited the characters of *cent* to only two characters. 

Let's create the _controller_ and the _view_ now. 

For this application, let's simply show the form on the show page of each Bill, since we want to add a new transaction specific to each bill anyways. With that said, we only need a **create** and **destroy** method in the `Transactions.rb` controller.


`rails g controller Transactions`

**_app/controller/transactions_controllerrb_**
{% highlight ruby%}
class TransactionsController < ApplicationController

  def create
    @bill = Bill.find(params[:bill_id])
    @transaction = @bill.transactions.build(transaction_params)

    if @transaction.save
      redirect_to bill_path(@bill)
    else
      flash[:error] = "Something went wrong, please check and re-enter your fields again!"
      redirect_to bill_path(@bill)
    end
  end

  def destroy
    @bill = Bill.find(params[:bill_id])
    @transaction = @bill.transactions.find(params[:id])
    @transaction.destroy
    redirect_to bill_path(@bill)
  end

  private

  def transaction_params
    params.require(:transaction).permit(:payer, :dollar, :cent)
  end

end

{% endhighlight%}

The code above is pretty self-explanatory, just a simple nested resource's controller. 

And as for the view...

**_views/bills/show.html.haml_**

{% highlight ruby %}

%p#notice= notice
%h1.event_name
  %strong Event Name:
  = @bill.event_name
  = link_to 'Edit', edit_bill_path(@bill)


.add_transactions
  - if @bill.transactions.size > 0
    %h2 All Transactions
    - @bill.transactions.each do |transaction|
      %p
        %strong #{transaction.payer}
        paid $
        %strong #{transaction.dollar}
        \.
        %strong #{transaction.cent}
        for
        %strong #{transaction.purpose}
        = link_to "Delete", [transaction.bill, transaction], method: :delete, data: {confirm: 'Are you sure"'}
      

.add_transactions
  = form_for [@bill, Transaction.new] do |f|
    %p
      = f.label :payer
      = f.select :payer, options_for_select(get_friends_array)
    %p
      = f.label :dollar
      = f.text_field :dollar
    %p
      = f.label :cent
      = f.text_field :centb
    %p= f.submit "Add Transaction"

{% endhighlight %}

Last but not least, don't be like me and forget to declare your **Transactions** resources in your `routes.rb` (took me a while to figure out the error...). 

**_routes.rb_**
{% highlight ruby %}
Rails.application.routes.draw do
  resources :bills do
    resources :transactions
  end
  root 'bills#index'
end
{% endhighlight %}

Now head over to your index page, create an Event, input some values for the Transactions, and the show page should display the payers name and how much the payer paid!


### Define our own UUID. 

One nice little feature of using MongoDB is its internal use of unique IDs when declaring objects. Since each object has its own unique IDs, the users can simply note the object ID somewhere and retrieve the information they need on the website by recalling the IDs. I am a fan of UUID in general, since users can have quick access to their own pages, and it is much safer than incremental numbers that everyone can blindly guess. 

However, it is very unlikely that anyone is going to memorize the 24 characters in a MongoDB Object ID. Let's be real, even copy-pasting such long string of characters is too much to ask for some users. Let's change that (because we can). 

What we want is to change it from

a) **http://localhost:3000/bills/55bd6e316a616387f2000003**

to something much simpler, maybe something similar to

b) **http://localhost:3000/bills/55bd6e** 

or even 

c) **http://localhost:3000/55bd6e**

Or maybe even allow the users to declare their own UUID.

d) **http://localhost:3000/bobs_steak_dinner**

This way not only do users not have to memorize some random string, they can define their own IDs for future references. 

The best part is that this feature is ridiculously easy to implement through rails and Mongoid. Let's break down what we want into two simple (and easy to implement) parts.

1. Remove the /bills/ portion of the URL
2. Define and append user-defined UUID

To achieve part one, all we need to do is the following.

**_routes.rb_**
{% highlight ruby %}
Rails.application.routes.draw do
  resources :bills, :path => '' do
    resources :transactions
  end
  root 'bills#index'
end
{% endhighlight %}

the addition of `:path => ''` tells rails that when routing the 'bills' resources, simply ignore and remove the resource name from the path. The result is what we want in step C shown above. 


To achieve part two, let's start by adding a field **urlID** to the _Bill_ model. After establishing a **urlID**, the _Bill_ model will tell the application "hey, as oppose to keeping the default ID, let's modify and point the IDs at the newly defined **urlID**"

After that, add some simple validations to make sure **urlID** is unique (don't want duplications) and both fields are non-empty.

This is all easily achievable with the [following solution by styliii](http://stackoverflow.com/questions/4744446/mongo-ids-leads-to-scary-urls)

**_model/bill.rb_**
{% highlight ruby %}
class Bill
  include Mongoid::Document
  field :event_name, type: String
  field :urlID, type: String

  # creates a urlID that the users can refer to afterwards
  field :_id, type: String, default: ->{ urlID }

  # contains many transactions
  embeds_many :transactions

  # validates presence of event_name and urlID.
  validates_presence_of :event_name, :urlID

  #validates the uniqueness of urlID
  validates_uniqueness_of :urlID

end

{% endhighlight %}

As always, also remember to update your strong parameters within the bills controller and add the form fields in your bills view page. Below I slightly modified the labels for the form displayed to give a bit more information. It's not the most visually pleasing method of doing so, but it gets the point across for this project.

**_views/bills/_form.html.haml_**
{% highlight ruby %}
= form_for(@bill) do |f|
  - if @bill.errors.any?
    #error_explanation
      %h2
        = pluralize(@bill.errors.count, "error")
        prohibited this bill from being saved:
      %ul
        - @bill.errors.full_messages.each do |message|
          %li= message
  .field
    = f.label :event_name, "Enter Event Name"
    %br/
    = f.text_field :event_name
  .field
    = f.label :urlID, "Choose a Unique ID for the event: "
    %br/
    = f.text_field :urlID
  .actions
    = f.submit
{% endhighlight%}

Let's open up `localhost:3000` and see how the application works now. Let's create a new bill, enter your desired event name and URLID, and now the page should be redirected to `localhost:3000/your_urlID`. Pretty useful feature, I must say.

### A Second Pivot

Even though the application is working as intended, there is still major room for improvements (aren't there always). One aspect that can be further adjusted is the **name** field. Currently, the users are required to enter the name each time there is a new transaction. This is perfectly fine simply showing a list of transactions, where the slight differentiations in spelling might not matter much (Ex: Annie vs annie). 

However, at some point during development the application do want to return the total amount spend by each individual on a Bill. When summing up each transactions, we can either do a bunch of validation checks (make everything lowercase, check the spelling...), or we can simply create a drop-down menu for the name field, select one of the names from there, and set it as the name. This way, there is no need to worry about the different spellings or typos for a given name, and the users do not have to constantly re-enter the same names over and over again for different transactions. 


Let's refractor the code once again to achieve what we want above.

### Adding a Friend Model

To have a list of names to choose from, we first need to have a list. As each bill will have its own individual list of friends, let's begin by adding another `embeds_many` to the bill model. We'll call this embedded model **friends**

**_model/bill.rb_**
{% highlight ruby %}
class Bill
  include Mongoid::Document
  field :event_name, type: String
  field :urlID, type: String

  # creates a urlID that the users can refer to afterwards
  field :_id, type: String, default: ->{ urlID }

  # contains many transactions and friends
  embeds_many :transactions
  embeds_many :friends

  # validates presence of event_name and urlID.
  validates_presence_of :event_name, :urlID

  #validates the uniqueness of urlID
  validates_uniqueness_of :urlID

end

{% endhighlight%}

And let's proceed with creating the Friend model. 

`rails g model Friend`

All we need in the friend model are the names. Create the **name** field and remember to establish the _embedded_in_ relationship with the Bill.

**_model/friend.rb_**
{% highlight ruby %}
class Friend
  include Mongoid::Document
  embedded_in :bill

  field :name, type: String

  # ensures that name is not empty
  validates_presence_of :name

end
{% endhighlight%}

After creating the Friend model, let's create the controller next. The content of the Friends controller will be very similar to the Transactions controller, in which we only need the create and delete function. Both are nested resources within Bills, so don't forget to include `resources :friends` within the bills resources in **_routes.rb_**

`rails g controller Friends`

**_friends_controller.rb_**

{% highlight ruby %}
class FriendsController < ApplicationController

  def create

    @bill = Bill.find(params[:bill_id])
    @friend = @bill.friends.build(friend_params)
    if @friend.save
      redirect_to bill_path(@bill)
    else
      flash[:error] = "Something went wrong, please check and re-enter your fields again!"
      redirect_to bill_path(@bill)
    end
  end

  def destroy
    @bill = Bill.find(params[:bill_id])
    @friend = @bill.friends.find(params[:id])
    @friend.destroy
    redirect_to bill_path(@bill)
  end

  private

  def friend_params
    params.require(:friend).permit(:name)
  end
end

{% endhighlight%}

And add the corresponding form to model Bill's show page

**_views/bills/show.html.haml_**

{% highlight ruby %}

%p#notice= notice
%h1.event_name
  %strong Event Name:
  = @bill.event_name
  = link_to 'Edit', edit_bill_path(@bill)


%h2 Friends who Pitched In

- if @bill.friends.size > 0
  - @bill.friends.each do |friend|
    %p
      %strong Name: #{friend.name}
      = link_to "Delete", [friend.bill, friend], method: :delete, data: {confirm: 'Are you sure"'}

.add_friends
  = form_for [@bill, Friend.new] do |f|
    %p
      = f.label :name
      = f.text_field :name
    %p= f.submit "Add Friend"


.... # some other code


{% endhighlight%}

Now there is an option on the show page of each Bill to add a list of names of those who contributed to the Bill. After the names are saved, they will be displayed on the show page as well. Try it out on `localhost:3000`, everything should be working fine!

### Creating the Drop Down

Now that there exists a Friend model, we can put it to good use. The creation of a drop down is probably what I was stuck on the most when I was creating this project. I attempted many "rails way" to create the friends drop-down menu before I finally found a solution that works. I will try my best to post all the resources I used and retraced the steps. 

First things first, we need to (somehow) get access to the list of friends on our show page so that we can maybe choose from it. This immediately presents an initial problem of "where to put this method?". There are two obvious (though not quite what we're looking for) destinations. 

One is the within the **Friend Model**. This seems to make sense, since we are looking for a list of Friends. The problem (as I encountered) is that if the method does resides within the Friend Model, it will return an array of ALL friends regardless of the Bill, which is not what we quite want. We do indeed get an array back, but it is not limited to just the Bill that we are interested in. 

With that said, the second place is the **Friend Controller**. This appears to work initially. We can limit the friends array to only those that are within a specific **:bill_id**, so that no other names will show up instead of the ones in the current Bill. However, because the method is within the Friend Controller, it appears to only be to be displayed in the Friends Views (which we have none, as everything is displayed on the Bill show page). We have no Friends view pages, so I could not figure out a way to actually use the method.

Finally I decided to put the method in the **Bill Controller** page. This works because not only can we limit the array of friends to be just those within a specific **:bill_id** (solved issue #1), but the method can also be used within the bill show page since it resides within the Bills Controller(solved issue #2). 

Here is my implementation first, and I'll give the explanation after.

**_bills_controller.rb_**
{% highlight ruby %}
class BillsController < ApplicationController

  before_action :set_bill, only: [:show, :edit, :update, :destroy]

  helper_method :get_friends_array

  # GET /bills
  # GET /bills.json
  def index
    @bills = Bill.all
  end

  # OTHER DEFAULT CRUD METHODS
  ....

  # NON CRUD METHOD

  # returns an array that includes all the Friends who split bills
  def get_friends_array

    friends_array = Array.new

    bill = Bill.find(params[:id])
    friends = @bill.friends

    (0...friends.length).each do |index|
      friends_array << friends[index]["name"]
    end
    friends_array

  end



  private
    ....
end

{% endhighlight %}

the method _get_friends_array_ is relatively simple. Initially, create a new **friends\_array**. Find the corresponding Bill and get all the friends from that bill. To understand why there is a need for a new array as oppose to simply using array **friends**, it is important to note that **friends** takes on the following format (Here I populate friends with three random names). 

[#<Friend _id: 55bfcab26a616387f2000006, name: "Annie">, #<Friend _id: 55bfd3906a616389e8000000, name: "Bob">, #<Friend _id: 55bfd3936a616389e8000001, name: "Robert">]

In the **friends** array above, we see that although it contains information that we want (the names), it also returns the Friend \_ID along with it, which is not needed. To adjust this, we will populate the new array with only the names. The loop will iterate from 0 to however many friends there are. At each iteration, friends[index]["name"] will pull only the name from the current index wthin **friends** and add it to **friends\_array**. 

The result is that **friends\_array** will now contain only ["Annie", "Bob", "Robert"], which is exactly what we're looking for.


To be able to use the method from a controller in the view, we need to declare the method as a [Helper Method](http://apidock.com/rails/ActionController/Helpers/ClassMethods/helper_method), which I learned from [this post](http://stackoverflow.com/questions/8906527/can-we-call-a-controllers-method-from-a-view-as-we-call-from-helper-ideally). By declaring _get_friends_array_ as a helper method, it is now able to be accessed within Bill's show page inside the form. 

To actually get a drop-down menu within the form, I firsted tried [collection_select](http://apidock.com/rails/ActionView/Helpers/FormOptionsHelper/collection_select) as suggested by [this SO](http://stackoverflow.com/questions/6191352/rails-form-for-with-collection-select). However, _collection_select_is failed to function for me. Even after [reading this SO](http://stackoverflow.com/questions/8907867/can-someone-explain-collection-select-to-me-in-clear-simple-terms) I still could not get the parameters right. 

In the end, I found out about _options_for_select_ from [this post](http://stackoverflow.com/questions/19120706/rails-erb-form-helper-options-for-select-selected) and decided to give it a shot. And it worked perfectly. 

**_views/bills/show.html.haml_**

{% highlight ruby%}
# some other code...

.add_transactions
  = form_for [@bill, Transaction.new] do |f|
    %p
      = f.label :payer
      = f.select :payer, options_for_select(get_friends_array)
    %p
      = f.label :dollar
      = f.text_field :dollar
    %p
      = f.label :cent
      = f.text_field :cent
    %p= f.submit "Add Transaction"
{% endhighlight%}

If there already exists some names under Friends, the drop-down menu should already contained all the names within that Bill. If not, create some names and witness the magic yourself!

### Calculating Total Per Person

Now, let's allow the users to view the sum of per person, to get a general sense of how much money each person spent on the Bill. As with above, we will put the method inside the Bills Controller again. Let's name this method _get_individual_total_. Again, here's the code that I came up with and I'll provide the explanation in a bit!

**_bills_controller.rb_**
{% highlight ruby %}
class BillsController < ApplicationController

  before_action :set_bill, only: [:show, :edit, :update, :destroy]

  helper_method :get_friends_array
  helper_method :get_individual_total

  def index
    @bills = Bill.all
  end

  # OTHER DEFAULT CRUD METHODS
  ....

  # NON CRUD METHOD

  # returns an array that includes all the Friends who split bills
  def get_friends_array
    ....
  end

  # return the INDIVIDUAL amount of dollars and cents per person. 
  def get_individual_total

    friends_array = Array.new

    bill = Bill.find(params[:id])
    transactions = @bill.transactions

    totals = Hash.new { |hash, key| hash[key] = { dollar: 0, cent: 0 } }
    transactions.each do |t|

      totals[t.payer][:dollar] += t.dollar
      totals[t.payer][:cent] += t.cent

      if totals[t.payer][:cent] >= 100
        totals[t.payer][:dollar] += totals[t.payer][:cent] / 100
        totals[t.payer][:cent] = totals[t.payer][:cent] % 100
      end
    end

    totals

  end



  private
    ....
end

{% endhighlight %}

This one took me a while, mainly my attempt to figure out how to create a hash within a hash. To start things off, once again initiate a new array **friends\_array**. After that find the **bill** and the corresponding **transactions**. Before we continue, let's take a quick look at what my current **transactions** will return. (Once again I will use our three musketeers: Annie, Robert, and Bob.)

[#<Transaction _id: 55bfe8a86a616389e8000007, payer: "Annie", dollar: 100, cent: 50>, #<Transaction _id: 55bfe8ae6a616389e8000008, payer: "Bob", dollar: 12, cent: 59>, #<Transaction _id: 55bfe8b56a616389e8000009, payer: "Robert", dollar: 56, cent: 12>, #<Transaction _id: 55bfe8bb6a616389e800000a, payer: "Annie", dollar: 65, cent: 65>]

From the information above, it appears that the transactions timeline reflects the following

1. Annie Paid $100.50
2. Bob Paid 12.59
3. Robert Paid 56.12
4. Annie Paid 65.65

With this information, we want to extract the **dollar* and **cent** at each index, do some magic and sum them up to their **payers** current total sum, and return it on the screen for the user to see. We expect such information to show that

1. Annie Paid $166.15 Total
2. Bob Paid $12.59 Total
3. Robert Paid 56.12 Total

Let's continue with the code to see how that is achieved.

`totals = Hash.new { |hash, key| hash[key] = { dollar: 0, cent: 0 } }` is without doubt the most interesting (though most difficult) concept I learned during this project. When I first created this method, I wanted a way to call something similar to _method\_name[payer\_name][dollar]_ to return the dollar values, and similarly for the cent values. With that logic, I needed a hash within a hash, but I had no idea how to create that. [Here is my SO that got me to the right direction](http://stackoverflow.com/questions/31711304/ruby-add-a-nested-key-to-an-existing-hash?noredirect=1#comment51393980_31711304). 

Basically (to the best of my understanding), _Hash.new_ can [take different parameters](http://ruby-doc.org/core-2.1.1/Hash.html), and for my purpose the parameter needs to be a Hash itself. By passing `{|hash, key| hash[key] = { dollar: 0, cent: 0 }` as the parameter, it's declaring that **totals** to have two arguments, **hash** and **key** ([here is a great SO on blocks if you are still confused what they do](http://stackoverflow.com/questions/4911353/best-explanation-of-ruby-blocks)), in the form of _hash[key]_. These two arguments will form a hash, with **Hash** will representing the key, and **key** as the value. Within the **key** value, it will set the default values of 0 to both dollar and cent. 

This declaration means that I can now call the following: _totals[hash][dollar]_ and it will return a default value of 0 for both dollar and cent. Note how this is very similar to what I want above (_method\_name[payer\_name][dollar]_ ). 

Naturally, I now loop through all the transactions, and for each transaction **t**, I call upon `totals[t.payer][dollar]` and update its value with `t.dollar` and `t.cent`. The if statement simply ensures that all cents are corrected to be less than 100 cents, since if it's above it is consider to be a dollar. If _[t.payer]_ does not exist currently, it will be created. Otherwise, it will simply update the current values. 

In the end, we return the nested Hashes **totals**. Let's see what **totals** output using the same info as above (simply call the method in my view).

{"Annie"=>{:dollar=>166, :cent=>15}, "Bob"=>{:dollar=>12, :cent=>59}, "Robert"=>{:dollar=>56, :cent=>12}}

Which is exactly the kind of information that we want.  To display such information, add the following to the show page.


**_views/bills/show.html.haml_**

{% highlight ruby %}

%p#notice= notice
%h1.event_name
  %strong Event Name:
  = @bill.event_name
  = link_to 'Edit', edit_bill_path(@bill)

.glance
  %h2 At a Glance...

  - @bill.friends.each do |friend|
    %p 
      %strong= friend.name
      paid for a total of $
      %strong= get_individual_total[friend.name][:dollar]
      \.
      %strong= get_individual_total[friend.name][:cent]

#other code
....
{% endhighlight%}

It's not the most sophisticated solution since both values are integers, but it does display the necessary information in a clear and understandable manner. What's more important is that we get the individual totals!

### Finally, the Grand Total Sum

This is the last technical portion of this project, so let's finish it strong shall we? Last thing that we'd like to add to the application is for the users to be able to see the total amount of the current Bill. After all, with all the money splitting and spotting, it'd be nice to see just exactly how much a group has spent. 

First thing first, we need a function that sums up the all the dollar values and cent values in all the transactions. Let's create another function within the Bill controller.

**_bills_controller.rb_**

{% highlight ruby %}
class BillsController < ApplicationController
  before_action :set_bill, only: [:show, :edit, :update, :destroy]

  helper_method :get_friends_array
  helper_method :get_individual_total
  helper_method :get_grand_total

  ....

  # return the total amount of dollars and cents in this bill
  def get_grand_total
    friends_array = Array.new

    bill = Bill.find(params[:id])
    transactions = @bill.transactions
    total_dollar = transactions.inject(0) { |sum, t| sum + t.dollar }
    total_cent = transactions.inject(0) { |sum, t| sum + t.cent }

    [total_dollar, total_cent]
  end

  ....
end

{% endhighlight%}

Similar to the previous two functions, start off by initiating an array **friends\_array**, get the current **bill** and all **transactions** within that bill. Now, we can either loop through the **transactions** array and add up all the dollars and cents, or we can simply use a powerful ruby function: inject(). The following two lines utilizes the [inject() function](http://apidock.com/ruby/Enumerable/inject) so that we can easily sum up everything within the array. [This SO ](http://stackoverflow.com/questions/710501/need-a-simple-explanation-of-the-inject-method) an excellent explanation on what we are doing here with the inject method. If you are not familiar with the inject method, I strongly suggest a quick read.

After summing up the the dollar values and store it in **total\_dollar** and likewise for **total\_cent**, the function returns those two values in an array. Don't forget to add in another _helper\_method)_ and apply it to _get\_grand\_total_.

Now, remember that **total_cent** is the sum of all the cent values within a Bill, which means that the value is very likely to be greater than 100. Let's create a function to apply a quick fix. Since this function can potentially be applied universally, we do not need to create it in the controller. Instead, let's put it in the Bill Model.

**_models/bill.rb_**
{% highlight ruby %}
def self.to_currenty_syntax(dollar, cent)
  	if cent > 100
  		dollar+= cent/100
  		cent = cent%100
	end
	[dollar, cent]
  end
{% endhighlight%}

And finally, our show page.

**_views/bills/show.html.haml_**

{% highlight ruby %}
.glance
  %h2 At a Glance...
  %p
    The total amount spent on this bill is $
    - total_bill_sum = Bill.to_currenty_syntax(get_grand_total[0], get_grand_total[1])
    %strong= total_bill_sum[0]
    \.
    %strong= total_bill_sum[1]

  - @bill.friends.each do |friend|
    %p 
      %strong= friend.name
      paid for a total of $
      %strong= get_individual_total[friend.name][:dollar]
      \.
      %strong= get_individual_total[friend.name][:cent]

{% endhighlight%}

And that's it !!! After refreshing the page, you should be able to see the total amount spent on a Bill. With this function added, we came to the conclusion of this project.

Thank you very much for sticking with this tutorial, and once again I hope you learned something useful!

If you have any comments or questions, please don't hesitate to post them below. 

### Some Self Reflection
Second application done within two weeks, it's been a productive month. Throughout the development process, I kept on asking myself "should I attempt the underlying [subset-sum](http://www.geeksforgeeks.org/dynamic-programming-subset-sum-problem/) problem and calculate how much each person should pay/receive?" After all, it is the logical progression of the app. In the end, I chose not to for two reasons. 

First is that I actually did not plan on spending this much time creating and experimenting with adding all the sums and returning individual sums. My initial plan was to experiment with MongoDB as much as possible, play around with it, and share my experiences. I somehow got too caught up trying to make this project a more "complete" application, and the result is what we made (not complaining though! Glad that I learned _helper\_method_ and _inject()_ along the way!).

Second is that I believe there are much more appropriate places for the subset-problem, especially on another stack. MongoDB is not ideal for monetary calculations due to the Integer only limitation. If I do choose to dive into the next step, I am afraid that as oppose to only focusing on the subset-sum problem, I will also spend a great deal of time trying to work around the Integer limitation. 

With that said, let's save that challenge for next time.

Again, I apologize for the lack of testing for this one (but definitely next one!).

No CSS again. I am thinking of adding some simple bootstrap next time, so that hopefully it'll look better and not as plain. 
Thanks again for reading, see you next time! 
































