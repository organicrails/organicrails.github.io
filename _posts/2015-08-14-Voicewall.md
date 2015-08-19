---
layout: post
title: Project 3 - VoiceWall. Say What You Wanna Say, As Long As You Test Them All First.
subtitle: Intro to Testing with RSpec, Capybara, and FactoryGirl
---

After pretty much a year of learning Rails, I think it's really time for me to actually learn how to test. When I first started trying to write tests, I remember how daunting it was. For me, the entire testing suite appeared to be another language entirely. There were syntaxes that I had to memorize and new conventions that I had to follow. All the testing tutorials that I came across pretty much says "This is what you write if you want to test feature X, and this is what you write if you want to test feature Y." They were helpful -- if I want to test feature X or Y. But more than that I just want to have a general understanding of what to do  without the tutorials. 

Personally, I think it's extremely important for me to actually learn the syntax first. Sure, I may not be able to test extremely indepth at first, but at least I'll have a solid understanding of the foundation and know what I am actually doing. To achieve that, I've decided to create an extremely simple and barebone application, and write my tests around the application. This ensures that if my test does fail at some point, the problem will probably be with my tests and not with the application itself.

**VoiceWall is a simple CRUD application that allows anyone to post opinions on anything. We will be writing test around this application! We will be using RSpec, Capybara, and FatoryGirl to test our application.**

### Intro to RSpec

[RSpec](http://rspec.info/) is a Test-Driven Development (TDD) testing framework for Ruby. With TDD, users will write tests before any actual development is done. TDD normally goes through a short and repetitive cycle: 1. Write tests for a specific feature that may be added, run the test and watch it fail. 2. Write enough code to implement said feature, run the test and watch it pass. 3. If necesserary, refractor the code. 4. Repeeat with other features. [The TDD Wiki](https://en.wikipedia.org/wiki/Test-driven_development) is very well-documented if you would like to know more about TDD. With respect to Rails, Team Treehouse [wrote an excellent blog on RSpec](http://blog.teamtreehouse.com/an-introduction-to-rspec) that is definitely worth taking a look.

### Intro to Capybara

[Capybara](http://jnicklas.github.io/capybara/) is a acceptance test framework that simulates how users might interact when testing a web application. As oppose to manually fill out forms and click around everytime you want to test a newly implemented feature, simply let Capybara test everything so you can focus on writing good code. With the combination of RSPec and Capybara, it is extremely easy to test and simulate how a user might think and interact with the application.

### Intro to FactoryGirl

[FactoryGirl](https://github.com/thoughtbot/factory_girl) allows you to create templates for reusable objects. Even though rails already have what are known as "fixtures", which allows users to predefined data for the tests, FactoryGirl objects are more much flexible and does not have to be loaded into the database. [This SO](http://stackoverflow.com/questions/5183975/factory-girl-whats-the-purpose) explains in detail why FactoryGirl is preferred over Rails fixtures. 


### To Start Off With...

We'll start off with by creating the application VoiceWall

`rails new voicewall`

and add the following gems to your **_GEMFILE_** under `group :development, :test`

{% highlight ruby %}
group :development, :test do

  ....

  gem 'rspec-rails'
  gem 'factory_girl_rails'
  gem 'capybara'
end
{% endhighlight%}


