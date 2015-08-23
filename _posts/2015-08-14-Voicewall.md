---
layout: post
title: Project 3 - VoiceWall. Say What You Wanna Say, As Long As You Test Them All First.
subtitle: Intro to Testing with RSpec, Capybara, and FactoryGirl
---
#[VoiceWall Repo](https://github.com/organicrails/voicewall)

After pretty much a year of learning Rails, I think it's really time for me to actually learn how to test. When I first started trying to write tests, I remember how daunting it was. For me, the entire testing suite appeared to be another language entirely. There were syntaxes that I had to memorize and new conventions that I had to follow. All the testing tutorials that I came across pretty much says "This is what you write if you want to test feature X, and this is what you write if you want to test feature Y." They were helpful -- if I want to test feature X or Y. But more than that I just want to have a general understanding of what to do  without the tutorials. 

Personally, I think it's extremely important for me to actually learn the syntax first. Sure, I may not be able to test extremely in depth at first, but at least I'll have a solid understanding of the foundation and know what I am actually doing. To achieve that, I've decided to create an extremely simple and bare-bone application, and write my tests around the application. This ensures that if my test does fail at some point, the problem will probably be with my tests and not with the application itself.

**VoiceWall is a simple CRUD application that allows anyone to post opinions on anything. We will be writing test around this application! We will be using RSpec, Capybara, and FatoryGirl to test our application.**

### Intro to RSpec

[RSpec](http://rspec.info/) is a Test-Driven Development (TDD) testing framework for Ruby. With TDD, users will write tests before any actual development is done. TDD normally goes through a short and repetitive cycle: 1. Write tests for a specific feature that may be added, run the test and watch it fail. 2. Write enough code to implement said feature, run the test and watch it pass. 3. If necessary, refractor the code. 4. Repeat with other features. [The TDD Wiki](https://en.wikipedia.org/wiki/Test-driven_development) is very well-documented if you would like to know more about TDD. With respect to Rails, Team Treehouse [wrote an excellent blog on RSpec](http://blog.teamtreehouse.com/an-introduction-to-rspec) that is definitely worth taking a look.

### Intro to Capybara

[Capybara](http://jnicklas.github.io/capybara/) is a acceptance test framework that simulates how users might interact when testing a web application. As oppose to manually fill out forms and click around every time you want to test a newly implemented feature, simply let Capybara test everything so you can focus on writing good code. With the combination of RSPec and Capybara, it is extremely easy to test and simulate how a user might think and interact with the application.

### Intro to FactoryGirl

[FactoryGirl](https://github.com/thoughtbot/factory_girl) allows you to create templates for reusable objects. Even though rails already have what are known as "fixtures", which allows users to predefined data for the tests, FactoryGirl objects are more much flexible and does not have to be loaded into the database. [This SO](http://stackoverflow.com/questions/5183975/factory-girl-whats-the-purpose) explains in detail why FactoryGirl is preferred over Rails fixtures. 


### To Start Off With...

We'll start off with by creating the application VoiceWall

`rails new voicewall`

and add the following gems to your **_GEMFILE_** under `group :development, :test`

{% highlight ruby %}

gem 'haml'
gem 'simple_form'

group :development, :test do

  ....

  gem 'rspec-rails'
  gem 'factory_girl_rails'
  gem 'capybara'
end
{% endhighlight%}

Run `rails g simple_form:install` to install simple_form, and run `rails generate rspec:install` to install RSpec. No additional configuration is needed for FactoryGirl or Capybara. 

In this application, we will be testing the applications in the following order - Model, Controller, and Requests. Before we proceed, you can delete the **_test_** directory from your application. We will not be using that directory, but rather the **_spec_** directory instead. 

The following tests are taken and modified from [everydayrails](http://everydayrails.com/2012/04/07/testing-series-rspec-controllers.html). 

### Testing the Model

Let's begin by creating a **_Voice_** model. Within a Voice, there will be a **title** describing the voice, and an **opinion** block describing the opinions. 

`rails g model Voice title:string opinion:text`
`rake db:migrate`

Notice the following

{% highlight ruby %}
invoke  active_record
create    db/migrate/20150819191625_create_voices.rb
create    app/models/voice.rb
invoke    rspec
create      spec/models/voice_spec.rb
invoke      factory_girl
create        spec/factories/voices.rb
{% endhighlight %}

Aside from the _active\_record_, the model generator also automatically created the **_voice\_spec.rb_** and FactoryGirl's **_voices.rb_**. Let's take a look at these two files

**_spec\_factories\_voices.rb_**

{% highlight ruby%}
FactoryGirl.define do
  factory :voice do
    title "MyString"
    opinion "MyText"
  end
end
{% endhighlight%}

When we created the model, FactoryGirl took note that within a Voice object, title is a string and opinion is a text. Without having to do anything, FactoryGirl automatically generated a factory **:vioce** which includes a title "MyString " and opinion "MyText". Throughout our application, we will make use of the symbol **:voice** to test the Voice objects. Now, whenever we want to create a Voice object somewhere in our spec, we can call something similar to the following ` FactoryGirl.create(:voice)`, and a Voice object would be initialized within the test. 

**spec\_models\_voice_spec.rb**

{% highlight ruby %}
require 'rails_helper'

RSpec.describe Voice, type: :model do
  pending "add some examples to (or delete) #{__FILE__}"
end
{% endhighlight %}

Very self-explanatory. The `RSpec.describe Voice do ... end` block infers that the following block of tests will be testing the Voice model. Let's write three simple tests for the Voice model to get a basic idea of how RSpec works. The first will test the voice factory within FactoryGirl, second will test that a title must be present, and third will test that opinion must be present. These tests are selected because the solutions to these tests are simple, so we can focus more on how RSpec works rather than actually figuring out how to code to make the tests pass.

**spec\_models\_voice_spec.rb**
{% highlight ruby %}
RSpec.describe Voice, type: :model do
  it "has a valid Voice factory" do
    expect(FactoryGirl.create(:voice)).to be_valid
  end

  it "is invalid without title" do
    expect(FactoryGirl.build(:voice, title: nil)).not_to be_valid
  end

  it "is invalid without opinion" do
    expect(FactoryGirl.build(:voice, opinion: nil)).not_to be_valid
  end
end
{% endhighlight%}

RSpec utilizes the `it...do` block to describe the tests. In this case, we have three tests with the following descriptions - "Voice has a valid Voice factory", "Voice is invalid without title", and "Voice is invalid without opinion". Let's go over each one and see how the RSpec syntax works.

{% highlight ruby %}
it "has a valid Voice factory" do
  expect(FactoryGirl.create(:voice)).to be_valid
end
{% endhighlight %}

RSpec utilizes a lot of human-understandable languages. Reading the test out loud, we see that this test is checking whether a valid Voice factory has been created. RSpec utilizes `expect(...).to` a lot. In some other tutorials, you might see `should` instead, which is now deprecated. Here, `expect(FactoryGirl.create(:voice)).to` is expecting the creation of **:voice** to be something. Reading on, this test is expecting the creation of **:voice** to be valid, aka there is indeed a Voice factory that has been created.

{% highlight ruby%}
it "is invalid without title" do
  expect(FactoryGirl.build(:voice, title: nil)).not\_to be\_valid
end
{% endhighlight %}

The test to check whether a title is present is also very similar to the one above. Here, we utilizes the `FactoryGirl.build` command, which will create a factory but not save it in the database. If you want a better understanding of create vs build when using FacotryGirl, [check out this SO](http://stackoverflow.com/questions/14098031/whats-the-difference-between-the-build-and-create-methods-in-factorygirl). This time, we use the `expect(...).not_to`, along with `be_valid` to check the invalidity of something. To check that all Voices without titles shall not pass, we need to create a Voice object with an invalid title. To achieve this, we can call upon the **:voice** factory that was automatically created earlier, but specify that the title must be nil. The result is `FactoryGirl.build(:voice, title: nil)`. After that, simply plug it into the rest of the test to form `expect(FactoryGirl.build(:voice, title: nil)).not_to be_valid`


{% highlight ruby%}
it "is invalid without opinion" do
  expect(FactoryGirl.build(:voice, opinion: nil)).not_to be_valid
end
{% endhighlight %}

Extremely similar to the test above, except that we are now testing that opinion must not be invalid. 

To run the tests above, simply call `rspec spec` in your terminal. You should see the following

{% highlight ruby %}
Failures:

  1) Voice is invalid without title
     Failure/Error: expect(FactoryGirl.build(:voice, title: nil)).not_to be_valid
       expected #<Voice id: nil, title: nil, opinion: "MyText", created_at: nil, updated_at: nil> not to be valid
     # ./spec/models/voice_spec.rb:9:in `block (2 levels) in <top (required)>'

  2) Voice is invalid without opinion
     Failure/Error: expect(FactoryGirl.build(:voice, opinion: nil)).not_to be_valid
       expected #<Voice id: nil, title: "MyString", opinion: nil, created_at: nil, updated_at: nil> not to be valid
     # ./spec/models/voice_spec.rb:13:in `block (2 levels) in <top (required)>'

Finished in 0.03569 seconds (files took 1.65 seconds to load)
3 examples, 2 failures

Failed examples:

rspec ./spec/models/voice_spec.rb:8 # Voice is invalid without title
rspec ./spec/models/voice_spec.rb:12 # Voice is invalid without opinion
{% endhighlight%}

We have 3 examples (3 tests we wrote), and 2 failed. This is expected since our first test (Voice has a valid Factory) already is a valid test, since FactoryGirl automatically created a Voice facotry for us. Let's fix the other two failures now. 

To fix the errors (which is extremely simple), simple add 

`validates_presence_of :title, :opinion` 

to your Voice Model.

Now run `rspec spec` again and you should see

{% highlight ruby %}
Finished in 0.03443 seconds (files took 1.61 seconds to load)
3 examples, 0 failures
{% endhighlight%}

Great, all passed! Testing with RSpec isn't that hard, right? 


### Testing the Controller

To test the controller, we first need to have one. 

` rails g controller Voices`

and (don't forget!!) add `resources :voices` to your **_routes.rb_**, as well set the root `root "voices#index"`

Like the creation of model above, the generator also automatically created the spec files. 

For the controller tests, we're going to test the most basic functions of all seven controller methods. This will once again ensures that we are focusing on how to write the controller specs and not the controller itself. In the following controller tests, some require the views of each action. Feel free to create your own views for this project, but if you do not want to, you can always copy my simple (but un-styled) views folder from the VoiceWall Repo above. 

Furthermore, the voices controller also simply dictates basic CRUD actions, nothing too fancy. If you want to write your own (and maybe alter some tests along the way for your own purpose), that is perfectly. Otherwise, the following is what I have for the controller. 

If you wish, you can wait to write each method after writing the test to get the full RED-GREEN-REFRACTOR cycle from TDD. But for my purpose (and not for best practice), I simply want to explain RSpec syntax and techniques, so I'll include the controller first. This will once again ensures that if I do get a red test, it will be because of the RSpec that I wrote and not because of the controller.

**_app\_controllers\_voices\_controllers.rb_**

{% highlight ruby %}
class VoicesController < ApplicationController

  before_action :find_voice, only: [:show, :edit, :update, :destroy]

  def index
    @voices = Voice.all
  end

  def new
    @voice = Voice.new
  end

  def create
    @voice = Voice.new(voices_params)
    if @voice.save
      redirect_to @voice
    else
      render 'new'
    end
  end

  def show
  end

  def edit
  end

  def update
    if @voice.update(voices_params)
      redirect_to @voice
    else
      render "edit"
    end
  end

  def destroy
    @voice.destroy
    redirect_to root_path
  end

  private

  def voices_params
    params.require(:voice).permit(:title, :opinion)
  end

  def find_voice
    @voice = Voice.find(params[:id])
  end
end

{% endhighlight %}

Let's get started!

PS: If, at any point, you see a "pending" when running `rspec spec`, (6 examples, 0 failures, 1 pending), simply delete the pending file as long as it's not used (should be the helper in most cases)!

#### INDEX Specs

What do we need to test in our index specs? How about let's see if it renders the correct view (index page), as well as whether or not there exists an array of Voices in our index method (`@voices = Voice.all`).

It's always good practice to list out all the potential tests that you might want to test in "it...do" blocks. For our index method, we currently have two "it...do" blocks that we want to implement, like the following. I'll be skipping showing the "it...do" blocks for the rest of the specs, but just remember this practice can only help you organizing your specs!

**_spec/controllers/voices\_controller\_spec.rb_**

{% highlight ruby %}
require 'rails_helper'

RSpec.describe VoicesController, type: :controller do
  describe "GET index" do
 
    it "creates an array of voices" do
    end

    it "renders the index template" do
    end
  end
end
{% endhighlight%} 

Let's proceed with our specs now. Before we write anything, let's quickly think about how we should approach this. "create an array of voices" sounds like it check whether a variable points to an array of Voices. "Renders the index template" sounds like it wants to make sure it lands on the index view page. With these directions in mind, here are the implementations.

**_spec/controllers/voices\_controller\_spec.rb_**
{% highlight ruby%}

require 'rails_helper'

RSpec.describe VoicesController, type: :controller do
  describe "#index" do
 
    it "creates an array of voices" do
      voice = FactoryGirl.create(:voice)
      get :index
      expect(assigns(:voices)).to eq([voice])
    end

    it "renders the index template" do
      get :index
      expect(response).to render_template :index
    end
  end
end
{% endhighlight%}

For "create an array of voices", first create a Voice factory and store it in the variable **voice**. Next, we need to get the current action, which is index. With `get :index`, we let the spec know that the current test is dealing with the index action. Last, we need to check for the array of Voices. In our controller index method, we stored the `Voice.all` within a `@voices` instance variable. How do we check that? Luckily, RSPec has the `assigns()` method. 

In short, `assigns()` simply [checks the value of an instance variable](http://stackoverflow.com/questions/8616508/what-does-assigns) within the controller. Since our instance variable is named `@voices` in our index controller, we simply have to call `assigns(:voices)` to perform validation tests on it. The rest is relatively self explanatory. `expect(assigns(:voices)).to eq([voice])` expects `@voices` to be an array of voice. 

For "render the index template", the test is even simpler. Like the previous test, we let the test to know it is the index action that we're testing by calling `get :index`. Next, we except the page to render the index page. This can all be done in one line with `expect(response).to render_template :index`. 

Run `rspec spec` in your terminal, and it should return `5 examples, 0 failures`. Great!


#### NEW and CREATE Specs

Next we'll target the new and create tests. Some of the specs will be familiar, but there are a few new cool things too. For the **new** action, we'll simply be making sure it renders the new page. For the **create** action, we'll check a bit more. First, we'll be checking for both _valid_ inputs and _invalid_ inputs. To do this, we'll need to modify the FactoryGirl settings a bit, which I'll show later. Then, for both valid and invalid inputs, we'll see if such inputs modified the database or not. Finally, we'll make sure both inputs render their respective pages after the actions. 

**_spec/controllers/voices\_controller\_spec.rb_**
{% highlight ruby %}
require 'rails_helper'

RSpec.describe VoicesController, type: :controller do


  describe "#index" do
    ...
  end

  describe "#new" do
    it "renders the new template" do
      get :new
      expect(response).to render_template :new
    end
  end

  describe '#create' do

    context "with valid inputs" do
      it "creates and increase Voice count by 1" do
        expect{
          post :create, voice: FactoryGirl.attributes_for(:voice)
        }.to change(Voice, :count).by(1)
      end

      it "redirects to show page" do
        post :create, voice: FactoryGirl.attributes_for(:voice)
        expect(response).to redirect_to Voice.last
      end
    end

    context "with invalid inputs" do
      it "does not increase Voice count" do
        expect{
          post :create, voice: FactoryGirl.attributes_for(:invalid_voice)
        }.not_to change(Voice, :count)
      end

      it 'renders #new again' do
        post :create, voice: FactoryGirl.attributes_for(:invalid_voice)
        expect(response).to render_template :new
      end
    end
  end
end

{% endhighlight%}

The tests regarding the **new** action should be very familiar. We're simply checking the GET response for new, and make sure that it renders the correct template (new page).

However, in the **create** tests, we also start to see something new. The most apparent one is the use of "context". I'd like to think of context as the little brother of "describe". While "describe" attempts to tell the users what the test is about on a "big picture" level, the "context" keyword tells the users more detailed, and maybe even categorized, descriptions. 

In our **create** action's tests, our "big picture" is described as "#create". Within "#create", we have two contexts. One is the creation of a Voice object with VALID input, and the other is with an INVALID input. Within these two contexts exist their own separate tests specific to that describe + context. 

Let's begin to understand the tests. First, we have the `it "creates and increases Voice count by 1"` test. This test is essentially checking whether or not the Voice object, if given valid inputs, is actually saved within the database. We know that by successfully saving a Voice object, the count of Voices will be raised by 1. This test utilizes such information. 

Immediately, we see a `expect{...}` block. Within the block, we first call `post :create` to tell the test that this is retrieving information on the POST action **create**. We then need to create a voice to simulate the create action. In this case, we can use the _attributes\_for_ method from RSpec to create and retrieve a hash of the attributes within `:voice`. [This SO](http://stackoverflow.com/questions/13150272/meaning-for-attributes-for-in-factorygirl-and-rspec-testing) was essential in my understanding of what `attributes_for` was really doing. Remember, all the code so far was within the `expect{...}` block, because what are we expecting? We're expecting that upon saving, the count of the Voice objects will increases by 1. This is simulated by calling `.to change(Voice, :count).by(1)` after the expect block. 

The "redirects to show page" test should a familiar sight by now. We're simply doing the same thing we did before, expecting the response to _redirect\_to_ (note it's not _render\_template_) **Voice.last**. Why is it **Voice.last**? Because after we created and saved, the last Voice object within the database would be the one we just created. 

We now approach our second context, "with invalid inputs". As the description suggests, every test within this context is to test Voices with invalid inputs. 

With invalid inputs, we want to make sure that invalid Voices does not get saved into the database. Another way to put it is to make sure the total count of Voices does not change with an invalid Voice object. We once again start off with an `expect{...}` block, and within the block we enter the familiar 

`post :create, voice: FactoryGirl.attributes_for(:invalid_voice)`

Note, however, that this time the object is **:invalid_voice** as oppose to **:voice**. How does `:invalid_voice` look like? Let's take a look

**_spec\_factories\_voices.rb_**
{% highlight ruby%}
FactoryGirl.define do
  factory :voice do
    title "MyString"
    opinion "MyText"
  end

  factory :invalid_voice, parent: :voice do
    title nil
  end
end
{% endhighlight%}

Here, we create another factory **:invalid_voice**, and reference it to the parent, which is **:voice**. We have to reference the parent whenever we attempt to create another type of Voice factory because FactoryGirl references the Voice model with **:voice**. Since there does not exist a invalid\_voice Model, we cannot simply create a **:invalid\_voice** factory. 

Another method to create an invalid\_voice Factory is to use a _trait_, which essentially does the same thing as referencing the parent factory. 

**_spec\_factories\_voices.rb_**
{% highlight ruby%}
FactoryGirl.define do
  factory :voice do
    title "MyString"
    opinion "MyText"

    trait :invalid_voice do
      title nil
    end
  end
end
{% endhighlight%}

With this method, you can call the **:invalid_voice** trait like so in your specs for the same example. 

`post :create, voice: FactoryGirl.attributes_for(:voice, :invalid_voice)`

Now that we understand how to create different types of factories for the same model, we simply employ the same technique as the previous tests, only this time we expect the count of Voice to NOT change. 

The "renders #new again" is once again self-explanatory. If the a Voice object has invalid inputs, expect the response to render the "new" view again. 

Run `rspec spec`, and you should get `10 examples, 0 failures`. Great, let's move on!

#### SHOW specs

The show method specs is pretty much the same as all other view specs, so I won't be going over it in detail. 

**_spec/controllers/voices\_controller\_spec.rb_**
{% highlight ruby %}
require 'rails_helper'

RSpec.describe VoicesController, type: :controller do


  describe "#index" do
    ...
  end

  describe "#new" do
    ...
  end

  describe '#create' do
    ...
  end

  describe "#show" do
    it "render the show template" do
      get :show, id: FactoryGirl.create(:voice)
      expect(response).to render_template :show
    end
  end
end

{% endhighlight%}

running `rspec spec` should return `11 examples, 0 failures` now. 

#### EDIT and UPDATE specs

With edit and update specs, we once again see something that we already know, as well as some new found knowledge. The edit specs is simply checking the correct view is rendered. The update specs, however, is a bit more complicated. With update, we need to first make sure that there exists a old voice already, and then we can successfully test if modifying the old voice into a new voice is valid or not. As with the **create** action, the **update** action will also have two contexts, one with valid inputs and one with invalid inputs. Let's take a look at how to achieve that.  

**_spec/controllers/voices\_controller\_spec.rb_**
{% highlight ruby %}
require 'rails_helper'

RSpec.describe VoicesController, type: :controller do


  describe "#index" do
    ...
  end

  describe "#new" do
    ...
  end

  describe '#create' do
    ...
  end

  describe "#show" do
  	...
  end

  describe "#edit" do
    it "render the edit template" do
      get :edit, id: FactoryGirl.create(:voice)
      expect(response).to render_template :edit
    end
  end

  describe "#update" do

    before :each do
      @initial_voice = FactoryGirl.create(:voice, title: "Chipotle", opinion: "It is awesome.")
    end

    context "with valid inputs" do

      it "located the requested @voice" do
        put :update, id: @initial_voice, voice: FactoryGirl.attributes_for(:voice)
        expect(assigns(:voice)).to eq(@initial_voice)
      end

      it "changes the @voice attributes" do
        put :update, id: @initial_voice, voice: FactoryGirl.attributes_for(:voice, title: "McDonalds", opinion: "It is only okay...")
        @initial_voice.reload
        expect(@initial_voice.title).to eq("McDonalds")
        expect(@initial_voice.opinion).to eq("It is only okay...")
      end

      it "redirects to the updated contact" do
        put :update, id: @initial_voice, voice: FactoryGirl.attributes_for(:voice)
        expect(response).to redirect_to @initial_voice
      end
    end

    context "with invalid inputs" do

      it "located the requested @voice" do
        put :update, id: @initial_voice, voice: FactoryGirl.attributes_for(:invalid_voice)
        expect(assigns(:voice)).to eq(@initial_voice)
      end

      it "changes the @voice attributes" do
        put :update, id: @initial_voice, voice: FactoryGirl.attributes_for(:voice, title: nil, opinion: "It is only okay...")
        @initial_voice.reload
        expect(@initial_voice.title).not_to eq("McDonalds")
        expect(@initial_voice.opinion).not_to eq("It is only okay...")
      end

      it "renders the edit template again" do
        put :update, id: @initial_voice, voice: FactoryGirl.attributes_for(:invalid_voice)
        expect(response).to render_template :edit
      end
    end  
  end
end

{% endhighlight%}

That's quite a bit of code, let's take it slow and break it down one by one.

I'll skip over the **edit** action test, since it's the same good old template test that we know and love. 

In the **update** action test, we see something new here. 

{% highlight ruby %}
before :each do
  @initial_voice = FactoryGirl.create(:voice, title: "Chipotle", opinion: "It is awesome.")
end
{% endhighlight%}

The above block states that before before anything, create an **@initial\_voice** variable that contains the first (initial) voice object. This object contains the title "Chipotle" and the opinion "It is awesome.". This steps makes sense in terms of what we're doing because for UPDATE, we require two voices, one as the starting voice and one as the updated voice. **@initial\_voice** is our starting voice, and is created before any tests are executed. 

We then jump into our first context within **update** tests: "with valid inputs".

The point "locate the request @voice" is to ensure that we are selecting the correct Voice object. To understand what the test is doing [check out this SO that I asked](http://stackoverflow.com/questions/31998692/rspec-testing-instance-variable-to-id-with-eq-explanation), because the answer explains it better than I ever could.

"changes the @voice attributes" is essentially checking whether the update is successful or not. Start out by calling `put: :update` (If you don't quite understand /  want to know more about all the RESTful methods, [check this out](http://www.restapitutorial.com/lessons/httpmethods.html)), and set the `id:` to **@initial_voice** ID. Then, we create a voice object, but notice how we overwrite the title and the opinion to "McDonalds" and "It is only okay...". This is essentially doing the "edit".  

`put :update, id: @initial_voice, voice: FactoryGirl.attributes_for(:voice, title: "McDonalds", opinion: "It is only okay...")`

After the above call, **@initial_voice** would still have the SAME ID, but DIFFERENT title and opinion. To test that this change did indeed go through, we first call `@initial_voice.reload` to reload it on a database level, then we do write two tests to make sure that the title is "Mcdonalds" and the opinion is "It is only okay..." as oppose to the original "Chipotle" and "It is awesome".

"redirects to the update contact" is once again another test that we are familiar with. The only new thing that we see here is the addition of `id: @initial_voice`. Remember, UPDATE is a PUT method, so it requires a specific Voice object for each individual update. One way to grant each update individuality is to find the id first. 

Moving on to the next context, "with invalid inputs" 

The first test "located the requested @voice" is the same as the previous context. Feel free to review [this SO](http://stackoverflow.com/questions/31998692/rspec-testing-instance-variable-to-id-with-eq-explanation) to understand what's going on.

The second test, "changes the @voice attributes" under the context "with invalid inputs", tests to make sure invalid inputs does not update the Voice. We once again start out with something that we're all too familiar

`put :update, id: @initial_voice, voice: FactoryGirl.attributes_for(:voice, title: nil, opinion: "It is only okay...")`

except notice how here, **title** is nil. By setting the **title** as nil, we successfully render this Voice object as an invalid Voice object. We then reload **@initial_voice** again, but this time we want to test that he title and opinion are NOT "Mcdonald" and "It is only okay". 

The "renders the edit template again" test simply tests that with invalid inputs, the response will render the edit view page again. 

Running `rspec spec` at this point should yield `18 examples, 0 failures`. Awesome, only one more method to go.

#### DESTROY Specs

I'll let you guys figure out how the Destroy specs work. Both tests are very similar to what we've been writing so far, so it shouldn't be too difficult to understand!

**_spec/controllers/voices\_controller\_spec.rb_**
{% highlight ruby %}
require 'rails_helper'

RSpec.describe VoicesController, type: :controller do

  ....

  describe "#destroy" do
    before :each do
      @voice = FactoryGirl.create(:voice)
    end

    it "deletes the Voice" do
      expect{
        delete :destroy, id: @voice
      }.to change(Voice, :count).by(-1)
    end

    it "redirects to root page" do
      delete :destroy, id: @voice
      expect(response).to redirect_to root_path
    end
  end
end
{% endhighlight%}

Running `rspec spec` should yield `20 examples, 0 failures` at this point.

And this concludes our controller tests!!

Let's move on to our final group of tests, the Request Specs

### Playing With Request Specs

Requests specs are generally tests on how the user interacts with the application. In a given a application, the users have a lot of options the moment they land onto the home page. To make sure that no unforeseen actions taken by the users will return errors, it's important to write a few request specs that will simulate the common paths users might take. 

In our tests, we'll be using Capybara to help us writing these request specs. In order for Capybara to work, you need to include Capybara within your **_spec\_helper.rb_**

**_spec/spec\_helper.rb_**

{% highlight ruby %}
require 'capybara/rspec'

RSpec.configure do |config|
  ...
end
{% endhighlight%}

In order to run the request specs, Capybara will deliberately look for a **_spec/features_** directory([from this SO](http://stackoverflow.com/questions/9059854/capybara-undefined-method-visit)). If you havn't done so already, create the **_features_** folder!. Within that folder, create a new file, let's call it **_voice\_request\_spec.rb_**.

Great, let's get started with our request specs! In our specs, we'll test three things: create a new voice, edit a voice, and delete a voice.

The skeleton of our request specs will be the following

**_spec/features/voice\_request\_spec.rb_**

{% highlight ruby %}
require 'rails_helper'

describe "Voices" do
  context "Manage Voices" do
    # our specs
  end
end
{% endhighlight %}

#### Request Specs for Creating New Voices

What does a user have to do in order to create a new Voice? The general steps will probably be the following: Go to the home page, click on New Voice link, fill in the form, and be redirected to the show page with all the information. Capybara can simulate the actions like so

**_spec/features/voice\_request\_spec.rb_**

{% highlight ruby %}
require 'rails_helper'

describe "Voices" do
  context "Manage Voices" do
    it "Add new voice and displays the results" do
      visit root_path
      expect{
        click_link "New Voice"
        fill_in "Title", with: "A Random Fact"
        fill_in "Opinion", with: "Capybara is the largest rodent in the world."
        click_button "Create Voice"
      }.to change(Voice,:count).by(1)

      within 'h2' do
        expect(page).to have_content("A Random Fact")
      end

      expect(page).to have_content("Capybara is the largest rodent in the world.")
    end
  end
end
{% endhighlight %}

The test "Add new voice and displays the results" will do the following. It will `visit` the root path so that we'll be on the right page. We then do two things simultaneously. First is that we will traverse through the form using some extremely helpful functions, `click_link`, `fill_in`, and `click_button`. These functions will simulate the users filling in the form with title and opinion. Second is that we include all these actions within an `expect{...}.to` block and check the count like we did in previous tests. Do we need to? No, since we have the correct value for both title and opinion. But if the title or the opinion does contain invalid values, the `expect{...}` block would have caught it for us. 

Now, after creating the voice in this test, we expect to see the show page to display both the title and the opinion. This is simulate with the `have_content` function. How does the spec knows that we're on the show page? Because when the spec ran `click_button "Create Voice"`, it automatically follows the path to wherever the **create** function redirects to, and in this case it's the show page. 

We can even get fancier and more in-dept with the testing by testing the HTML tags on the page. Since we declared that all title should have a **h2** tag, we can use the `within 'h2' do...end` block to test the title. The `within` block also works on other HTML and CSS tags, so the testing scope is limitless!

#### Request Specs for Editing Voices

The tests for "Edit Voice and Display the Result" is also very similar. We once again think about how we're going to simulate the users' actions when someone attempts an edit. 

**_spec/features/voice\_request\_spec.rb_**

{% highlight ruby %}
require 'rails_helper'

describe "Voices" do
  context "Manage Voices" do

    it "Add new voice and displays the results" do
      ...
    end

    it "Edit Voice and Display the Result" do
      initial_voice = FactoryGirl.create(:voice, title: "Another Random Fact", opinion: "A flock of crows is known as a murder.")
      visit voice_path(initial_voice)


      expect(page).to have_content("Another Random Fact")
      expect(page).to have_content("A flock of crows is known as a murder.")

      click_link "Edit"
      fill_in "Title", with: "Yet Another Random Fact!"
      fill_in "Opinion", with: "You cannot snore and dream at the same time."
      click_button "Update Voice"

      within 'h2' do
        expect(page).to have_content("Yet Another Random Fact!")
      end

      expect(page).to have_content("You cannot snore and dream at the same time.")
    end
  end
end
{% endhighlight %}

For the edit request specs, we fist create an **initial\_voice**, because in order to edit a voice, there needs to already exist another voice. For this **initial\_voice**, let's change the title and opinion into something more amusing. After creating the **initial_voice**, we need to head to the show page, so we call upon `voice_path(initial_voice)`. 

At this point, we expect to have the title and opinion to be displayed in the show page, so we can write some simple tests for that.

Utilizing the `click_link`, `fill_in`, and `click_button` methods again (aren't they just so useful), we edit the **initial_voice** with a new title and a new opinion. At this point, we expect the show page to contain the newly edited title and opinion. 

And we're done with the edit request specs! Wasn't that also pretty simple?

#### Request Specs for Deleting Voices

After reading the two request specs above, I am very sure you can now understand the following delete request specs. If not, feel free to ask about it in the comments!

**_spec/features/voice\_request\_spec.rb_**

{% highlight ruby %}
require 'rails_helper'

describe "Voices" do
  context "Manage Voices" do

    it "Add new voice and displays the results" do
      ...
    end

    it "Edit Voice and Display the Result" do
      ...
    end

    it "Delete Voice " do
      initial_voice = FactoryGirl.create(:voice, title: "Another Random Fact", opinion: "A flock of crows is known as a murder.")
      visit voice_path(initial_voice)

      expect{
	      click_link "Destroy"
      }.to change(Voice, :count).by(-1)

      expect(page).not_to have_content("Another Random Fact")
      expect(page).not_to have_content("A flock of crows is known as a murder.")
    end
  end
end
{% endhighlight %}


At this point, running `rspec spec` should yield `23 examples, 0 failures`. If you only want to run the request specs, you can run the following `rspec spec/features` to just run all the tests within the features folder! This will yield `3 examples, 0 failures` (which is correct, since we only have three request specs). Running specs on specific folders, or even files, can be really useful when you have a lot of tests to run on bigger projects. This will prevent you from keep on re-running tests that you know will pass for sure. 


And we're all done!!

Thank you again for joining me today, and I'll see you next time


### Some Self Reflection
Nothing much to say this time, I think the entire process went by pretty smoothly. Even though the majority of the tests were taken from another site, I still learned a lot (especially regarding TDD foundations). For future testing walkthroughs, I think I'll focus more on how to create the tests by myself, as I can't follow other testing templates forever. After understanding the syntaxes, understanding what to test on future apps is the logical next step, and shouldn't be too difficult. 




















