# Launch School Course 310 Notes

## Pre-Course

### Prototype Process for Web Apps

1. Idea
2. Wireframes
  - Low fidelity representation of the applicatin's UI
  - Not for design, but for communication, not only for other people, but with  yourself.
  - Allows us to see what the scope and workflow and features of the project are
  - Starting point for communication
  - TIPS
    - Don't over-design
    - Create several versions
      - It's focused on speed
      - Frees you from your attachment to your own idea/design
  - Serves as a scoping tool
3. Design
  - Layouts
  - Colors
  - Look & feel
  - Give wireframes as input, typically done with photoshop
4. Slicing
  - Turning design files into HTML mockups
  - Aiming for pixel perfection
5. Development

This process isolates different tasks into layers. This allows us to focus on the right things. Each layer allows us to explore many alternatives, and encourages creativity and explorations.

### The Design to Development Handoff

HTML mockups aren't backed by any kind of data on the back end, we typically just have links to all

### Seeding Data

This lets us play with the application using the dummy data.

We go under `/db/seeds.rb`. Here we can create a bunch of dummy models:

```ruby
# seeds.rb

Todo.create name: 'cook dinner'
Todo.create name: 'eat'
Todo.create name: 'do dishes'
```

Then we run:

```
rake db:seed
```

## Lesson 1

### The Rationale Behind Testing

Why test (automated tests)

Alternatives:

1. No tests at all.
  - Rely on customers to do the testing
  - Raises support costs
2. Rely on QA teams (verification teams)
  - Clear division betwen software development and testing
  - Developers work on features, code is thrown over the wall to the QA to test against the code base.
  - Problems:
    - Organizational mistrust: communication isn't very good
    - QA gets blamed for things (nobody likes the QA team). When the code works, the developers get the credit. Junior guys comes in as testers, QA team has a very high turnover, knowledge doesn't accumulate.
    - Release cycle - two teams are competing for time.
     - As a result, releases are likely to get delayed

How does the lack of automated tests affect development?

- People don't care as much about code quality.
  - When someone else does tests, developers get lazy/sloppy about quality. Things get thrown together, and as long as it works it's fine.
- Isolated knowledge: When you don't have automated tests, people work on their own components and are the only ones that are familiar with that component. People won't touch that component.
  - Sometimes this leads to code becoming overly complex in order for the dev to have more job security.
- How do you know things work at all?
  - How do you know when things break? (The regression problem)

### Technical Debt

A typical response to TDD is related to the time investment. At the beginning, this is true. As the size of the project grows, new features in an application without TDD cause an exponential growth in time investment, due to regressions. The problem of "I don't remember what I did 3 months ago" context switching.

The problem can cascade throughout the project, and may lead to rescue projects. Rescue projects always start by building a test suite.

This is the problem of technical debt. Time investment compounds over time. Rescue projects are a way of paying off technical debt.

### Unit, functional, and integration tests

Types of Tests

1. Unit tests
  - Components in isolation
  - Models, views, helpers, routes
2. Functional tests
  - Components inside a region
  - Controllers - one single request or response cycle
3. Integration tests
  - Follow a business workflow
  - Emulate the end user - filling forms, clicking buttons.
  - Go beyond one single request and response

There are multiple styles of testing in the rails community. You have to balance what you test and how much you test on each level. This is a matter of opinion.

This course we'll focus on unit tests and functional tests, and we'll only write integration tests on important business workflows.

We'll test on models and important helpers, controllers, and some browser-style tests.

We can think about it in terms of a cost-benefit situation. Unit tests are very easy and fast to write, while integration tests take the most amount of time. There are a lot of components involved, making it difficult to have good test coverage. Integration tests are also slow, so we have to mind that.

Think about coverage per time investment, and realism per time investment.

Integration tests are the most realistic, since it follows user workflows.

Here's what we'll focus on, in summary.

1. Maximize unit and functional tests
2. Integraion tests for the most important workflows.

### First test in rspec

We're going to use rspec as our testing framework.

Make sure we have the right gem in our Gemfile and it is installed, and use the following command to set up rspec within an application:

```
rails g rspec:install
```

We set options in `.rspec`, and configure rspec in `spec/spec_helper.rb`

We create directories for our components:

```
mkdir spec/models
```

This is where we put all the tests related to models. There, we create a spec for the `Todo` model.

```
touch spec/models/todo_spec.rb
```

Here we write our first spec:

```ruby
require 'spec_helper' # Gives us rails stuff

describe Todo do

end
```

To test, we run

```
rspec
```

RSpec is a BDD tool: instead of thinking about writing tests to verify functionality, we think about writing specifications. Rather than test perspective, we want to think about what components should do.

```ruby
require 'spec_helper'

describe Todo do
  it 'saves itself'
end
```

`it` is a keyword to describe a piece of functionality of a component.

So let's write the spec:

```ruby
require 'spec_helper'

describe Todo do
  it 'saves itself' do
    # Setup the data
    todo =Todo.new name: 'cook dinner', description: 'I love cooking!'

    # Perform the action
    todo.save

    # Verify the functionality
    Todo.first.name.should == 'cook dinner'
  end
end
```

`should` is a keyword for verification. Call the matcher on it (`==`, `>=`, etc).

### Github Flow and the Code Review Process

The workflow:

- Anything in the `master` branch is deployable.
- To working on something now, create a descriptively named branch off `master` (a feature branch).
- Commit to that branch locally and regularly push your work to the same named branch on the server.
- When ready for merging, issue a pull request.

## Week 1

### From Test Later to Test Driven

First, we have "Code first, test later." Implement first, and wrap the test around it later. This is what we have done so far. This guards against regressions.

Second, we have "code a little, test a little". Here the feedback loop is smaller.

Third, we have "Test First" (TFD). Here we write the test first before any implementation code. Run the test first and make sure it fails, then we write the code to make the test pass. The benefit is that it challenges us to think up front about what we want to do. We lock in the behavior first, and then we write the code to hit that target.

Then we have TDD. The most important term here is "driven." We use tests not only to set a target with our code, but also to interactively drive out the implementation code. This lets us gradually increase the complexity of the code.

Next we have Test Driven Design--similar to TDD, but on a more system/global/architecture level: the collaboration between different objects.

Any one of these isn't superior to the others. It depends on the problem we're trying to solve. As the implementation grows in complexity, we reach more toward TDD and test driven design.

Here we're going to focus on code a little test a little, reaching for TFD for the more complex things, and touch a little bit on TDD.

### TDD and Red-Green-Refactor

First we write the backbone for our tests. Here we have a todo model, and we're going to test a method that checks if the model doesn't have a description.

```ruby
# todo_spec.rb

require 'spec_helper'

describe Todo do
  describe '#name_only?' do
    it 'returns true if the description is nil'
    it 'returns true if the description is an empty string'
    it 'returns false if the description is a non-empty string'
  end
end
```

Now we have the tests. If we run the test suite, we get three pending tests.

So we set up the tests:

```ruby
# todo_spec.rb

require 'spec_helper'

describe Todo do
  describe '#name_only?' do
    it 'returns true if the description is nil' do
      todo = Todo.new name: 'cook dinner'
      expect(todo.name_only?).to be(true)
    end

    it 'returns true if the description is an empty string' do
      todo = Todo.new name: 'cook dinner', description: ''
      expect(todo.name_only?).to be(true)
    end

    it 'returns false if the description is a non-empty string' do
      todo = Todo.new name: 'cook dinner', description: 'maybe fish?'
      expect(todo.name_only?).to be(true)
    end
  end
end
```

Here our tests fail. We need to define the method `name_only?`.

```ruby
# todo.rb

class Todo < ActiveRecord::Base
  has_many :tags

  def name_only?
  end
end
```

The tests still fail. The first, for example, expects true and gets nil. We always start with the easiest implementation for the test to pass.

```ruby
# ...
  def name_only?
    true
  end
# ...
```

This passes for the first two tests! When the third test is written, then get a failure. This forces us to write real implementation code. This is referred to as generalization or trianglization.

We can then start to implement the method:

```ruby
# ...
  def name_only?
    description.nil? || description == ''
  end
# ...
```

Here our tests pass! We now can refactor:

```ruby
# ...
  def name_only?
    description.blank?
  end
# ...
```

This is a method Rails gives us for a column that is either nil or empty.

This is the red-green-refactor approach. We start with a failing test, make it pass, and then since we have a functionally working component, we can refactor it accordingly.

### Member and Collection Routes

#### Collection routes

Consider a todo app's router:

```ruby
TodoApp::Application.routes.draw do
  root to: 'todos#index'

  resources :todos, only: [:index]
end
```

Say we want to implement a search path within `todos`, say `/todos/search`. This has nothing to do with a specific todo, so we use a `collection` route.

```ruby
# ...
  resources :todos, only: [:index] do
    collection do
      get 'search', to: 'todos#search'
    end
  end
# ...
```

This creates a route that matches `/todos/search`, mapped to `get`, with the named path `search_todos`.

#### Member routes

Member routes, like collection routes, are nested under a certain resource. Rather than the index path, they are appended after a specific resource id: `/todo/1/highlight`.

```ruby
resources :todos, only: [:index] do
  # ...

  member do
    post 'highlight', to: 'todos#highlight'
  end
end
```

This creates a route that matches `/todos/:id/highlight`, mapped to `get`, iwth the named path `highlight_todo`.

### Custom Form Builders

Suppose we have we have a Todo app, and want to create a form to create a todo. Here's the HAML for that form:

```haml
%section.new_todo
  %h3 Add a new todo
  = form_for @todo do |f|
    = f.label :name
    = f.text_field :name
    = f.label :description
    = f.text_area :description, rows: 6
    &br
    = f.submit 'Add a Todo'
```

If we want to display validation errors attached to the object, we need to add something to the form:

```haml
/ ...
  = form_for @todo do |f|
    - if @todo.errors?
      / ...

```

This would be very cumbersome, especially if we have a lot of places in which we need this. To get around this, we can use custom form builders.

```ruby
# app/helpers/my_form_builder.rb

class MyFormBuilder < ActionView::Helpers::FormBuilder
  def label(method, text = nil, options = {}, &block)
    errors = object.errors[method.to_sym]
    if errors
      text += " <span class=\"error\">#{errors.first}</span>"
    end
    super(method, text.html_safe, options, &block)
  end
end
```

Here we are overwriting the standard form builder, so we can append the errors to the form's labels. To use it we can modify our call to `form_for` thusly:

```haml
%section.new_todo
  %h3 Add a new todo
  = form_for @todo, builder: MyFormBuilder do |f|
    / ...
```

If we get validation errors, they'll appear next to the label:

```
Name `can't be blank`
```

If we don't want to do this every single time we create a form, we can just create a wrapper helper in `ApplicationHelper`:

```ruby
# app/helpers/application_helper.rb

module ApplicationHelper
  # Make sure it has the same parameters.
  def my_form_for(record, options = {}, &proc)
    form_for(record, options.merge!({ builder: MyFormBuilder }), &proc)
  end
end
```

We merge in the builder option so we can just make a call to `my_form_for`.

### Form Builders in the Wold

#### Formtastic

Gives us `semantic_form_for`. Allows us to forgo labels.

#### simple_form

Gives us `simple_form_for`. Integrates with Bootstrap and Zurb Foundation. Only really works if your style is similar to the default. If you want to customize, you need to fight with the DSL.

It has a default mapping, so it allows us to generate form elements based on the datatypes.

#### bootstrap_form

Gives us `bootstrap_form_for`. It doesn't have a lot of DSL with it, but it allows us to add bootstrap's classes to elements and format it nicely. Keeps us from writing the verbose markup that comes with bootstrap. It automatically creates the control groups for us automatically so we can just add fields generated by the form builder.

## Week 2

### The Built-in Rspec Matchers

Found at: https://relishapp.com/rspec/rspec-expectations/v/3-5/docs/built-in-matchers

### Controller Tests

Typically categorized under functional tests.

Suppose we have a todo app with Todo models. We have a controller with `index`, `new`, and `create` actions.

We create a new spec:

```
touch spec/controllers/todos_controller_spec.rb
```

```ruby
# todos_controller_spec.rb
require 'spec_helper'

describe TodosController do

  # The convention is to name the block 'VERB action'
  describe 'GET index' do
  end
end
```

In a rails controller, the action has two functions: to get the data ready, and to render the templates (or redirect). So let's describe the two functions:

```ruby
# todos_controller_spec.rb

# ...
  describe 'GET index' do
    it 'sets the @todos variable'
    it 'renders the index template'
  end
# ...
```

First we do the first example. We want to make sure the variable is set, and it has all the todos in the database.

```ruby
# todos_controller_spec.rb

# ...
  it 'sets the @todos variable' do
    cook = Todo.create name: 'cook'
    sleep = Todo.create name: 'sleep'

    get :index # Issue an HTTP GET for the #index action.
    expect(assigns(:todos)).to match_array([cook, sleep])
  end
# ...
```

Then we work on the second example. `rspec-rails` gives us a way to make sure a template is rendered:

```ruby
# todos_controller_spec.rb

# ...
  it 'renders the index template' do
    get :index
    expect(response).to render_template(:index)
  end
# ...
```

Then we can test the `new` action:

```ruby
# todos_controller_spec.rb

# ...
  describe 'GET new' do
    before(:each) do # Create a hook to keep the specs DRY
      get :new
    end

    it 'sets the @todo variable' do
      expect(assigns(:todo)).to be_new_record # This is an rspec-rails matcher
      expect(assigns(:todo)).to be_instance_of(Todo) # Our var is a Todo obj
    end

    it 'renders the new template' do
      expect(response).to render_template(:new)
    end
  end
# ...
```

Then we test the `create` action:

```ruby
# todos_controller_spec.rb

# ...
  describe 'POST create' do
    context 'valid input' do # We create contexts for each case
      # Keep it dry:
      before :each { post :create, todo: { name: 'cook', description: 'hi' } }

      # Query database, make sure the record matches:
      it 'creates the todo record' do
        expect(Todo.first.name).to eq('cook')
        expect(Todo.first.description).to eq('hi')
      end

      it 'redirects to the root path' do
        # rspec-rails gives us the redirect_to matcher.
        expect(response).to redirect_to(root_path)
      end
    end

    context 'invalid input' do
      before :each { post :create, todo: { description: 'hi' } }
      it 'does not create a todo' do
        expect(Todo.count).to eq(0)
      end

      it 'renders the new template' do
        expect(response).to render_template(:new)
      end
    end
  end
# ...
```

### Dealing with Cardinality and Boundary Conditions

Techniques for growing test suites.

Suppose we want to introduce the concept of tags, a M:M relationship. On our index page, let's say we want to display tags inline with each todo, but only display four at a time, with an elipsis to signify more than four tags.

So in our `Todo` model test, we want to describe a method `#display_text`

```ruby
# todo_spec.rb

# ...
describe Todo do
  # ...

  describe '#display_text' do

  end
end
```

We want to describe all of the following cases:

0, 1, many, boundary condition (if applicable)

```ruby
# todo_spec.rb

# ...
  describe '#display_text' do
    before :each { @todo = Todo.create name: 'cook' }

    # '0' condition
    context 'zero tags' do
      it 'displays the name only' do
        expect(@todo.display_text).to eq('cook')
      end
    end

    # '1' condition
    context 'one tag' do
      before :each { @todo.tags.create name: 'home' }

      it 'displays the only tag with the word "tag"' do
        expect(@todo.display_text).to eq('cook (tag: home)')
      end
    end

    # 'many' condition
    context 'multiple tags' do
      before :each do
        @todo.tags.create name: 'home'
        @todo.tags.create name: 'urgent'
      end

      it 'displays tags with the word "tags"' do
        expect(@todo.display_text).to eq('cook (tags: home, urgent)')
      end
    end

    # 'boundary' condition
    context 'more than four tags' do
      before :each do
        @todo.tags.create name: 'home'
        @todo.tags.create name: 'urgent'
        @todo.tags.create name: 'help'
        @todo.tags.create name: 'book'
        @todo.tags.create name: 'patience'
      end

      it 'displays only up to four tags' do
        expect(@todo.display_text).to eq('cook (tags: home, urgent, help, book, more...')
      end
    end
  end
#...
```

The final implementation for our `display_text` method is below:

```ruby
# todo.rb

class Todo < ActiveRecord::Base
  # ...

  def display_text(tag_limit = 4)
    if tags.any?
      name + " (#{'tag'.pluralize(tags.length)}: #{tags.map(&:name).first(tag_limit).join(', ')}#{', more...' if tags.count > tag_limit})"
    else
      name
    end
  end
end
```

Now we can refactor. Add a feature in the red, and refactor in the green. `display_text` is pretty complex, so let's clean it up.

First we can pull the big hairy string out:

```ruby
# todo.rb

# ...
  def display_text(tag_limit = 4)
    if tags.any?
      name + tag_text(tag_limit)
    else
      name
    end
  end

  private

  def tag_text(tag_limit)
    " (#{'tag'.pluralize(tags.length)}: #{tags.map(&:name).first(tag_limit).join(', ')}#{', more...' if tags.count > tag_limit})"
  end
# ...
```

Then we can make `display_text` a one-liner:

```ruby
# todo.rb

# ...
  def display_text(tag_limit = 4)
    name + tags.any? ? tag_text(tag_limit) : ''
  end
# ...
```


Or we can put the logic that decides whether or not to display the tag text in `tag_text`:

```ruby
# todo.rb

# ...
  def display_text(tag_limit = 4)
    name + tag_text
  end

  def tag_text(tag_limit)
    if tags.any?
      " (#{'tag'.pluralize(tags.length)}: #{tags.map(&:name).first(tag_limit).join(', ')}#{', more...' if tags.count > tag_limit})"
    else
      ''
    end
  end
# ...
```

### An Alternative Style of Rspec

So with each of our test cases we always assign a 'todo' variable to the same thing. We can use `let` to factor it out:

```ruby
# todo_spec.rb

# ...
  describe '#display_text' do
    let :todo { Todo.create name: 'cook' }

    # '0' condition
    context 'zero tags' do
      it 'displays the name only' do
        expect(todo.display_text).to eq('cook')
      end
    end

    # '1' condition
    context 'one tag' do
      before :each { todo.tags.create name: 'home' }

      it 'displays the only tag with the word "tag"' do
        expect(todo.display_text).to eq('cook (tag: home)')
      end
    end

    # 'many' condition
    context 'multiple tags' do
      before :each do
        todo.tags.create name: 'home'
        todo.tags.create name: 'urgent'
      end

      it 'displays tags with the word "tags"' do
        expect(todo.display_text).to eq('cook (tags: home, urgent)')
      end
    end

    # 'boundary' condition
    context 'more than four tags' do
      before :each do
        todo.tags.create name: 'home'
        todo.tags.create name: 'urgent'
        todo.tags.create name: 'help'
        todo.tags.create name: 'book'
        todo.tags.create name: 'patience'
      end

      it 'displays only up to four tags' do
        expect(todo.display_text).to eq('cook (tags: home, urgent, help, book, more...')
      end
    end
  end
#...
```

Now if we look at the last line of each test case, we're testing the same method. So we can specify a subject using `let`:

```ruby
# todo_spec.rb

# ...
  describe '#display_text' do
    let :todo { Todo.create name: 'cook' }
    let :subject { todo.display_text }

    # '0' condition
    context 'with zero tags' do
      it 'displays the name only' do
        expect(subject).to eq('cook')
      end
    end

    # '1' condition
    context 'with one tag' do
      before :each { todo.tags.create name: 'home' }

      it 'displays the only tag with the word "tag"' do
        expect(subject).to eq('cook (tag: home)')
      end
    end

    # 'many' condition
    context 'with multiple tags' do
      before :each do
        todo.tags.create name: 'home'
        todo.tags.create name: 'urgent'
      end

      it 'displays tags with the word "tags"' do
        expect(subject).to eq('cook (tags: home, urgent)')
      end
    end

    # 'boundary' condition
    context 'with more than four tags' do
      before :each do
        todo.tags.create name: 'home'
        todo.tags.create name: 'urgent'
        todo.tags.create name: 'help'
        todo.tags.create name: 'book'
        todo.tags.create name: 'patience'
      end

      it 'displays only up to four tags' do
        expect(subject).to eq('cook (tags: home, urgent, help, book, more...')
      end
    end
  end
#...
```

The video puts everything into contexts, but we've already done this.

Then, since we have a subject we can shorten all of our `it` calls:

```ruby
# todo_spec.rb

# ...
  describe '#display_text' do
    let :todo { Todo.create name: 'cook' }
    let :subject { todo.display_text }

    # '0' condition
    context 'with zero tags' do
      it { should eq('cook') }
    end

    # '1' condition
    context 'with one tag' do
      before :each { todo.tags.create name: 'home' }

      it { should eq('cook (tag: home)') }
    end

    # 'many' condition
    context 'with multiple tags' do
      before :each do
        todo.tags.create name: 'home'
        todo.tags.create name: 'urgent'
      end

      it { should eq('cook (tags: home, urgent)') }
    end

    # 'boundary' condition
    context 'with more than four tags' do
      before :each do
        todo.tags.create name: 'home'
        todo.tags.create name: 'urgent'
        todo.tags.create name: 'help'
        todo.tags.create name: 'book'
        todo.tags.create name: 'patience'
      end

      it { should eq('cook (tags: home, urgent, help, book, more...') }
    end
  end
#...
```

Pros:

- Takes all the common setups for each case and factors it out.

Cons:

- If you have a lot of test cases, the setup would be very far (in LOC) from where we describe each test.

`let` uses lazy evaluation. It doesn't execute the code when we call `let`. It'll evaluate it when it needs it, basically defining a proc for us to move around.

If we don't want lazy evaluation to occur (i.e., if we want the block to be evaluated right when the call to `let` happens), we can use `let!`:

```ruby
# ...
  let! :todo { Todo.create name: 'cook' }
# ...
```

More information on `let` and its rationale here:

http://stackoverflow.com/questions/5359558/when-to-use-rspec-let/5359979#5359979

### The Single Assertion Principle

This states that each test case should only have one assertion. So, for each `it` call we want to have a single expectation.

There are exceptions, for instance when we're testing new records:

```ruby
assigns(:todo).should be_new_record
assigns(:todo).should be_instance_of(Todo)
```

### Object Generation with Fabrication

Suppose we have a huge object with ten attributes and five required attributes. Then every time we create a `Todo` we have to supply five attributes. These can change over time, so we'd have to change the tests as well.

To get around this we can use an object generator framework like Fabrication. Fabrication supports a bunch of ORMs and provides a nice DSL.

First we create a new directory:

```
mkdir spec/fabricators
touch spec/fabricators/todo_fabricator.rb
```

```ruby
# todo_fabricator.rb

Fabricator(:todo) do
  name { 'cook' }
end
```

Then in our `let` call:

```ruby
# todo_spec.rb

# ...
  descrbie '#display_text' do
    let :todo { Fabricate :todo }
    # ...
  end
# ...
```

We can also associate our fabricator with another AR object:

```ruby
# todo_fabricator.rb

Fabricator :todo do
  name { 'cook' }
  user { Fabricate :user }
end
```

It'll now fabricate a user. We can just call `user` and not pass it a block. `Fabricator` will know what to do:

```ruby
# todo_fabricator.rb

Fabricator :todo do
  name { 'cook' }
  user
end
```

This will allow us to run our tests with minimal desctracting setup.

`Fabricate :todo` will create an object in the database. If you don't want to do this, we just need to call `build` on `Fabricate`:

```ruby
let :todo { Fabricate.build :todo }
```

This will create a `Todo` object in memory.

### Generate Fake Data

We can generate fake data using a data faker. This will allow us to avoid the uniqueness problem.

The Faker gem will let us do that--it can generate addresses, emails, names, and so on. Let's use Faker for our fabrication:

```ruby
# todo_fabricator.rb

Fabricator :todo do
  name { Faker::Lorem.words(5).join(' ') }
  user
end

Fabricator :user do
  email { Faker.Internet.email }
end
```
