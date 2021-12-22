## RSpec tutorial

- Learn the basics of RSpec, a testing framework often used in Rails.
- You can read and work on the code by yourself.
  - You can write the code in this document and try to run it.
- Sample code repository: https://github.com/igaiga/rails6132_ruby301_rspec
  - I think you can find most of the code used in the explanation here.
  - The version of Rails and Ruby is a bit old.

## Prerequisite Knowledge

- You should be able to understand CRUD-like operations in Rails.
- Specifically, we'll be writing test code for a CRUD application created with scaffold.
- If you want to learn about CRUD in Rails, I recommend you to read my book "Rails Textbook".

## Rough flow

- Build the environment
  - Create a Rails app to write RSpec code.
- Model spec
- Basic RSpec syntax
- How to use FactoryBot
- System spec
- How to use mocks and stubs
- Request spec

## Environment

- If you have a newer version than the following, you can use the newer version

- Rails 6.1 series latest
    - Rails 6.1.4.2
- Ruby3.0 series latest
    - Ruby 3.0.3
- rspec-rails Latest
    - rspec-rails 5.0.2
    - rspec-core 3.10.1

## Difference between RSpec and minitest

- This document is about RSpec.
- I'll explain the difference between RSpec and minitest, which is the default for Rails.
- If you want to know how to write test code with minitest, please refer to the book "Perfect Ruby on Rails" (Japanese).

- RSpec
  - The de facto standard with a large market share.
  - Many engineers have written it before
  - Lots of tools, but you can also write overly-structured tests
  - RSpec has a wrapper around the minitest, so the codes inside are a bit hard to follow

- minitest
  - Default in Rails
  - New features are introduced quickly, such as multiple DBs
  - Simple, with a policy of adding tools as add-ons
  - Not much market share, many engineers have never written it before

## References

- These are the main reference documents used in this course
- RSpec reference page https://relishapp.com/rspec/rspec-rails/docs
- Rails Guide "Testing Rails Applications": https://guides.rubyonrails.org/testing.html
- Capybara reference page: https://rubydoc.info/github/teamcapybara/capybara/master/Capybara/Node/Matchers

## Building the RSpec environment

- Create a Rails app to write RSpec code
  - Or you can use your rails app.
    - In this case, you can skip "rails new" command.

- $ rails new app_name --skip-test
  - The `--skip-test` option is an option to not create the minitest related files that are standard in Rails.
  - If you want to add RSpec to an app that has already been rails new with minitest, just delete the test folder after adding RSpec and you're good to go.

- Edit the Gemfile and add the rspec-rails gem
  - https://github.com/rspec/rspec-rails

```Gemfile
group :development, :test do
  ...
  gem 'rspec-rails'
  ...
end
```

- $ bundle install
- $ bin/rails generate rspec:install
  - Create a configuration file.
- To check that it works, you can run the following
  - $ bundle exec rspec spec/
  - If you see something like `0 examples, 0 failures`, you're good to go.

- Reference: https://github.com/igaiga/rails6132_ruby301_rspec

## Use spring in bin/rspec command

- You don't have to do this, but if you do, it will create a bin/rspec command and use spring to run it, making it faster.
    - Spring will be removed from the default in Rails 7.0.

Gemfile

```Gemfile
group :development do
  ...
  gem 'spring-commands-rspec'
  ...
end
```

- Edit the Gemfile and add rspec-rails gem
    - It seems that only development is needed for the additional environment.
    - It seems that you need to leave it in not only during the installation, but also during the whole time you are using it.

- $ bundle install
- $ bundle exec spring binstub rspec

- To check that it works, just run the following
  - $ bin/rspec spec/
  - If you see something like `0 examples, 0 failures`, you're good to go.

- spring caches the information from the last time it was run.
- If the behavior doesn't reflect the code change, stop spring and re-run it with the following command
  - $ bin/spring stop
  - There is no command to start spring, it will start by itself when you run it.

## Preparing to use factory_bot

- factory_bot is a tool to create model data for testing.
    - I'll explain the details later.
- Edit the Gemfile and add factory_bot_rails gem
    - https://github.com/thoughtbot/factory_bot_rails

```Gemfile
group :development, :test do
  ...
  gem 'factory_bot_rails'
  ...
end
```

- $ bundle install

## Create the code to be tested

- This time we will use scaffold to create the code to be tested.
    - This is an application to CRUD books.
    - We'll call the code to be tested production code as opposed to test code.
- $ bin/rails g scaffold books title author
    - It also creates a template for rspec and factorybot.
- $ bin/rails db:migrate
- If you want to get an idea of what it looks like, try to play around with it by running rails s

## Commonly Written Tests in Rails Apps

- There are three types of tests that are commonly used in Rails apps
- Model spec: Model tests
- System spec: E2E (= application wide) tests using the browser
- Request spec: E2E tests that don't use the browser
    - In minitest, it corresponds to Controller test and Integration test.
- System spec is recommended when you want to run JS, and Request spec is recommended when you want to run API.
- There are also tests for other Rails components such as ActiveJob and ActionMailer.
    - Details: https://guides.rubyonrails.org/testing.html
- Controller spec and View spec are rarely used because they can be covered by E2E tests.
    - It's a bit complicated, but Controller spec (RSpec) and Controller test (minitest) are not the same thing.

## Write a Model spec - Try to run RSpec

- Now let's write the test.
- Open the spec/models/book_spec.rb (the spec for the Book model) generated by scaffold in an editor

spec/models/book_spec.rb

```spec/models/book_spec.rb
require 'rails_helper' # All tests have code that reads the configuration file rails_helper.rb

RSpec.describe Book, type: :model do # We will write the test code for the Book model in the block.
  # Write the test code for the Book model here.
  pending "add some examples to (or delete) #{__FILE__}"
end
```

- pending is a method that pauses the test execution and displays a comment.
- First, let's run the test as is
- $ bin/rspec spec/models/book_spec.rb
    - `1 example, 0 failures, 1 pending`.
- Let's edit this file and write the test

## List of ways to run RSpec

- $ bin/rspec spec/
  - Run all tests
- $ bin/rspec spec/models/
    - Execute all tests under the specified folder
- $ bin/rspec spec/models/book_spec.rb
    - Execute specified file
- $ bin/rspec spec/models/book_spec.rb:5
    - File specification + line number specification execution
- As a prerequisite, you'll need to prepare a DB with RAILS_ENV=test beforehand.
    - If you have a development environment DB, you can copy the schema and generate it automatically in the test environment.
    - DB creation command $ RAILS_ENV=test bin/rails db:create db:migrate

## Write Model spec - Write tests that fail

- Next, let's write a test that will fail.
    - This is also a good way to check if the environment works correctly.
- We'll write a test that will be true when it expects false.

spec/models/book_spec.rb

```spec/models/book_spec.rb
require 'rails_helper'
RSpec.describe Book, type: :model do
  it "To be false if true" do # Write the "description" that will be displayed on NG after it
    expect(true).to eq(false)
    # expect(code under test).to matcher(expected test result)
    # A matcher is a tool to determine if there is a match.
    # Here we use eq to determine match with ==.
  end
end
```

- $ bin/rspec spec/models/book_spec.rb
    - `1 example, 1 failure`
    - If you see a failure of 1, you are on the right track.
- The following explains how to read the error message.

## How to read the message on NG

- The most useful information is the expected/got, the line number that failed.


```console
$ bin/rspec spec/models/book_spec.rb
(Abbreviation)
Failures:
  1) 1) Book true when it should be false # Test name written after it
     Failure/Error: expect(true).to eq(false) # Test code that resulted in NG

       expected: false # result expected by the test code
            got: true # result of execution in this expect.

     # . /spec/models/book_spec.rb:5:in `block (2 levels) in <main>' # Failed line number
(Abbreviated)
Finished in 0.0556 seconds (files took 0.1898 seconds to load)
1 example, 1 failure # Number of failed tests in all tests

Failed examples: # List of tests that failed
rspec . /spec/models/book_spec.rb:4
```

## Write a Model spec - Write a test that will be OK

- Write a test that will be "true when it expects true".
    - Review: Write expect(code under test).to matcher(expected test result) in it block

spec/models/book_spec.rb

```spec/models/book_spec.rb
require 'rails_helper'
RSpec.describe Book, type: :model do
  it "When true, must be true." do
    expect(true).to eq(true)
  end
end
```

- If `1 example, 0 failures` is OK, the result is easy to see.
- If you want to write it in one line, you can write it like this
    - `it("to be true if true"){ expect(true).to eq(true) }`.
    - In Ruby, it is customary to write a block with `{}` for a single line, and `do end` for multiple lines

## Writing Model spec - Using the Book model

- Let's use the Book model as a test object.
- We'll write a test to make sure that Book.new is executed.
- In this document, we'll omit the other parts and just write it.

spec/models/book_spec.rb

```spec/models/book_spec.rb
it "Book model should not be nil when new is created." do
  expect(Book.new).not_to eq(nil)
end
```

- Writing tests for negation, using expect().not_to
- `eq(nil)` also has a matcher called be_nil
- `expect(Book.new).not_to be_nil`.
- List of eq and other standard matchers: https://rubydoc.info/gems/rspec-expectations/frames#built-in-matchers

## Write a Model spec - call a method of the Book model

- Implement the test target as the title_with_author method of the Book model and write the test code for it

app/models/book.rb

```app/models/book.rb
class Book < ApplicationRecord
  def title_with_author
    "#{title} - #{author}"
  end
end
```

spec/models/book_spec.rb

```spec/models/book_spec.rb
RSpec.describe Book, type: :model do
  describe "Book#title_with_author" do # It's easier to read if you use the describe method and separate each method
    it "When you call Book#title_with_author, it should return a string with title and author" do
      book = Book.new(title: "RubyBook", author: "matz")
      expect(book.title_with_author).to eq("RubyBook - matz")
    end
  end
end
```

## How to debug RSpec

- If you want to debug, try stopping it by binding.irb in the it block

spec/models/book_spec.rb

```spec/models/book_spec.rb
require 'rails_helper'
RSpec.describe Book, type: :model do
  describe "Book#title_with_author" do
    it "When you call Book#title_with_author, it should return a string connecting title and author" do
      book = Book.new(title: "RubyBook", author: "matz")
      binding.irb # execution stops here and we can use irb
      expect(book.title_with_author).to eq("RubyBook - matz")
    end
  end
end
```


## describe: a tool for separating

- The structure of the previous test code is now separated into methods using describe.
    - Instance methods are denoted by #   `describe "Book#title_with_author" do`
    - Class methods are represented by .   `describe "Book.new" do`
- It is recommended to create a describe block for each method and separate them.
    - This is a common delimitation technique in my observation, but it is not a rule but a convention.
- The describe block is a syntax for separating and creating structure.
    - It works the same without it.
    - It is possible to write multiple describe blocks and nest them, so please write them when you want to delimit.

spec/models/book_spec.rb

```spec/models/book_spec.rb
RSpec.describe Book, type: :model do
  describe "Book#title_with_author" do # It is easier to read if you use the describe method and separate each method
    it "Calling Book#title_with_author should return a string." do
      expect(book.title_with_author).to eq("RubyBook - matz")
...
```

## context: a tool for delimiting by context

- The context block is also a grammar for creating structure.
    - It works the same as describe, but the meaning interpreted by the programmer is different.
- `context "Book#title is a string" do end` is a tool for writing "when".
- context is often written inside describe.
- describe and context can be nested together

spec/models/book_spec.rb

```spec/models/book_spec.rb
describe "Book#title_with_author" do
  context "When Book#title is a string" do # describe the situation
    it "that the string connecting title and author is returned" do
      book = Book.new(title: "RubyBook", author: "matz")
      expect(book.title_with_author).to eq("RubyBook - matz")
    end
  end
  context "When Book#title is nil" do # explain the situation
    it "Returning a string with empty title and author" do
      book = Book.new(author: "matz")
      expect(book.title_with_author).to eq(" - matz")
    end
  end
end
```

## Basic Form of RSpec Test Code Structure

- The basic RSpec test code structure can be summarized as follows
    - The same applies to the rest of the Model spec
    - describe, context may be nested

```ruby
require 'rails_helper'
RSpec.describe Book, type: :model do
  describe "#.method_name" do
    context "When it happens" do
      it "A should be xxx" do end
      it "B should be xxx" do end
    end
    context "When it happens" do
      it "A should be xxx" do end
    end
  end
  describe "#.method_name" do ... end
end
```

- Not only the Model spec, but other tests as well

## before: A tool to prepare for testing

- By using the before method, you can write the block to be executed before executing it.
    - If you want to use a common variable between before and it, use instance variables.

```ruby
context "When Book#title is a string" do
  before do
    @book = Book.new(title: "RubyBook", author: "matz")
  end
  it "that the string connecting title and author is returned" do
    expect(@book.title_with_author).to eq("RubyBook - matz")
  end
end
```

- Prepare the test object in before and write the expected result in it
- Whether to use before or not depends on the project and the case.
    - I tend to use it.
- There is also after, although I don't use it often.
- Details: https://relishapp.com/rspec/rspec-core/docs/hooks/before-and-after-hooks

## subject: a tool for writing test targets

- There is also a tool to specify the subject of a test called the subject method.
- The following code specifies that `Book.new(title: "RubyBook", author: "matz")` is the test subject.
- Be careful to write outside of it!
- Write the test object in subject and the expected result in it

```ruby
context "When Book#title is a string" do
  subject { Book.new(title: "RubyBook", author: "matz") }
  it "that the string connecting title and author is returned" do
    expect(subject.title_with_author).to eq("RubyBook - matz")
  end
end
```

- I think it depends on the project whether you use it or not.
    - I don't use them.

## You can also give a name to a subject

- You can also give a name to a subject

```ruby
context "When Book#title is nil" do
  subject(:book){ Book.new(author: "matz") } # name the subject as book
  it "that the string connecting the empty title and author is returned" do
    expect(book.title_with_author).to eq(" - matz") # book points to Book.new(author: "matz")
  end
end
``

## let, let!: Tools for writing variables

- let and let! are provided as tools for writing variables.
- The syntax is the same as for named subjects.
- subject includes the fact that it is a test object, but let and let! are not.
- I think it depends on your project whether you use them or not.
    - I'm in the camp of using before instead of let and let!
- The difference between let and let! is the timing of execution.
    - Let is executed at the time of use.
    - let! is executed at the place where it is written.
    - By the way, there are also those who don't use let and those who don't use let!

```ruby
context "When Book#title is a string" do
  let(:book){ Book.new(title: "RubyBook", author: "matz") }
  it "that the string connecting title and author is returned" do
    expect(book.title_with_author).to eq("RubyBook - matz")
  end
end
```

## System Spec

- E2E testing to test an entire Rails app by running a browser
    - Runs the browser so you can run JS
    - Slow because it runs in a browser
    - We can't explicitly POST, so we'll need to do things like submit the form in the browser
- It will be placed in the spec/system folder
- List of matchers available in Rails: https://relishapp.com/rspec/rspec-rails/docs/matchers
- The Capybara gem is often used to control the browser
- Capybara provides a set of methods and matchers for manipulating the browser
- List of matchers available in Capybara: https://rubydoc.info/github/teamcapybara/capybara/master/Capybara/Node/Matchers

## System Spec environment creation

Gemfile

```Gemfile
group :test do
  gem 'capybara'
  gem 'selenium-webdriver'
  gem 'webdriver'
end
```

spec/rails_helper.rb

```spec/rails_helper.rb
RSpec.configure do |config|
  ...
  config.before(:each, type: :system) do
    driven_by(:selenium_chrome_headless)
    # driven_by(:selenium_chrome)
    # driven_by(:rack_test)
  end
end
```

- $ bundle install

- When using headless Chrome `driven_by(:selenium_chrome_headless)`
- when using Chrome with head (normal) `driven_by(:selenium_chrome)`
- When using rack_test (fast, but browser features such as JS execution are not available) `driven_by(:rack_test)`

- If you get the following error, try to install ChromeDriver manually
  - https://chromedriver.chromium.org/downloads
      - Download and install it as /usr/local/bin/chromedriver
  - Or you can use homebrew to install it


```
Selenium::WebDriver::Error::SessionNotCreatedError:
            session not created: This version of ChromeDriver only supports Chrome version 87
            Current browser version is 90.0.4430.212 with binary path /Applications/Google Chrome.app/Contents/MacOS/Google Chrome
```

- When switching between "headless Chrome" and "head and Chrome", it is convenient to use the following mechanism to switch between them by using environment variables in the execution command

spec/rails_helper.rb

```ruby
if ENV['WITHHEAD'].present?
  driven_by(:selenium_chrome)
else
  driven_by(:selenium_chrome_headless)
end
```

- When using Chrome with head, define WITHHEAD environment variable by command
  - $ WITHHEAD=t bin/rspec spec 
- By default, headless Chrome is used
  - $ bin/rspec spec 

## Write a System Spec to GET

- Create spec/system/book_spec.rb as follows
    - If you don't have a spec/system folder, please create one as well.

```spec/system/book_spec.rb
require "rails_helper"

RSpec.describe "books", type: :system do
  it "GET /books" do
    visit "/books" # access /books with HTTP method GET
    expect(page).to have_text("Books") # Make sure the page displayed has the word Books in it
  end
end
```

- bundle exec rspec spec/system/book_spec.rb
- Matchers other than have_text can be found in the following Capybara documentation
  - https://rubydoc.info/github/teamcapybara/capybara/master/Capybara/Node/Matchers

## When System spec is set to NG, a scrapshot is automatically taken.

- When the System spec is set to NG, a screenshot is automatically taken.
- Useful for debugging.

spec/system/book_spec.rb

```spec/system/book_spec.rb
require "rails_helper"

RSpec.describe "books", type: :system do
  it "GET /books" do
    visit "/books"
    expect(page).to have_text("foo")
  end
end
```

```console
$ bin/rspec spec/system/book_spec.rb
...
[Screenshot Image]: /Users/igarashi/work/rails6132_ruby301_rspec/tmp/screenshots/failures_r_spec_example_groups_books_enables_me_to_create_widgets_252.png
...
```

- Even if you don't fail, you can take a screenshot with Capybara's function
- `save_screenshot('ss.png')`
    - Save the screenshot to /tmp/capybara/ss.png

## Write a System spec that POST from a form.

- Let's write a test that displays a form in the new screen, enters values in each field of the form, and presses a button to make a POST request.

spec/system/book_spec.rb

```spec/system/book_spec.rb
require "rails_helper"
RSpec.describe "books", type: :system do
  it "GET /books" do
    visit "/books"
    expect(page).to have_text("Books")
  end

  it "POST /books" do
    visit "/books/new"
    fill_in "Title", with: "RubyBook" # Enter "RubyBook" in the Title input field.
    fill_in "Author", with: "matz"
    click_button "Create Book" # Click the Create Book button.

    expect(page).to have_text("Book was successfully created.")
    expect(page).to have_text("Title: RubyBook")
    expect(page).to have_text("Author: matz")
  end
end
```

## Request Spec

- E2E tests for testing an entire Rails app without running a browser
    - Fast
    - Can explicitly write tests for methods other than HTTP method GET
    - Can't run JS because it doesn't run the browser
    - Can't even take a screenshot in case of failure
- Will be placed under spec/requests folder
- Mainly used for testing APIs (paths that return JSON)

## Create the API to be tested

- Create the API to be tested in order to write the Request Spec.
    - Create a path that returns { status: "ok" } in JSON when GET /status is done.
- $ bin/rails generate controller status
- spec/requests/status_request_spec.rb will be generated as well

config/routes.rb
```config/routes.rb
Rails.application.routes.draw do
  resources :books
  get 'status' => 'status#index', defaults: { format: 'json' }
end
```

app/controllers/status_controller.rb
```app/controllers/status_controller.rb
class StatusController < ApplicationController
  def index
  end
end
```

app/views/status/index.json.jbuilder
```
json.status "ok"
```

## Write a Request spec

spec/requests/status_spec.rb
```spec/requests/status_spec.rb
require 'rails_helper'
RSpec.describe "Statuses", type: :request do
  it "GET /status" do
    get "/status"
    expect(response).to have_http_status(200)
    expect(response.content_type).to eq("application/json; charset=utf-8")
    expect(response.body).to include({ status: "ok" }.to_json)
  end
end
```

## Using factory_bot to create test data

- factory_bot: A tool to create model data for testing
    - In Rails standard, there is a tool called fixture, but factory_bot is often used.
    - Comparison is described later.
- In our Rails app, we use the aforementioned factory_bot_rails gem
- The definition files are placed in spec/factories folder.
- When you scaffold, a template of factory is created in spec/factories/books.rb
- If you want to create a new factory definition file, you can generate it with the following command
  - $ bin/rails g factory_bot:model book

## Write the definition file for factory_bot

spec/factories/books.rb
```spec/factories/books.rb
FactoryBot.define do
  factory :book do # :book is the model name in all lowercase as a symbol
    title { "RubyBook" } # use "RubyBook" as data for column title
    author { "matz" } # use "matz" as data for column author
  end
end
```

- Write a block `{}` following the column, and write the data to be assigned in the block.
- Using this definition file, let's create the following data

## Create model data from factory_bot definition file.

- You can use `FactoryBot.create(:book)` to create a record in the DB and get a model object.
- You can also try this from the rails console.

```console
$ bin/rails c
irb> book = FactoryBot.create(:book)
irb> book
=> #<Book:0x... id: 1, title: "RubyBook", author: "matz", created_at: ..., updated_at: ...>
```

- If you want to create a model object without creating records in the DB, you can use the build method.
  - If you use the build method instead of the create method, you can create a model object without saving to the DB.
  - `FactoryBot.build(:book)`.
- You can also refer to the factory_bot documentation: https://github.com/thoughtbot/factory_bot/wiki

## Write a definition file with associations

- Create a relationship between book and variation.
  - Use variation models to represent types such as PDF and paper books.

```console
$ bin/rails g model variation kind book:references
$ bin/rails db:migrate
```

app/models/book.rb
```ruby
class Book < ApplicationRecord
  ...
  has_many :variations
  ...
end
```

spec/factories/variations.rb

```ruby
FactoryBot.define do
  factory :variation do
    kind { "PDF" } # change to "PDF"
    book { nil } # leave as default *1
  end
end
```

- Run from rails console

```console
$ bin/rails c
irb> book = FactoryBot.create(:book)
irb> book
=> #<Book id: 3, title: "RubyBook", memo: "Good", created_at: "2020-07-07 04:29:51", updated_at: "2020-07-07 04:29:51">
irb> variation = FactoryBot.create(:variation, book: book)
irb> variation
=> #<Variation id: 1, kind: "PDF", book_id: 3, created_at: "2020-07-07 04:30:11", updated_at: "2020-07-07 04:30:11">
irb> variation.book
=> #<Book id: 3, title: "RubyBook", memo: "Good", created_at: "2020-07-07 04:29:51", updated_at: "2020-07-07 04:29:51">
```

- It is recommended to associate them at object creation time.
- It is also possible to have the association in the factory definition, but it is very, very difficult to maintain
  - spec/factories/variations.rb *1, the line `book { nil }`.
  - This is because it becomes difficult to write when the number of models increases and the relationships become intertwined.
- Alternatively, I recommend using the following traits

## trait

- trait is a feature that allows you to write another pattern generation method in the factory definition
  - https://github.com/thoughtbot/factory_bot/blob/master/GETTING_STARTED.md#traits
- If you want to write relations in the factory definition, we recommend using traits.
- Sample code for a Book model with related destination variations

spec/factories/books.rb
```ruby
FactoryBot.define do
  # same as before                
  factory :book do
    title { "RubyBook" }
    author { "matz" }
  end

  # Add
  trait :with_variations do
    after(:create) do |book|
      book.variations.create!(kind: "paper book")
    end
    # (Not tested) Instead of the after method above, you can write this if you have a FactoryBot.
    # If you have a FactoryBot for the related destination instead of the after method above, you can write
    # FactoryBot.create_list(:variations, count, book: book)
  end
end
```

- You can create a factory in the following three ways
- `FactoryBot.create(:book)` : create a factory with only the base part without trait.
  - Create an unrelated object.
- `FactoryBot.create(:book, :with_variations)`: execute the `trait :with_variations` block in addition to the base part.
  - = Create the related destination together.
- `FactoryBot.create(:book, variations: [pdf_variation])`: specify pdf_variation as variations
  - pdf_variation is a pre-created Variation instance: `pdf_variation = Variation.find_by(kind: "PDF")`.
  - = Specify the associated destination

```console
irb> book = FactoryBot.create(:book, :with_variations)
irb> book
=> <Book id: 6, title: "RubyBook", author: "matz", created_at: "2021-05-26 09:47:06.869090000 +0000", updated_at: "2021-05-26 09:47:06.869090000 +0000">
irb> book.variations
=> [#<Variation id: 3, kind: "paper book", book_id: 6, created_at: "2021-05-26 09:47:06.874868000 +0000", updated_at: "2021-05-26 09:47:06.874868000 +0000">]
```

## sequence

- This function can be used to make a group of records numbered sequentially.
- You can read more about it in the "Perfect Rails" book 7-3.

```ruby
FactoryBot.define do
  factory :book do
    sequence { |i| "RubyBook vol.#{i}" }
  end
end
```

## fixture

- A tool for registering standard Rails test data.
    - But it's not used very often.
    - factory_bot is more used
- factory_bot creates the data when you write a create
- fixture creates all fixture data before the whole test starts
- If you can specify the test data to be used in each test, it reduces the coupling of the individual data.
    - I think this is one of the main reasons why factory_bot is preferred.

## Create test data with factory_bot and write System spec.

```ruby
require "rails_helper"

RSpec.describe "books", type: :system do
  it "enables me to create widgets" do
    book = FactoryBot.create(:book)
    visit "/books" # access /books with HTTP method GET
    expect(page).to have_text(book.title)
  end
end
````

## How to use mocks and stubs

- rspec-mocks
    - When you use the rspec-rails gem, rspec-mocks will be added as a dependency gem and available for use.
        - https://github.com/rspec/rspec-rails/blob/main/rspec-rails.gemspec#L47-L57
    - It's used a lot, so everyone knows the syntax.
    - Can be used with minitest
- There are other libraries like rr, mocha, etc. with different functions and syntax
    - https://relishapp.com/rspec/rspec-core/v/3-10/docs/mock-framework-integration

- If you want to use a mock other than rspec-mock, write the configuration in spec_helper.rb
    - I think the default configuration is `config.mock_with :rspec` (=rspec-mock) for rspec-mock

```ruby
config.mock_with :rspec do |mocks|
  ...
end
```

- As an example, let's consider the case where a book is raffled off for a prize

app/models/book.rb


```app/models/book.rb
class Book < ApplicationRecord
...
  def bonus
    return "author-signed photo" if lucky?
    "bookmarks"
  end

  def lucky?
    [true, false].sample
  end
end
```

- If you write the test code in a normal way, it will randomly succeed or fail.

spec/models/book_spec.rb
```ruby
describe "Book#bonus" do
  context "when lucky? is true" do
    it "returns Author-signed photo" do
      book = Book.new
      expect(book.bonus).to eq("Author-signed photo")
    end
  end
end
```

- In such cases, mocks can be used to solve the problem.
- We will use a mock to manipulate another method, lucky? which is called in the method under test, bonus.

spec/models/book_spec.rb
```ruby
describe "Book#bonus" do
  context "when lucky? is true" do
    it "returns Author-signed photo" do
      book = Book.new
      allow(book).to receive(:lucky?).and_return(true)
      expect(book.bonus).to eq("Author-signed photo")
    end
  end
end
```

- allow(target object).to receive(symbol of method name).and_return(return value)
- If you write it like this, you can change the behavior of only the method calls specified by receive in the target object.
- The method specified by receive will not call the original method, but will only return the return value specified by and_return.
- In addition to and_return, there are other tools available
  - https://relishapp.com/rspec/rspec-mocks/v/3-10/docs/configuring-responses
- Documentation of the allow method
  - https://relishapp.com/rspec/rspec-mocks/v/3-10/docs/basics/allowing-messages

- You can also check if the method specified by receive is called or not.

```ruby
describe "Book#bonus" do
  context "when lucky? returns true" do
    it "returns Author-signed photo" do
      book = Book.new
      allow(book).to receive(:lucky?).and_return(true)
      expect(book).to receive(:lucky?) # Write before executing the method call to be checked.
      expect(book.bonus).to eq("Author-signed photo")
    end
  end
end
```

- An object whose method return value has been tampered with is called a "mock" or "stub," as in the case of book here.
- When there is a function to check if the method has been called, it is called a "mock".
- When the only function is to tamper with the method return value, it is called a "stub.
- However, in practical use, either "mock" or "stub" will work, so there is no problem.

- In this example, we created a test object by changing the behavior of a part of an existing object.
- By using the double method, you can create a mock or stub without using an object of the target class.
- If you want to know more, please read the documentation.
  - https://relishapp.com/rspec/rspec-mocks/v/3-10/docs/basics/test-doubles
  - https://relishapp.com/rspec/rspec-mocks/v/3-10/docs/verifying-doubles

- Reference Materials
  - Description of the foundation
    - https://relishapp.com/rspec/rspec-mocks/v/3-10/docs
  - Various sample codes are written.
    - https://relishapp.com/rspec/rspec-core/v/3-10/docs/mock-framework-integration/mock-with-rspec

## A technique to check for exceptions

- A test to verify that an exception is thrown can be written as follows
    - Note that it is a block, unlike the other ways of writing expect.

```ruby
expect { ... }.to raise_error
expect { ... }.to raise_error(ErrorClass)
expect { ... }.to raise_error(ErrorClass, "message")
```

app/models/book.rb

```ruby
class Book < ApplicationRecord
...
  def take_pictures
    raise RuntimeError.new("Please refrain from taking photos")
  end
end
```

spec/models/book_spec.rb
```ruby
describe "Book#take_pictures" do
  context "When to call" do
    it "Exceptions must be thrown" do
      book = Book.new
      expect{ book.take_pictures }.to raise_error(RuntimeError, "Please refrain from taking photos")
    end
  end
end
```

## Technique to make it DRY

- There is a way to do this.
- If you don't use it carefully, it will be hard to read and maintenance will be difficult.

- shared_example
    - A technique to combine multiple it's.
    - If you use it too much, the test code will be hard to read, so it is not recommended.
    - https://relishapp.com/rspec/rspec-core/v/3-9/docs/example-groups/shared-examples

- shared_context
    - A technique to combine multiple contexts.
    - Not recommended, as overuse will make your test code hard to read.
    - https://relishapp.com/rspec/rspec-core/v/3-9/docs/example-groups/shared-context

## How to change the current time

- Rails 4.1 introduces TimeHelpers to change the time!
- If you're using the timecop gem, you're probably using outdated code
    - You can remove the timecop gem by replacing it with the TimeHelpers methods

Reference : http://api.rubyonrails.org/classes/ActiveSupport/Testing/TimeHelpers.html

### RSpec configuration using TimeHelpers

rails_helper.rb

```ruby
require 'active_support/testing/time_helpers' # Add
  config.include ActiveSupport::Testing::TimeHelpers # Add
end
```

### Settings for use in a development environment

Write these settings where you can include, for example spec/rails_helper.rb.

```ruby
require 'active_support/testing/time_helpers'
include ActiveSupport::Testing::TimeHelpers
```

### travel_to

https://api.rubyonrails.org/classes/ActiveSupport/Testing/TimeHelpers.html#method-i-travel_to

- The time will be fixed at the time specified in the argument, and will not move forward in time.
- To go back, use the travel_back method.
- If you call it with a block, it will automatically travel_back when you leave the block.
    - Time does not advance while in a block.
    - If you can call the travel_back method with a block, it's better because you won't forget to call it.

```ruby
it do
  travel_to(Time.current) do
    p Time.current #=> Mon, 11 Sep 2017 12:00:00 JST +09:00
    sleep 3
    p Time.current #=> Mon, 11 Sep 2017 12:00:00 JST +09:00
  end
end
```

### travel

https://api.rubyonrails.org/classes/ActiveSupport/Testing/TimeHelpers.html#method-i-travel

- It is similar to travel_to, but the difference is that the time is advanced by the interval specified in the argument and fixed.
- To go back, use the travel_back method as well.
- If you call it with a block, it will automatically travel_back when it leaves the block as well.

```ruby
Time.current # => Sat, 09 Nov 2013 15:34:49 EST -05:00
travel 1.day do # Proceed one day.
  User.create.created_at # => Sun, 10 Nov 2013 15:34:49 EST -05:00
end
Time.current # => Sat, 09 Nov 2013 15:34:49 EST -05:00
```

### freeze_time

- Alias for `travel_to Time.now`.
- You can also pass a block
- To return, use the travel_back method or the unfreeze_time method of the alias

## Multiple DB functionality

- The ability to access multiple DBs has been introduced in Rails 6.0.
- minitest will automatically build multiple DBs and run them in parallel.
- It's not available in RSpec yet, but it might be soon.
- You can read more about it in the "Perfect Rails" book chapter 3-3.

## CI(Continuas Integration)

- A service that automatically executes RSpec and other test code.
- Automatically executes tests triggered by PR creation, etc.
- GitHub Actions and CircleCI are famous.
- You can read more about them in Chapter 9 of the "Perfect Rails" book.

## Reference material (republished)

- RSpec reference page https://relishapp.com/rspec/rspec-rails/docs
- Rails Guide "Testing Rails Applications": https://guides.rubyonrails.org/testing.html
- Capybara: https://rubydoc.info/github/teamcapybara/capybara/master/Capybara/Node/Matchers

## Reference Materials (Books)

- Everyday Rails Testing with RSpec (English)
    - https://leanpub.com/everydayrailsrspec

- Perfect Ruby on Rails 2nd Edition (Japanse): https://gihyo.jp/book/2020/978-4-297-11462-6
    - This book contains explanations of various Rails features such as ActionMailer and test code as well.
    - I'm using minitest.

- Software Testing Techniques Practice Book (Japanse): https://gihyo.jp/book/2020/978-4-297-11061-1
    - A useful resource for those who want to learn how to create test patterns.

- Baba-san's approach to writing RSpec (Japanese)
    - https://qiita.com/kbaba1001/items/5333d325d0a76dd29581

- willnet's Clean Test Code Revised (Japanese)
    - https://blog.willnet.in/entry/2019/03/27/092642
    - How to write tests that don't overload your brain.
    - The author of the Perfect Rails book

- willnet's RSpec Style Guide (Japanese)
    - https://github.com/willnet/rspec-style-guide
    - The aforementioned willnet has implemented the guidelines for writing readable RSpec as a style guide.

- Yuki Iwasaki's Anatomy of a Rails System Test (Japanese)
    - https://speakerdeck.com/yusukeiwaki/railsfalse-sisutemutesutojie-pou-xue
    - A detailed explanation of what we're doing in Capybara and System Test.
