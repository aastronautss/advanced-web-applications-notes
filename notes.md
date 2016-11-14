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

## Week 3

### Growing Complexity Guided by Tests

Going back to our todo app, let's say we want to add complexity. For example, we want our app to parse out parts of the todo's title ('study at home' is recognized as being at the location 'home'), and automatically creates tags based on that information (location:home, in this case). We're going to cover how we want to increase complexity, and further, how we'll recover from dead ends.

We start by writing tests for `TodosController`, in the case of `POST create`. We create a context:

```ruby
# todos_controller_spec.rb

# ...
  context 'with inline locations' do
    it 'creates a tag with one location' do
      post :create, todo: { name: 'cook at home' }
      expect(Tag.all.map(&:name)).to eq(['location:home'])
    end
  end
# ...
```

We're hitting the create action with a new todo, and we're expecting the tags to be parsed.

Then we go to the `create` action in the controller:

```ruby
# todos_controller.rb

# ...
  def create
    @todo = Todo.new params[:todo]
    if @todo.save
      location_string = @todo.name.split('at').last.strip
      @todo.tags.create name: "location:#{location_string}"
    # ...
  end
# ...
```

This passes! Next we want to write another test case to create tages with two tags:

```ruby
# todos_controller_spec.rb

# ...
  it 'creates two tags with two locations' do
    post :create, todo: { name: 'cook at home and work' }
    expect(Tag.all.map(&:name)).to eq(['location:home', 'location:work'])
  end
# ...
```

This test fails. If we have multiple locations, we want to take them out and separate them further.

```ruby
# todos_controller.rb

# ...
  def create
    @todo = Todo.new params[:todo]
    if @todo.save
      location_string = @todo.name.split('at').last.strip
      locations = location_string.split('and').map(&:strip)
      locations.each do |location|
        @todo.tags.create name: "location:#{location}"
      end
    # ...
  end
# ...
```

It isn't apparent in these notes, but we're making small steps toward making the tests pass. We want to keep track of where we are so we don't have to hack away at the implementation code.

So now we want to test multiple tags:

```ruby
# todos_controller_spec.rb

# ...
  it 'creates two tags with multiple locations' do
    post :create, todo: { name: 'cook at home, work, school, and library' }
    expect(Tag.all.map(&:name)).to eq(['location:home', 'location:work', 'location:school', 'location:library'])
  end
# ...
```

So we want to be able to follow natural english. We'll need a more flexible way of tokenizing the string. The answer is a regex! `/and|\,/i` will match `and` and `,`, but we'll need to worry about the Oxford comma, since it'll create empty strings. So, we can just put a guard clause when we go to create the tag:

```ruby
# todos_controller.rb

# ...
  def create
    @todo = Todo.new params[:todo]
    if @todo.save
      location_string = @todo.name.split('at').last.strip
      locations = location_string.split(/and|\,/i).map(&:strip)
      locations.each do |location|
        @todo.tags.create name: "location:#{location}" unless location.blank?
      end
    # ...
  end
# ...
```

### Interactive Debugging for Solution Discovery

When we finish a feature we typically want to do a sanity test by running it in a browser. Our current code has a bug where it'll create a location tag even if we don't specify an 'at' clause.

Instead of diving into the code, we'll write a test case that wraps around the bug.

Outside the context of `'with inline locations'`, we put a test case:

```ruby
# todos_controller_test.rb

# ...
  it 'does not create tags without inline locations' do
    post :create, todo: { name: 'cook' }
    expect(Tag.count).to eq(0)
  end
# ...
```

With our current code this test fails.

We have the line which seems to be problematic:

```ruby
# todos_controller.rb

# ...
  location_string = @todo.name.split('at').last.strip
# ...
```

We drop a `binding.pry` after that line of code. To run the code within the context of the test case, we use the line number when we use the `rspec` command. In the video's case, it's line 53:

```
rspec ./spec/controllers/todos_controller_spec.rb:53
```

When we examine `location_string`, we find out that it is `'cook'`. We'd expect it to be blank or nil. When we call split, if nothing matches the delimiter we specify, it just returns a one-element array with the same string.

Instead of split, we can use `slice` with a regex:

```ruby
'cook'.slice /.*at(.*)/, 1
```

This will capture anything that occurs after an 'at'. We can replace our existing line:

```ruby
# todos_controller.rb

# ...
  def create
    @todo = Todo.new params[:todo]
    if @todo.save
      location_string = @todo.name.slice(/.*at(.*)/, 1).strip
      locations = location_string.split(/and|\,/).map(&:strip)
      locations.each do |location|
        @todo.tags.create name: "location:#{location}" unless location.blank?
      end
    # ...
  end
# ...
```

If there's nothing being returned from the regex, it'll try to call `strip` on `nil`. So we can just call `try`:

```ruby
location_string = @todo.name.slice(/.*at(.*)/, 1).try(:strip)
```

The next line will throw, so we can create an `if` branch:

```ruby
# todos_controller.rb

# ...
  def create
    @todo = Todo.new params[:todo]
    if @todo.save
      location_string = @todo.name.slice(/.*at(.*)/, 1).try(:strip)
      if location_string
        locations = location_string.split(/and|\,/i).map(&:strip)
        locations.each do |location|
          @todo.tags.create name: "location:#{location}" unless location.blank?
        end
      end
    # ...
  end
# ...
```

Our tests now pass. It'll probably be beneficial to put that location parsing into another method.

### Respond to Feature Changes

Say we have our todo app and have launched it. Users are finding that if they create a todo named something like `eat an apple`, the `at` is parsed as a location delimiter, and is tagged under `location:an apple`.

We now how to deal with this. We create a new test case:

```ruby
  it 'does not create tags with "at" in a word without inline locations' do
    post :create, todo: { name: 'eat an apple' }
    expect(Tag.count).to eq(0)
  end
```

This will fail at first.

The fix is pretty easy. We need to make it so it only parses `at`s that are their own words.

```ruby
location_string = @todo.name.slice(/.*\bat\b(.*)/, 1).try(:strip)
```

`\b` denotes the boundary of a word.

Now, we notice that 'get good at swimming' triggers our location tag creation. This is really part of a phrase. This is not so much a bug as it is a flaw in the design of our application. We didn't think that it could be a phrase or something similar.

So we need another direction. Instead of using `at` as our tokenizer, we need to do something else.

One solution is to only use the upcase `AT`, and we will just need to train our user to use the app in this fashion.

First we need to change our specification. We create a spec:

```ruby
  it 'creates a tag with upcase AT' do
    post :create, todo: { name: 'shop AT the apple store' }
    expect(Tag.all.map(&:name)).to eq(['location:the apple store'])
  end
```

If we implement the code to pass this test, we have to know that we'll break all the other tests. That's OK.

```ruby
location_string = @todo.name.slice(/.*\bAT\b(.*)/, 1).try(:strip)
```

This now breaks our other tests, so we need to adjust all our other tests to make it so they're using the new upcase `AT`.

It's very common that we'll have to change a lot of code for these direction changes.

This is the reason why we create our initial test before we change the code. It serves as our protection--as long as it passes, we know that the new feature specification is implemented. We can safely go back and change our other tests.

### Transactions

Going to Myflix, we want to make sure the list order values in our queue are integers.

The traditional Rails form lets us submit to one record, and if there are errors they'll bounce back. In the queue, we're updating list orders as a batch operation. All the items in the queue have to be saved, or, if there is a validation error with any of the items, we have to roll back the items. This is the concept of transactions.

A transaction is a batch operation that makes it so all items have to succeed. If one or more fails, all the changes are rolled back.

### Structural Refactor

We want to refactor `TodosController#create`, since we have a lot of logic for creating tags that shouldn't be in a controller. A controller is more of a coordinator, delegating tasks to different models. Let's push it down to the model level.

We create a `save_with_tags` method on the `Todo` model, and move the logic from the controller to that method:

```ruby
# todo.rb

# ...
  def save_with_tags
    location_string = @todo.name.slice(/.*\bAT\b(.*)/, 1).try :strip

    if location_string
      locations = location_string.split(/\,|and/).map(&:strip)
      locations.each do |location|
        @todo.tags.create name: "location:#{location}"
      end
    end
  end
# ...
```

Now we call that method from the controller:

```ruby
# todos_controller.rb

# ...
def create
  # ...
  if @todo.save_with_Tags
  # ...
end
# ...
```

We know that we want to return truthy if the save is successful, falsey otherwise. So we change the method (we also remove the references to `@todo`:

```ruby
# todo.rb

# ...
  def save_with_tags
    if save # Save the todo, which is a new record
      location_string = name.slice(/.*\bAT\b(.*)/, 1).try :strip

      if location_string
        locations = location_string.split(/\,|and/).map(&:strip)
        locations.each do |location|
          tags.create name: "location:#{location}"
        end
      end
      true # make sure it returns a truthy value
    else
      false # return false and don't make the tags if it fails
    end
  end
# ...
```

Running rspec, our tests pass. With a comprehensive test suite, we can refactor with confidence.

The option to write tests for our new method isn't very straightforward. The new code is covered by our existing controller tests. If it's a simple refactor like this one, we don't really need to move the test. However, if we're using the new code elsewhere or implementing new features with it, we'd want to write tests at the model level and move the relevant tests from the controller level to the model level.

Now, our method is a little to long, so let's refactor it.

```ruby
# todo.rb

# ...
  def save_with_tags
    if save # Save the todo, which is a new record
      create_location_tags
      true # make sure it returns a truthy value
    else
      false # return false and don't make the tags if it fails
    end
  end
# ...

# ...
  private

  def create_location_tags
    location_string = name.slice(/.*\bAT\b(.*)/, 1).try :strip

    if location_string
      locations = location_string.split(/\,|and/).map(&:strip)
      locations.each do |location|
        tags.create name: "location:#{location}"
      end
    end

  end
# ...
```

This makes our code more declarative and easier to read.

### Skinny Controller, Fat Model

We've moved a bunch of logic from the controller to the model. This is a very common refactor in rails, as well as a well-known architectural principle: Skinny Controller, Fat Model.

Our controller needs to stay pretty simple, while the model should be the one that knows how to do stuff.

Reference:

http://weblog.jamisbuck.org/2006/10/18/skinny-controller-fat-model

### Further Notes on SCFM Structure

We typically don't want to have functionality that handles form input (e.g. params hashes) living on the model layer, since we don't want the model tightly coupled with forms. This is the reason we have the MVC architecture: so we can have a controller to handle the communication between the views/router and the models.

### Rspec Macros

In our Todos app, we've added some simple authentication, in which if we see if the user is logged in and redirect to the dashboard if he/she is.

```ruby
class PagesController < ApplicationController
  def front
    redirect_to todos_path if current_user
  end
end
```

We want to add a `before_action` to each action related to dealing with the todo list, and this will break our tests due to lack of authenticated.

Requiring authentication is pretty common with our tests, since a lot of our controller tests use actions that will require authentication. We could set a user with each of our specs, but we can instead use a macro to DRY them up.

First we can use a `before` (in this example we're modifying the `TodosController` specs):

```ruby
# todos_controller_spec.rb

describe TodosController do
  before do
    john = Fabricate :user
    session[:user_id] = john.id
  end

  # The rest of the specs
end
```

Since we're putting the `before` outside of all of our `describe`s, it will be run before all of our specs on that controller.

This doesn't look quite as obvious what it's doing, so we can move the logic into a macro. This way we can use this in multiple specs.

We create a new directory:

```
mkdir spec/support
```

Rspec automatically loads everything in the `support` dir. Here we can put a file called `macros.rb`.

```ruby
# spec/support/macros.rb

def set_current_user
  john = Fabricate :user
  session[:user_id] = john.id
end
```

In our `before` we can simply make a call to that method.

```ruby
# todos_controller_spec.rb

describe TodosController do
  before { set_current_user }

  # ...
end
```

Sometimes we need to refer to the user, so we can create another macro called `current_user`:

```ruby
# macros.rb

# ...
def current_user
  User.find session[:user_id]
end
```

### Shared Examples in Rspec

Now that we have some macros for signing in, we want to test that for all of our actions, the user is redirected to the login page. To do this, we first need to clear the session, since our `before` to create the session is called before each test case.

We create another macro called `clear_current_user`:

```ruby
# macros.rb

# ...
def clear_current_user
  session[:user_id] = nil
end
```

Now we can make an expectation for our `todos_controller GET index` for the case where there is no signed-in user.

```ruby
# todos_controller_spec.rb

# ...
  describe 'GET index' do
    # ...

    it 'redirects user to the root path when they are not signed in' do
      clear_current_user
      get :index
      expect(response).to redirect_to(root_path)
    end
  end
# ...
```

This gives us a better sematics to work with using the macros for both cases. Now we want to test this same case for all of our other actions for which we want to require a logged in user:

We could copy and paste this, but rspec gives us the ability to keep it DRY using shared examples.

We create a file in `spec/support` called `shared_examples.rb`.

```ruby
# shared_examples.rb

shared_examples 'require_sign_in' do
  it 'redirects to the front page' do
    expect(response).to redirect_to(root_path)
  end
end
```

Now in our spec for our `todos_controller#index` action, we can replace the example with:

```ruby
# todos_controller_spec.rb

# ...
  describe 'GET index' do
    # ...

    it_behaves_like 'require_sign_in'
  end
# ...
```

This isn't enough, since our shared example doesn't clear the session or hit `get :index`. So we can add a context for this case:

```ruby
# todos_controller_spec.rb

# ...
  describe 'GET index' do
    # ...

    context 'not signed in' do
      before do
        clear_current_user
        get :index
      end

      it_behaves_like 'require_sign_in'
    end
  end
# ...
```

Now our tests pass. This doesn't seem much better, since we're just moving a single line around. We could add the before block to the shared example, but the problem is that `get :index` is unique to our `index` action. There is a way we can pass in the code to be executed within the shared example. In this instance we'll use a block, which we will pass in to `it_behaves_like`:

```ruby
# todos_controller_spec.rb

# ...
  describe 'GET index' do
    # ...

    context 'not signed in' do
      it_behaves_like 'require_sign_in' do
        let(:action) { get :index }
      end
    end
  end
# ...
```

We then add a call to the action to our shared example:

```ruby
# shared_examples
shared_examples 'require_sign_in' do
  it 'redirects to the front page' do
    clear_current_user
    action
    expect(response).to redirect_to(root_path)
  end
end
```

We can also remove the context:

```ruby
# todos_controller_spec.rb

# ...
  describe 'GET index' do
    # ...

    it_behaves_like 'require_sign_in' do
      let(:action) { get :index }
    end
  end
# ...
```

Now we can put those three lines of code wherever we need it, swapping `action` out for whatever we need! This is a much clearer way to test for cross-cutting concerns. We can use shared examples on the model level as well as the controller level.

### Feature Specs

So far we've written specs for models and controllers. We're obviously missing something. We haven't tested views and things like routes, helpers, and mailers.

In Rspec, there are specs for each of these components, so we could write specs for these components. However, we don't necessarily have to. We use a Feature Spec to test the integration of these components. We're operating on the browser level, mimicing the user's experience in browser. We can simiulate form fills, button clicks, and expect things to be rendered to the screen.

Feature specs are broken up into different features. Every feature in the app has a test for the user's experience of a feature. It is a way of vertical integration, penetrating the entire stack and exercising them all in integration.

Another kind of integration is a "horizontal" integration. With this, we test multiple requests and responses that go across multiple different controllers. This make sure the entire request works properly. This is called a 'Request spec' in rspec.

When we work with features, we use the Capybara gem which we'll cover in the next section. Then, we work on the level of user experience.

With request specs, we test a specific route, and expect the response to render a template or return something or redirect. We can follow redirects and expect the body to contain some content. When we're working with request specs, we aren't working on the browser level. There are no form filling or button clicks. We just fill in hashes.

In this course we aren't going to use request specs. If we want to test integration that goes beyond a single req/res, we're just going to write feature specs. This is because we want to make sure that a business process works.

### Capybara

Capybara is a way to simulate a real user's interaction with the application through the browser. There are two styles to use it with rspec/rails. First, we can use the rspec convention, with `describe`, `before`, `it`, etc.. Another style relates to feature specs. Instead of `describe`, we use `feature` and pass it a description of the feature name. `before` is replaced with `background`, and `it` is replaced with `scenario`, and `let` becomes `given`. This is just a different syntax that reads a little better w/r/t a feature spec (an acceptance test).

We can swap out drivers for Capybara. The default is `RackTest`, which is very fast. It doesn't really fire up a real browse--rather, it uses a headless driver. `RackTest` doesn't support javascript, so if JS is part of the user experience, we need to use `Selenium` or `Capybara-webkit`. `Selenium` fires up a real brower and we can watch how things are happening. This is pretty slow, so we can alternatively use `Capybara-webkit`. This allows us to do JS testing, which allows us to test JS code. It's headless and faster than `Selenium`, but slower than `RackTest`.

#### The DSL

##### Interaction

- `visit` to visit a route.
- `click_link`, `click_button`, `click_on` to simulate a user clicking a link
- `fill_in`, `choose`, `check`, `uncheck`, `attach_file`, `select`, etc. for form controls.
- `find_field`, `find_link`, `find_button`, `find`, `all` return specific elements wrapped in objects that we can call methods like `#value`, `#visible?`, and `#click` on.
- use `within`, pass it a CSS selector and a block if we want to interact with stuff within a certain part of the page
- We can use `page.execute_script` and pass it a string with some JS code to execute some JS.

##### Verification

- `page.has_selector?`, `page.has_xpath?`, `page.has_css?`, `page.has_content?` to check if the page has stuff.
  - These can be used with Rspec magic matchers, e.g. `page.should have_selector`, etc.

This is a start, and Capybara is a pretty big topic. We'll go through a very simple feature spec with Capybara.

### First Feature Spec

To install Capybara, we add `gem 'capybara'` to the `:test` group. We then add `require 'capybara/rails'` to our spec helper. We'll use the new Capybara style for our feature specs. We create a directory `spec/features`, and create our first feature spec file `user_signs_in.rb`.

```ruby
# user_signs_in.rb

require 'spec_helper'

feature 'user signs in' do
  scenario 'with existing username' do
    visit root_path
    fill_in 'Username', with: 'john'
    click_button 'Sign in'
    page.should have_content 'John Doe'
  end
end
```

Before this test can pass, we need to make sure we have a user `john` in the database.

```ruby
# user_signs_in.rb

# ...
feature 'user signs in' do
  background do
    User.create username: 'john', full_name: 'John Doe'
  end

  # ...
end
```

In the video, we run the test and find that it couldn't find the field 'Username'. This is because the label was not connected to the input field. Capybara is trying to find a label linked to a text input field. The label in the markup was a simple `<p>` tag. To fix this, we just needed to use a form helper to match the label to the field:

```haml
= form_tag sessions_path do
  = label_tag :username, 'Username'
  = text_field_tag :username
  /- ...
```

For `fill_in`, we can use the text in the label, or the `name` attribute on the element. Capybara will look for either a name, an ID, or text. Typically, the best practice is to use the label text since it's easier to read.

## Week 4

### Three styles of BDD with Rails

We have been learning the following:

1. Mockup -> View Template
2. Controller tests: specs for each controller action. This may or may not drop down to the model level.
3. Model tests: The conroller will drive the models. The controller tests will requre the models, routes, and db migrations to be set up. When the controller level is finished we pop up back to the controller level and make sure the view template correctly renders.
4. After several of these circles, we do some integration tests. This is a layer on top of the view templates and controllers--the outermost layer.

This process is called "Meet in the middle."

We also have "inside out": We start from the models/migrations/db. We do model tests, then pop up to the controller, making sure the controller is setting the right data. Then we look at the view level or integration tests.

Then there's the "outisde in" approach. We start from the integration level. This is a little different from our normal horizontal integration in which we test across multiple requests. Instead, this is more of a vertical integration test. These specs written in a very business-level description. We want to think of the business value that we're delivering. This drives insight into what we want to accomplish, challenging us to figure out the controllers, models, etc.

Another term for outside-in is "ABBD", for "Acceptance BDD". This process is used a lot by consultancies, since it's useful for working with clients. When you have a client, you have features specified very clearly to you in a business language. "I want a certain user to be able to do this," with the stakeholder writing the acceptance level. Then as a developer, we take one of these and write an integration spec, which drives out step by step the implementation details for us. If the integration spec passes, we know the integration is implemented from a business standpoint. We submit it to the client, and they accept it.

The advantage of ABBD is that it flows more naturally to the business requirements, letting that drive out the internals of our application. We don't need to think too much ahead. The hurdle of this is that this requres a very proficient grasp of Rails and testing in general. You have to write out the very high level integratoin spec, which requires several technologies to work well together. Some people use Cucumber for this, or Rspec feature specs. This takes some exercise to get familiar with.

If we do inside-out, we can just focus on a local model: start small, and then we expand. However, it isn't as clear what we are driving toward. It requires some assumptions that can be wrong when we work to the outside. This is use more in product-type shops where it's more it's about incrementally adding things to an existing product.

We use the meet-in-the-middle approach at LS. We have the advantage of the outside-in approach, writing integration specs when we have internals that work well together. This is not as rigorous as the outside-in approach, but we don't write as many integration specs, which run the slowest.

### HMT and HABTM

In rails, we typically want to look at `has_many :through` when we have a M:M mapping between our models. It is a pattern that the join table `belongs_to` both of its associated models, with the joined models having `has_many` of both the other joined model and the join table model.

Another way to model this is the `has_and_belongs_to_many` association. This requires our join table to have a specific naming convention, but does not requre an additional join model that we have to specifically name.

We typically want to use HMT, since while it's simpler to use HABTM, it's a good exercise to actually name the join table in the context of our application. This is good, because it will give our data and logic a place to live. We can include additional data to the data, as well. Also, it's more work to do this, but not _that_ much work.

### MyFlix social networking features

We can view a user's profile, in which we can see the videos they have collected in their queue, as well as their queue. We can also follow a user--clicking the `Follow` button bings us to the "people I follow" page, where we can see some brief data on that user, as well as a link to unfollow them.

### Self Referential Associations

Sometimes we want to do something called a "self join": a record referring to another record on the same table.

We can do this through a simple `has_many` association, but it's useful to name it something with meaning which isn't the name of the table. Using the RailsGuides example, an Employee can have many subordinates, which are also employees. The easiest way to accomplish this is to just use `has_many :employees`, with an `employee_id` column in the table. This doesn't really give us the meaning we are attempting to put across. So, we specify a name for each side of the relation:

```ruby
class Employee < ActiveRecord::Base
  has_many :subordinates, class_name: 'Employee', foreign_key: 'manager_id'
  belongs_to :manager, class_name: 'Employee'
end
```

This points the association back to the same model. Note that `belongs_to :manager` doesn't specify a foreign key, because there is a column in the database with the corresponding name, `manager_id`, per the Rails convention.

### Sending Email

Back to our Todos app. Whenever we add a Todo, we want to send out an email to the current signed-in user. This will take place in the `TodosController`. Rails provides `ActionMailer` to accomplish this.

We make a new mailer under `app/mailers/app_mailer.rb` and extend `ActionMailer`.

```ruby
class AppMailer < ActionMailer::Base

end
```

We define a method to send out notification emails:

```ruby
class AppMailer < ActionMailer::Base
  def notify_on_new_todo(user, todo)
    mail from: 'info@mytodoapp.com',
      to: user.email,
      subject: 'You created a new todo!'
  end
end
```

We provide all the header information to the `mail` method. For our body, `ActionMailer` uses something similar to view templates. So, we can store our todo to a `@todo` instance variable.

```ruby
# app_mailer.rb

class AppMailer < ActionMailer::Base
  def notify_on_new_todo(user, todo)
    @todo = todo

    mail from: 'info@mytodoapp.com',
      to: user.email,
      subject: 'You created a new todo!'
  end
end
```

To store the template, we create a template:

```
mkdir app/views/app_mailer
touch notify_on_new_todo.html.erb
```

```erb
<!doctype html>
<html>
<body>
  <p>
    You created a new todo!
  </p>
  <p>The todo is <%= @todo.name %></p>
</body>
</html>
```

Then, in our controller:

```ruby
# todos_controller.rb

# ...
  def create
    # ...
    if @todo.save_with_tags
      AppMailer.notify_on_new_todo(current_user, @todo).deliver
      # ...
    end
    # ...
  end
# ...
```

Note that we call `deliver` on the object returned by our call to `notify_on_new_todo`. This will send the email.


When we invoke that action via a request to the related route, the server's logs the email's information.

### Email Configurations

The ActionMailer configuration gives us a lot of control over how the email is delivered. This can be handled differently depending on the environment we are using.

Let's add some configuration settings for our `ActionMailer` action. Since this is our local environment, we don't want to send real emails. We aren't going to add smtp settings to development, but we can do so with our production enviroment. In `config/environments/production.rb`, we can copy/paste the boilerplate Gmail example from the RailsGuides entry on `ActionMailer`:

```ruby
# production.rb

# ...
  config.action_mailer.delivery_method = :smtp

  config.action_mailer.smtp_settings = {
    address:              'smtp.gmail.com',
    port:                 587,
    domain:               'example.com',
    user_name:            '<username>',
    password:             '<password>',
    authentication:       'plain',
    enable_starttls_auto: true  }
# ...
```

We can change all of this based on our gmail settings. When we deploy this to production, it will send out emails for real.

In the development environment, we can also set the delivery method. If nothing is set, it is all dumped in the server logs. However, there is a gem called `letter_opener` (which we add to the gemfile under `development`) which we can set the delivery method as in `development`:

```ruby
# development.rb

# ...
  config.action_mailer.delivery_method = :letter_opener
# ...
```

Make sure we restart the server when we change the gemfile. This gem will present the email in a browser tab. This is much nicer to look at than to sift through the server logs.

### Handling Sensitive Account Info

There are a lot of different ways to keep passwords and API keys from being hardcoded into our codebase. If we leave our Gmail username and password in plain text in our source code, anybody who has access to the code (on Github, etc) will have access to our gmail credentials.

We get around this by storing them as environment variables on our server. Fewer people will have access to this information in this case.

With Heroku, this is as easy as running the following command:

```
heroku config:add ENV_VAR_KEY=env_var_val OTHER_ENV_VAR_KEY=other_env_var_val
```

The keys are what we want them to be, but it's useful to name them something useful. For example, with our Gmail SMTP config, we can use `ENV['gmail_username']` and `ENV['gmail_password']`.

Other ways to do this is detailed on `railsapps.github.com/rails-env/environment-variables.html`. We can set UNIX environment variables, use the Figaro gem, or use the `local_env.yml` file. We want to make sure none of the files associated with these methods are checked in to our source control, which means in our case adding them to `.gitignore`.

### Testing Email Sending

In our `TodosController`, we have a line to send an email in the `create` action. In our test, we want to test the sending of emails under this action. We create a context in our spec for that action called 'email sending'. Here we outline those specs.

```ruby
# todos_controller_spec.rb

# ...
  describe 'POST create' do
    # ...
    context 'email sending' do
      it 'sends out the email'

      it 'sends to the right recipient'

      it 'has the right content'
    end
    # ...
  end
# ...
```

Now we expand:

```ruby
# todos_controller_spec.rb

# ...
  describe 'POST create' do
    # ...
    context 'email sending' do
      it 'sends out the email' do
        post :create, todo: { name: 'shop AT the apple store' }
        ActionMailer::Base.deliveries.should_not be_empty
      end

      it 'sends to the right recipient' do
        post :create, todo: { name: 'shop AT the apple store' }
        # We call `last` rather than `first` since we're just adding elements
        # to an array.
        message = ActionMailer::Base.deliveries.last

        # We stuff it in an array because we can put many recipients into
        # a mailer's header. Consequently, `#to` gets returned as an array.
        message.to.should == [alice.email]
      end

      it 'has the right content' do
        post :create, todo: { name: 'shop AT the apple store' }
        message = ActionMailer::Base.deliveries.last
        message.body.should include('shop AT the apple store')
      end
    end
    # ...
  end
# ...
```

These tests are mostely not for TDD, but are here to check for regressions.

### Setting Random Tokens

How to set random tokens to identify resources

This solves a similar problem that a slug solves. We don't ever really want to give out information regarding the database structure. Instead, we can use a random toaken to idenfity our resources. Lots of websites already do this, like Youtube and Trello.

We'll do this for the `Todo` resources in our Todo app. First we create a migration:

```
rails g migration add_token_to_todos
```

Then we write the mgiration.

```ruby
# The generated migration .rb

# ...
  add_column :todos, :token, :string
# ...
  add_index :todos, :token
# ...
```

Then we migrate.

```
rake db:migrate
```

Make sure we restart the server after we do this.

In our `Todo` model, we need to add an instance method called `#to_param`. `#to_param` is called whenever we pass the model to a named path (e.g. `todo_path(todo)`). By default, it returns the `id` of the model. We can overwrite it to return the token instead.

```ruby
# todo.rb

# ...
  def to_param
    token
  end
# ...
```

Now we need to automatically set the token when we generate the model. We can add a `before_create` to the model to accomplish this. This only needs to be done once, so we don't use, e.g., `before_save`.

```ruby
# todo.rb

# ...
  before_create :generate_token
# ...
  def generate_token

  end
# ...
```

We can use the Ruby module `SecureRandom` to do this. One method we can call is `base64` to make sure the return is textual. A better method, though, is `urlsafe_base64`, which allows us to use the token as part of the URL.

```ruby
# todo.rb

# ...
  def generate_token
    self.token = SecureRandom.urlsafe_base64
  end
# ...
```

Since it is run before the model is created, we don't need to call `save`.

When we first try to run this, we get a RoutingError, which means Rails has failed to generate the path. The reason is that our existing data doesn't have any data in the `token` column. We'll need to migrate this data when we run our migration. Let's modify our recently-created migration to handle this.

```ruby
# that_same_generaged_migration_from_above.rb

# ...
  add_column # etc...
  Todo.all.each do |todo|
    todo.token = SecureRandom.urlsafe_base64
    todo.save
  end
# ...
```

This is OK if we don't have a lot of data in the database. If that were the case we'd instead want to use SQL to do the same thing.

```
rake db:rollback db:migrate
```

Now we have retrofitted the exisitng data to include our tokens.

Now, when we try to use any action requiring a spcific record in `todos`, we'll get an error. This is because our logic for setting the instance variable is still looking for an `id`. We need to modify this code to make it look for the token instead.

```ruby
# todos_controller.rb

# ...
  @todo = Todo.find_by token: params[:id]
# ...
```

### Adding Password Resets to MyFlix

I'm putting the an assignment-related portion to these notes, in order to just describe the workflow of resetting passwords. The password reset process goes as follows:

1. There is a link to reset the password on the login page. The user clicks that link.
2. The user is brought to the "forgot password" form, in which they enter their email address and submit the form.
3. The user is brought to the "confirm password reset" page, letting them know to check their inbox.
4. The system sends an email to that email address with a link pointing to something like `password_reset/tokentokentokentoken`. The user clicks on that link.
5. The user is brought the "reset password" form, and they enter a new password and submits the form.
6.

The token should only be valid before the user resets their password. If they reset their password using the token, we need to expire that token.

## Week 5

### Concerns

So in our Todo app, we've specified that we want to generate a token every time we want to create a todo. If we have multiple models where we want this to happen, we wouldn't want to repeat the implementation of token generation. In a software application, every piece of knowledge should have one authorative definition. So we want to extract the process of generating tokens and take it to one place, so we can refer to that process with every model to which this applies.

First we make a module called `tokenable`, placing `tokenable.rb` in the `lib` directory.

```ruby
# lib/tokenable.rb

module Tokenable
  extend ActiveSupport::Concern
end
```

Extending ActiveSupport::Concern is a way for Rails to orgainize cross-cutting concerns in our applications.

We add an `included` call in the module:

```ruby
# lib/tokenable.rb

module Tokenable
  extend ActiveSupport::Concern

  included do
    before_create :generate_token
  end
end
```

This calls the includer's `before_create` class method, passing in the usual args. We can then define `generate_token` in our module:

```ruby
# lib/tokenable.rb

module Tokenable
  extend ActiveSupport::Concern

  included do
    before_create :generate_token
  end

  private

  def generate_token
    self.token = SecureRandom.urlsafe_base64
  end
end
```

In our model, we can just call `include`:

```ruby
# todo.rb

class Todo < ActiveRecord::Base
  include Tokenable

  # ...
end
```

That piece of knowledge is abstracted as a concern, and now we can share that logic with other AR models.

In order for this to work, we need to `require` the file, since Rails doesn't load the `lib` directory automatically:

```ruby
# todo.rb

require_relative '../../lib/tokenable'

class Todo < ActiveRecord::Base
  # ...
end
```

There are some more options for including the `tokenable.rb` file. We can simply place it in the `app/models` directory, since that directory is automatically loaded. The other option is to open the `application.rb` config file, adding the following to `Application` class:

```ruby
# config/application.rb

# ...
module TodoApp
  class Application < Rails::Application
    # ...
    config.autoload_paths << "#{Rails.root}/lib"
  end
end
```

`autoload_paths` is an array, so we need to push that path onto it using `#<<` or `#push`. Now we don't have to use `require_relative` in our model file.

### Email Service Providers

Gmail imposes lots of restrictions on sending emails, so email is something we should absolutely offload to a transactional email provider. This is because having emails arrive to a user's inbox rather than spam is such a critical part of many applications.

LaunchSchool recommends Mailgun and Postmark, since we want to opt for deliverability, which is where these two services really shine. We'll use Mailgun here at LaunchSchool.

### Background Jobs

In our Create action, we send out an email synchronously. Typically, email sending isn't very fast since it typically relies on a 3rd party service. Therefore, before the email sends the controller won't generate a response. The creation of a new todo will feel a little slow. Furthermore, this will block all subsequent requests, queuing them up.

The email sending doesn't have to happen synchronously with the req/res, since it's not time-sensitive. We can offload the email sending to a Background Process.

Here's how a typical Rails application works re: processes. We have one or multiple foreground processes (also known as web processes), which handle incoming requests. We also have one or many background processes (aka worker processes), which handle background jobs.

Web processes each have an instance of rails, which handle incoming requests. If there is a job that doesn't need to be included in the response, they can offload the job to worker processes. Worker processes act like a queue, processed one by one.

There are several frameworks that can help with this. First, we have `Resque`, which came out as a response to other background job solutions.

Sidekiq is a newer, high performance solution. Here we'll take a look at it in detail. We're going to make the email sending in our todo app asynchronous:

We install the gem:

```ruby
gem 'sidekiq'
```

```
bundle install
```

Sidekiq already comes with an `ActiveMailer` extension, so we just need to call `delay` before we call the mailer's method in question:

```ruby
# todos_controller.rb

class TodosController < ApplicationController
  # ...

  def create
    # ...
    AppMailer.delay.notify_on_new_todo(current_user, @todo)
    # ...
  end

  # ...
end
```

Here we just call `delay` and remote `deliver` from the end. If we do this, our tests related to the action will fail, because the test is looking for deliveries.

To fix this, we need to look at the actual email, so we need to require `sidekiq/testing` to test our workers inline, and add some lines of config. So we require the module in `spec_helper.rb`

```ruby
# spec_helper.rb

# ...
require 'capybara/rails'
require 'sidekiq/testing'

# ...

Sidekiq::Testing.inline!
```

Now our tests will pass.

### Procfile and Foreman

When working with background processes locally, we need to run both the rails server as well as the sidekiq worker process. When we want to do this on the production server, we use a Procfile. The `Procfile` allows us to define all processes that will run on our server.

Web services will go like this:

```
web: bundle exec rails server -p $port
```

This is the default rails server. For worker processes Heroku uses the following as a default:

```
worker: bundle exec rake jobs:work
```

We obviously have to change this for sidekiq. The convention for a `Procfile` goes like this:

```
<Process type>: <Command to launch process>
```

When we push to Heroku, these processes will automatically start. We can go one step further by developing locally with Foreman. We can just use `foreman start` if we have a `Procfile` defined.

With Heroku, an increase in the number of processes running may increase the price of the deployment.

### Unicorn

Unicorn is a popular alternative to the default Rails server. By default, Rails will only handle one request at a time. Unicorn makes it very easy to start multiple instances of the Rails server.

We install Unicorn the normal way, and use the file `config/unicorn.rb` to configure the server.

In the procfile, we need to use a different command:

```
web: bundle exec unicorn -p $PORT -c ./config/unicorn.rb
```

### Continuous Integration

(Copied from the lesson itself)

By this time, you should be aware and comfortable with automated tests and the benefits. Up until now, we're relying on developer's discipline to always ensure tests pass locally before they deploy features. This is probably ok in small and high-trust teams, but in larger teams you typically see Continuous Integration (CI) introduced as part of the development process.

In his article on Continuous Integration, Martin Fowler defines continuous integration as:

> Continuous Integration is a software development practice where members of a team integrate their work frequently, usually each person integrates at least daily - leading to multiple integrations per day.

The problem that CI solves is to force integration on a regular basis. For example, you may have just finished a feature on my_feature branch and have had all the tests pass locally. You open a PR and your fellow coworkers are happy with the way the code looks. Github shows the PR can be automatically merged without conflict - so you do that, merge it back to the master branch and ready to deploy the feature!

The danger of this workflow is that while you are working on your feature, someone else could have just finished their feature and pushed to the master branch to Github, and your new feature never integrated with this new piece of code.

A Continuous Integration solution would watch your repository and force integration (running the entire test suite on the CI server) whenever necessary.

Once we have our CI server set up to monitor our repository, it'll pull the latest code and run the entire test suite every time we push code to any branch on Github (when you work with a CI server, you want to make sure that every push should always make the tests pass) and notify you on the results. The CI server also runs every time we merge our pull request to the master branch to ensure integration.

A CI server helps us catch integration errors before they reach production, and the earlier we catch errors, the less costly we can fix them.

A CI solution doesn't eliminate the necessity to run your tests locally - your will lose friends quickly if you constantly push code that "breaks the build" and have everyone notified. However, in cases of long running test suites, it is acceptable to run only the tests pertaining to the new feature built, and have the CI server to run the entire test suite to catch any potential regression. Most modern CI services allow you to run the tests in parallel (with a paid plan, typically) so it could run your tests several times faster.

### Continuous Delivery

Continuous Delivery (CD) or sometimes known as Continuous Deployment goes one step beyond CI to automatically deploy features when the new code passes the continuous integration phase. Continuous Delivery encourages small and incremental software updates over big and infrequent releases to shorten the feedback loop and fix bugs earlier.

Let's look at an example development workflow that has both CI and CD enabled, based on the Github Flow process:

- we pull the latest code from Github
- we create a new feature branch and develop a new feature
- after we finish the feature, we push it to a branch with the same name on Github
- we create a PR from this branch to the staging branch.
- we wait for the the CI server to ensure all tests pass.
- we allow the CI server to automatically deploy the code from the staging branch to our staging server
- we perform sanity tests on our staging server
- we create a PR from the staging branch to the master branch on Github
- this will trigger another round of integration and if it passes, the CI server will automatically deploy the code to the production server.

If adopted by everyone in the development team, this process will make sure our master branch is always in sync with the production server and new features are also always build on top of what's on the production server.

### Setting Up a CD Thing with Circle CI

For this assignment, you are going to enable Continuous Delivery with Circle CI.

Here are the steps:

- create a `staging` branch locally if you don't have it yet.
- in Circle CI, follow `Project settings` for your project and select `Heroku Deployment` under Continuous Deployment
- configure the Heroku API key. (Get it from your Heroku account page)
- associate Heroku SSH key with your Circle account so Circle can have the authority to deploy to Heroku on your behalf.
- create a `circle.yml` file in your projects root directory and make sure your adjust `production_app_name` & `staging_app_name` to your own app name.

Here is an example of a circle.yml file that you can use:

```yaml
machine:
  ruby:
    version: 2.1.5
deployment:
  production:
    branch: master
    commands:
      - heroku maintenance:on --app production_app_name
      - heroku pg:backups capture --app production_app_name
      - git push git@heroku.com:production_app_name.git $CIRCLE_SHA1:refs/heads/master
      - heroku run rake db:migrate --app production_app_name
      - heroku maintenance:off --app production_app_name
  staging:
    branch: staging
    commands:
      - heroku maintenance:on --app staging_app_name
      - git push git@heroku.com:staging_app_name.git $CIRCLE_SHA1:refs/heads/master
      - heroku run rake db:migrate --app staging_app_name
      - heroku maintenance:off --app staging_app_name
```

Note: you should change the Ruby version to the version you're using for this project.

This code should be pretty self explanatory - this allows Circle to monitor your staging branch and deploy to the staging server, and monitor your master branch to deploy to the production server. It'll run migrations for you and for the production server, it also automatically backs up the database before a deploy.

## Week 6

### Separating Actors

What if we wanted to be able to have admins for our todo application? It is very typical for an application to have several types of users within our application, typically called 'actors,' because they interact with the application in different ways. How we can add admin functions?

One way to do this is conditionals within our controllers:

```ruby
if current_user.admin?
  @todos = Todo.all
else
  @todos = current_user.todos
end
```

This is very cumbersome if we have this type of conditional check throughout our application--even sometimes in the views.

Another way is to have a separate action--say `admin_index` vs `index`. The issue with is that if we want our admins to be able to do more things, or even have more actors, then it becomes very cumbersome, and the controller will blow up. We also have to manage access control for different actors, which requires additional `before_action`s.

Instead, we can add namespaces to our `routes.rb` file.

```ruby
# routes.rb

TodoApp::Application.routes.draw do
  # ...
  namespace :admin do
    resources :todos, only: [:index, :destroy]
  end
  # ...
end
```

This allows us to specify the resources to which each actor has access, as well as how we want to present data to the actor, and how the actor can interact with the resource. So, if we have multiple actors, we can have multiple namespaces for these.

So how do these namespaces work? If we do a `rake routes`, we notice that we've added an `admin_todos` named route under `/admin/todos`, which points to the `admin/todos` controller, `#index` action.

So, all the admin urls are going to start with `admin`. To set up the controller, we're going to put the contoller under `app/controllers/admin` subdir.

```
mkdir app/controllers/admin
touch app/controllers/admin/todos_controller.rb
```

```ruby
# app/controllers/admin/todos_controller.rb

class Admin::TodosController < ApplicationController

end
```

Notice how this is namespaced under the `Admin` module. This is how Rails differentiates between the two controllers.

```ruby
# app/controllers/admin/todos_controller.rb

class Admin::TodosController < ApplicationController
  def index
    @todos = Todo.all
  end
end
```

We need to create an admin, so we need to add an `admin` boolean column to our `users` table. Once we do that, we can just add a user as an admin through the console for now.

```ruby
User.create username: 'admin', full_name: 'Admin User', email: 'admin@example.com', admin: true
```

We need to wire the app to point the user to the `admin_todos_path`:

```ruby
# sessions_controller.rb

class SessionsController < ApplicationController
  def create
    # ...
    redirect_to current_user.admin? ? admin_todos_path : todos_path
  end
end
```

We need to create a view template for this. The convention for the view template is `admin/todos` under the `views` path. We add a `index.html.haml` file in that dir. We can pretty much copy/paste from the other view template, perhaps adding a display for the user to which each todo belongs.

### Securing Access

With our current setup, any user can go to the `admin/todos` path to view any of the admin stuff. There are a few ways to lock this down.

First we can create a `before_action` for each of our controllers under `admin` so that only admin can see the admin areas. This is fine for a simple app, but we may have lots of controllers namespaced with `Admin`, and it's very easy to miss this. This is bad, since admin stuff is typically something we want to secure.

We can instead create a controller called `AdminsController` that our admin-namespaced controllers can inherit. So first we subclass that controller:

```ruby
# admin/todos_controller.rb

class Admin::TodosController < AdminsController
 # ...
end
```

We can then create an `admins_controller.rb` file under `controllers`:

```ruby
# admins_controller

class AdminsController < ApplicationController
  before_filter :ensure_admin

  def ensure_admin # this is coded wrong but I'm too lazy to come back to it.
    flash[:danger] = 'You do not have access to that area.'
    redirect_to root_path unless current_user.admin?
  end
end
```

We can even go one step further and make all our controllers that require signed in users to have the `ensure_sign_in` `before_action` is called, by creating an `AuthenticatedController` that the relevant controllers (like `TodosController`) can subclass. Furthermore, since we want our admin stuff to require signed in users, we can subclass `AdminsController` with `AuthenticatedController`!

### Amazon S3

A cloud-based storage service! If your website serves a lot of content like images, etc, or if you allow users to upload content, this is a good solution for doing so.

You just need to create an account and a bucket. S3 has a web interface for storing S3 objects, but it's also useful to user Cyberduck or Transmit.

### File Uploading with Carrierwave

Carrierwave is the go-to library for file uploading in the Ruby and Rails community.

You add the gem and run bundler, and that will give us a generator that will allow us to generate uploaders. Carrierwave uses uploaders which will manage all the infrastructure concerns for file uploading, including file uploading, processing, as well as storage.

Once we generate an uploader, it will create a class that subclasses from `CarrierWave::Uploader::Base`. We can specifies the storage:

```ruby
class AvatarUploader < CarrierWave::Uploader::Base
  storage :file
end
```

If we've defined an uploader, we just need to create an instance and store it like so:

```ruby
uploader = AvatarUploader.new
uploarder.store! my_file # where my_file is an instance of a CW file

# retrieve a file
uploader.retrieve_from_store! 'my_file.png'
```

We get `store` for permanent storage, and `cache` for temp storage. We can use `cache` help with validation.

With ActiveRecord, we can mount our uploaders to columns in the database using `mount_uploader`:

```ruby
class User < ActiveRecord::Base
  mount_uploader :avatar, AvatarUploader
end
```

This allows us to call some methods to accomodate for file storage:

```ruby
u = User.new

# either...
u.avatar = params[:file]

# or...
u.avatar = File.open 'somewhere'

# then we can just:
u.save!

# this lets us call some cool methods
u.avatar.url #=> '/url/to/file.png'
u.avatar.current_path #=> '/path/to/file.png'
u.avatar.intentifier #=> 'file.png'
```

By default Carrierwave saves file uploads locally, but you can change the directory by defining a `store_dir` method and a `cache_dir` method, returning a string with a path to that directory.

Configuring mostly works similarly to that. You can add a whitelist for uploads to, say, a certain set of file extensions:

```ruby
class MyUploader < CarrierWave::Uploader::Base
  def extension_white_list
    %w(jpg jpeg gif png)
  end
end
```

We can also add multiple versions of the same file. For example, we can have image thumbnails. We just include `CarrierWave::RMagick` which is just a wrapper for the `ImageMagick` library. In order for this to work, we need to install ImageMagick both locally and on our server.

Once that's done, we just need to make calls to `process` and `version`:

```ruby
class MyUploader < CarrierWave::Uploader::Base
  include CarrierWave::RMagick

  process resize_to_fit: [800, 800]

  version :thumb do
    process resize_to_fill: [200, 200]
  end
end
```

The first `process` call processes every file that gets stored, while the second will create a thumbnail version of that file.

```ruby
uploader = AvatarUploader.new

uploader.store! my_file

uploader.url #=> '/url/to/my_file.png'
uploader.thumb.url #=> '/url/to/thumb_my_file.png'
```

We can provide a default url for our images by defining a `defualt_url` file.

When we test uploaders, we want to test them each separately. CarrierWave even comes with its own matchers for rspec.

#### Amazon S3

We just need to install the `fog` gem to use Carrierwave with S3. Once we do that we just need to configure in `lib/carrierwave/storage/fog.rb`. A full sample of this is listed in the docs.

### Full File Upload Workflow

#### 1. Create and Mount Uploaders

```ruby
# video.rb

class Video < ActiveRecord::Base
  # ...
  mount_uploader :large_cover, LargeCoverUploader
  mount_uploader :small_cover, SmallCoverUploader
  # ...
end
```

```ruby
# app/uploaders/large_cover_uploader.rb

class LargeCoverUploader < CarrierWave::Uploader::Base

end
```

```ruby
# app/uploaders/small_cover_uploader.rb

class SmallCoverUploader < CarrierWave::Uploader::Base

end
```

#### 2. Make sure sizes are right

```ruby
# app/uploaders/large_cover_uploader.rb

class LargeCoverUploader < CarrierWave::Uploader::Base
  include CarrierWave::MiniMagick

  process resize_to_fill: [665, 375]
end
```

```ruby
# app/uploaders/small_cover_uploader.rb

class SmallCoverUploader < CarrierWave::Uploader::Base
  include CarrierWave::MiniMagick

  process resize_to_fill: [166, 236]
end
```

### 3. Set up files to upload to S3

```ruby
# config/initializers/carrier_wave.rb

CarrierWave.configure do |config|
  if Rails.env.staging? || Rails.env.production?
    config.storage    = :aws
    config.aws_bucket = ENV.fetch('S3_BUCKET_NAME')
    config.aws_acl    = 'public-read'

    # The maximum period for authenticated_urls is only 7 days.
    config.aws_authenticated_url_expiration = 60 * 60 * 24 * 7

    config.aws_credentials = {
      access_key_id:     ENV.fetch('AWS_ACCESS_KEY_ID'),
      secret_access_key: ENV.fetch('AWS_SECRET_ACCESS_KEY'),
      region:            ENV.fetch('AWS_REGION') # Required
    }
  else
    config.storage = :file
    config.enable_processing = Rails.env.development?
  end
end
```

### Collecting Credit Card Payments

There are a lot of moving parts w/r/t accepting payments--merchant accounts, payment gateways, and automation. All this is compounded by the requirement for PCI compliance.

If you're building a small app, this all becomes too prohibitive. Stripe is a major pioneer in this space, making it so you don't need a merchant account or payment gateway. Furthermore, the customer's information doesn't ever flow through your application server. Instead, it produces a token which you store on a server.

#### First Charge with Stripe

We want to stay in test mode for this purpose.

First we include strip in the Gemfile:

```ruby
# Gemfile

# ...
gem 'stripe'
# ...
```

First we get our API keys and add them to `figaro`.

First let's test things out in the console:

```ruby
# rails c

Stripe.api_key = ENV['STRIPE_API_KEY']

Stripe::Charge.create(
  amount: 400,
  currency: 'usd',
  card: {
    number: '4242424242424242',
    exp_month: '3',
    exp_year: '2014',
    cvc: 314
  },
  description: 'Charge for test@example.com'
)
```

Remember that the amount is in cents!

#### Accepting Payments with Checkout Widget

So this is how we do a simple Stripe charge. We'll focus on how to collect card information from our customers. The easiest way to do this is to add a widget to our view, which will pop up a modal for the customer to enter our card information.

With our todo app, we can create a resource for payments:

```ruby
# routes.rb

# ...
resources :payments, only: [:new]
# ...
```

Now we can create a view for that resource:

```haml
-# app/views/payments/new.html.haml

%p Thank you for your support.
= form_tag payments_path do
```

We can simply include a JS tag from the Stripe docs from `payments -> checkout`:

```haml
-# app/views/payments/new.html.haml

%p Thank you for your support.
= form_tag payments_path do
  %script(src="https://checkout.stripe.com/checkout.js" class="stripe-button"
      data-key="pk_test_Oktlq7NDL2eOfbcgF09Sfpm3" data-amount="999" data-name="Support Todo App!" data-description="Thank you for your donation!" data-image="https://stripe.com/img/documentation/checkout/marketplace.png" data-locale="auto")
```

The docs will automatically include your public key in the sample snippet.

This will generate a button on the page letting the user pay with a card. Now, this goes to Stripe and comes back. However, this will also send a request through our app, so we need to add a `:create` action to the resource in our `routes.rb` file.

We also need to add a `PaymentsController`. If we through a `pry` in the `create` action, we can see the params, where we can see a `stripeToken` key. This is a token representation of our credit card, which is stored at Stripe.

In our `create` action, we'll start by copy/pasting some code from Stripe's docs, under 'Creating Charges.'

```ruby
class PaymentsController < ApplicationController
  Stripe.api_key = ENV['STRIPE_API_KEY']

  # Get the credit card details submitted by the form
  token = params[:stripeToken]

  # Create a charge: this will charge the user's card
  begin
    charge = Stripe::Charge.create(
      :amount => 999, # Amount in cents
      :currency => "usd",
      :source => token
    )

    flash[:success] = 'Thank you for your support!'
    redirect_to new_payment_path
  rescue Stripe::CardError => e
    flash[:danger] = e.message
    redirect_to new_payment_path
  end
end
```

This code takes the token from the `params` hash and starts a `begin/rescue` block, hitting the Stripe server and checking to see if the charge runs. If the card is declined, we can return a message.

So here's how the Stripe workflow works with the Checkout widget:

1. User fills and submits the form with their credit card information.
2. The form sends the card information to Stripe, which then returns a unique token corresponding to the card.
3. A `post` request is submitted to the application server with the token in the params.
4. The application server calls `Stripe::Charge.create`, sending the token to the server with the amount to charge.
5. Two things can happen:
  a. If the card is declined, a `Stripe::CardError` is thrown, which we can catch.
  b. If the card is charged, we can continue with the script.

Notice how none of this attaches any data to our user's record.

#### Accepting Payments with Custom Forms

We sometimes want to include the payment information in our interface, rather than on a widget.

In our Todo app, we have a `form_tag`, with fields for the card number, security code, etc. `stripe.js` gives us the ability to do this.

First we need to include the script. To do this, we just add a `yeld :head` line in the head of our main layout:

```haml
-# app/views/layouts/application.html.haml

-# ...
  %head
    -# ...
    = yield :head
    -# ...
-# ...
```

Then, we can add a `content_for` line in our view corresponding to accepting payments:

```haml
-# app/views/payments/new.html.haml

= content_for :head do
  %script(src='https://js.stripe.com/v1/')
  :javascript
    Stripe.setPublishableKey('the stripe publishable key!');
  = javascript_include_tag 'payments'

-# the rest of the payment form...
```

We now need to call `createToken`. We create a JS file called `payments.js` in `app/assets/javascripts`. In the video we use a boilerplate JS file, which does the following:

1. Hijack the submit of the form with the id `#payment-form`.
2. In the event handler's callback function:
  a. Disable the `submit` tag (by calling `prop`, passing in `'disabled', true`) to make sure the button is only clicked once.
  b. Call `Stripe.createToken`, passing in the credit card info as a JS object, then the variable `stripeResponseHandler` (defined below).
    i. Note that we don't have any `name` attributes attached to our credit card info field--furthermore, we set `name` to `nil` in our auto-generated fields for our expiration month and year. Therefore, we need to select the elements by the `class` attribute. This is all so that none of the credit card information gets posted to our server when the form gets submitted. This is for PCI compliance--the credit card information never flows through our own server.
    ii. Note also that `createToken` is an async call, and the callback function, `stripeResponseHandler`, is passed as the second parameter.
  c. Return false.
3. We define `stripeResponseHandler`, which takes variables named `status` and `response`. This will be the callback for our `createToken` call.
  a. If the `response` has a truthy `error` property, we add some information to the page regarding that error, also re-enabling the submit button.
  b. Else, we set `token` to the `id` property of `response`, adding a hidden `input` element, with the name `stripeToken` and the value of `token`. Then we submit the form (`$(form).get(0).submit();`).

Here's `payments.js` in full:

```javascript
$(function() {
  $('#payment-form').submit(function(e) {
    e.preventDefault();
    var $form = $(this);
    $form.find('.payment-submit').prop('disabled', true);
    Stripe.createToken({
      number: $('.card-number').val(),
      cvc: $('.card-cvc').val(),
      exp_month: $('.card-expiry-month').val(),
      exp_year: $('.card-expiry-year').val()
    }, stripeResponseHandler);
    return false;
  });

  var stripeResponseHandler = function(status, response) {
    var $form = $('#payment-form');

    if (response.error) {
      $form.find('.payment-errors').text(response.error.message);
      $form.find('.payment-submit').prop('disabled', false);
    }
    else {
      var token = response.id;
      $form.append($('<input type="hidden" name="stripeToken" />').val(token));
      $form.get(0).submit();
    }
  }
});
```

## Week 7

### Wrapping APIs

We're going to make a local wrapper for the Stripe APIs. Wrappers exist because there may be many places in our application in which we need to process payments. Instead of having them around our application, we can centralize them in once place. We just need to interact with the Stripe wrapper, and the wrapper itself will interact with the Stripe service.

First, we create a model, under `stripe_wrapper.rb`. This is a plain Ruby object that we don't need to inherit from `AR`.

```ruby
# app/models/stripe_wrapper.rb

module StripeWrapper
  class Charge
    def self.create(options = {})
      Stripe::Charge.create(
        amount: options[:amount],
        currency: 'usd', # We'll just hardcode this
        card: options[:card]
      )
    end
  end
end
```

Now, when we go to create the charge, we use the `StripeWrapper` module instead of `Stripe` itself, and we don't need to pass in `currency`. This is the simplest wrapper we can have.

We can also move the API key to the module, but let's put it in a different place, since we might need other classes to access the API key.

```ruby
# stripe_wrapper.rb

module StripeWrapper
  # ...

  def self.set_api_key
    Stripe.api_key = ENV['STRIPE_API_KEY']
  end
end
```

Then we just need to call `StripeWrapper.set_api_key` whenever we need to use it!

```ruby
# stripe_wrapper.rb

module StripeWrapper
  class Charge
    def self.create(options = {})
      StripeWrapper.set_api_key
      # ...
    end

    # ...
  end

  # ...
end
```

The other part of wrapping 3rd party APIs is to handle the response part (we've just done the request part). When we create a payment in our controller, we still have a reference to `Stripe`, so ideally we don't want to have any of those outside of the wrapper.

We need to move a few things around, so let's start with the controller. First we want to remove the `begin/rescue` block, instead using an `if` block:

```ruby
class PaymentsController < ApplicationController
  def create
    token = params[:stripeToken]

    charge = StripeWrapper::Charge.create amount: 3000, card: token

    if charge.successful? # Check if the charge was successful
      flash[:success] = "blah"
      redirect_to new_payment_path
    else
      flash[:danger] = charge.error_message # Error messages have been stored
      redirect_to new_payment_path
    end
  end
end
```

So this implies we need to do a few things to our `Charge` class. First, `create` needs to return an object, and that object needs to store some stuff like `#successful?` and `#error_message`.

```ruby
# stripe_wrapper.rb

module StripeWrapper
  class Charge
    attr_reader :response, :status

    def initialize(response, status)
      @response = response
      @status = status
    end

    def self.create(options = {})
      StripeWrapper.set_api_key

      begin
        response = Stripe::Charge.create(
          amount: options[:amount],
          currency: 'usd',
          card: options[:card]
        )

        new(response, :success)
      rescue Stripe::CardError => e
        new(e, :error)
      end
    end

    def successful?
      status == :success
    end

    def error_message
      response.message
    end

    # ...
  end

  # ...
end
```

A few things going on here. First we're returning a new `StripeWrapper::Charge` object in either case, passing in a `response` and a symbol indicating whether the charge was successful or not (using a convenience method `#successful?` to check this). The `response` will be the return value of the `Stripe::Charge.create` call, or the `Stripe::CardError` object for the success or failure case, respectively. If the card is declined or otherwise results in an error, we can access the `message` attribute in the response object.

### Fully Integrated API Tests

When we developed our wrapper, we didn't really follow the TDD process. This is because we didn't write this from the ground up as a wrapper. We definitely wrap tests around this wrapper, since it's such an important piece of code. We create a model test:

```ruby
# spec/models/stripe_wrapper_spec.rb

require 'spec_helper'

describe StripeWrapper::Charge do

end
```

3rd party API tests can be written in two ways. First, we can fake the response generated by the 3rd party API, or fully hit the 3rd party server. With this test, we're going to fully integrate Stripe. So let's create two contexts: one with a valid card and the other with an invalid card.

```ruby
# spec/models/stripe_wrapper_spec.rb

require 'spec_helper'

describe StripeWrapper::Charge do
  context 'with valid card'

  context 'with invalid card'
end
```

With a valid card, we only need one touch point, and that's `#successful?`. With an invalid card, we have two: `#successful` (returning `false`) and `#error_message`.

```ruby
# spec/models/stripe_wrapper_spec.rb

require 'spec_helper'

describe StripeWrapper::Charge do
  context 'with valid card' do
    it 'charges the card successfully'
  end

  context 'with invalid card' do
    it 'does not charge the card successfully'

    it 'contains an error message'
  end
end
```

So let's write this out:

```ruby
# spec/models/stripe_wrapper_spec.rb

require 'spec_helper'

describe StripeWrapper::Charge do
  before do
    StripeWrapper.set_api_key # Make the Stripe API key available to `Stripe`
  end

  let(:token) do
    Stripe::Token.create(
      card: {
        number: card_number,
        exp_month: 3,
        exp_year: 2019,
        cvc: 123
      }
    ).id # Pull the token ID property from the return value of `#create`
  end

  context 'with valid card' do
    let(:card_number) { '4242424242424242' }

    it 'charges the card successfully' do
      res = StripeWrapper::Charge.create(
        amount: 3000,
        card: token
      )

      expect(res).to be_successful
    end
  end

  context 'with invalid card' do
    let(:card_number) { '4000000000000002' }
    let(:res) do
      StripeWrapper::Charge.create(
        amount: 3000,
        card: token
      )
    end

    it 'does not charge the card successfully' do
      expect(res).to_not be_successful
    end

    it 'contains an error message' do
      expect(res.error_message).to be_present
    end
  end
end
```

### Isolated API Tests

Our current Stripe tests take quite a long time to finish. This is because we're hitting the Stripe servers twice for each test. As our application grows, this amount of time will also grow. To get around this, we use `webmock` to stub the HTTP request at the HTTP client lib level.

`webmock` operates at a very low level, so we need to wire up all the headers and bodies manually. We'd want to work at a higher level, and with something that integrates with Rspec. For that we have `vcr`.

`vcr` works by going through the wire the first time a test runs, recording the request and response data. The next time the test runs, it will play back the recording.

For our todo app, we just add the gems to our gemfile:

```ruby
#test group
gem 'vcr'
gem 'webmock'
```

Then we setup `vcr` to work with rspec, adding the block of config code from the "getting started" section of the relishapp/vcr docs. Now, for every `it` block with which we would like to use `vcr`, we just need to add an additional parameter to the `it` call:

```ruby
# stripe_wrapper_spec.rb

# ...
  it 'charges the card successfully', :vcr do
    # ...
  end
# ...
```

When we run the spec the first time, it will send requests through the wire, taking a few seconds. Any subsequent runs of the spec will be much quicker. VCR adds a `cassettes` directory under `spec`, saving datafiles for the requests and responses.

VCR supports a lot of configuration options, allowing us to record different interactions. We can also configure record modes--by default it uses `:Once` which records a single interaction. If we set it to `:All`, it will never use playbacks, always going through the wire. It is a good idea to flip the record mode to this, just to make sure the API hasn't changed and that our cassettes are still valid.

### Test Doubles and Method Stubs

Now we should write tests for our Payments controller.

```ruby
# payments_controller_spec.rb

require 'spec_helper'

describe PaymentsController do
  describe 'POST create' do
    context 'with a successful charge' do
      it 'sets the flash success message'

      it 'redirects to the new payment path'
    end

    context 'with an error charge' do
      it 'sets the flash'

      it 'redirects to new payment path'
    end
  end
end
```

So here, we don't really want to test the output of the Stripe wrapper, which we have already written test for. Instead, we want to stub `StripeWrapper::Charge.create` to do something else.

First we can create a test double:

```ruby
# payments_controller_spec.rb

# ...
  context 'with a successful charge' do
    it 'sets the flash' do
      charge = double('charge') # Create a fake object standing in for
                                # the `Charge` wrapper object. We pass
                                # `'charge'` in, which is just the name
                                # of the variable.

      charge.stub(:successful?).and_return(true)
                                # Stubs the `#successful?` method.

      StripeWrapper::Charge.stub(:create).and_return(charge)
                                # Stubs the `#create` method on the wrapper.

      post :create, token: '123'

      expect(flash[:success]).to be_present
    end
  end
# ...
```

So there's a lot of stuff going on here. First, we create a double, which stands in for the actual return value of our wrapper. We're going to create a stub on that test double, which mimicks a method call which will always return true. So if we call `charge.successful?`, we'll always get `true`.

Next, we are going to stub `create` on `StripeWrapper::Charge` so that it'll always return our double when we call it. This avoids the execution of the actual method. Because of this, we don't have to generate an actual Stripe token. The reason for this is that we no longer need to call the actual methods, triggering a request to Stripe's servers.

We can stick that logic into the `before` block, using it in our other test in the context. We can create similar doubles and stubs for our other context.

### Feature Tests with JS

When we use `stripe.js` to charge a credit card, to test it in our feature specs we need to do something different. Remember, `stripe.js` sends a request to Stripe before submitting the form to our own server.

Let's make a feature spec for submitting a payment for our Todo app:

```ruby
# visitor_makes_payment_spec.rb

require 'spec_helper'

describe 'visitor makes payment' do
  scenario 'with valid card'

  scenario 'with invalid card number'

  scenario 'with declined card'
end
```

These cases have been tested extensively in our lower-level specs, but this is such an important part of the user experience we want to make sure everything is handled correctly.

Let's do the first scenario:

```ruby
# visitor_makes_payment_spec.rb

require 'spec_helper'

describe 'visitor makes payment' do
  scenario 'with valid card' do
    visit new_payment_path
    fill_in 'Credit card number', with: '4242424242424242'
    fill_in 'Security code', with: '123'
    select '3 - March', from: 'date_month'
    select '2017', from: 'date_year'
    click_button 'Submit Payment'

    expect(page).to have_content 'Thank you for your generous support.'
  end

  # ...
end
```

This test fails, since our Stripe token doesn't get passed on to our server. This is because we rely on JS to be turned on. We need to turn on JavaScript with a parameter in the `scenario` call:

```ruby
# visitor_makes_payment_spec.rb

require 'spec_helper'

describe 'visitor makes payment' do
  scenario 'with valid card', js: true do
    # ...
  end
  # ...
end
```

This will now make our spec pass IF we are using a test runner that supports JS. By default, Capybara uses a test runner that is much faster, but doesn't support JS. When we use `js: true`, Capybara automatically switches to a test runner that supports JS (`selenium`). It is the slowest one, but it is great for debugging, since we can watch what it is doing in our browser directly.

Now we write our other scenarios, (we also abstract out some of the logic of paying with a credit card):

```ruby
# visitor_makes_payment_spec.rb

require 'spec_helper'

describe 'visitor makes payment', js: true do # Move `
  scenario 'with valid card' do
    pay_with_credit_card '4242424242424242' # Changed!

    # ...
  end

  scenario 'with invalid card number' do
    pay_with_credit_card '40000000000000069'
    expect(page).to have_content('Your card\'s expiration date is incorrect')
  end

  scenario 'with declined card' do
    pay_with_credit_card '4000000000000002'
    expect(page).to have_content('Your card was declined')
  end

  def pay_with_credit_card(card_number)
    visit new_payment_path
    fill_in 'Credit card number', with: card_number
    fill_in 'Security code', with: '123'
    select '3 - March', from: 'date_month'
    select '2017', from: 'date_year'
    click_button 'Submit Payment'
  end
end
```

These tests pass. There are a few things to point out here. Selenium opens up a real Firefox browser. A major reason for tests not passing is that the version of Firefox you have installed might not be compatible with Selenium. The solution would be to upgrade everything to the latest version.

Furthermore, since Selenium is a very slow test runner, we can instead use `capybara-webkit` by adding it to our Gemfile. In order for this to work, we need to install `QT` with Homebrew. After we run Bundler, we need to add capybara-webkit to our `spec_helper.rb` as our default JS spec runner. We can do that with the instructions on the gem's Github page.

```ruby
# spec_helper.rb

Capybara.javascript_driver = :webkit
```

This won't open an actual browser, making our specs much faster. If for any scenario we want to have that visual information, we just add `driver: :selenium` to the call to `scenario`.

### Transactions and test database setup

Selenium needs to run off of a real HTTP server, so Capybara starts one on a different thread. This causes a problem, since by default we uses 'transactions' as a strategy to reset the database after each spec. `database_cleaner` makes it so we get to use a different strategy for cleaning the database after each test. To make it work with Selenium, we need to use 'truncation'. This is slower than using transactions, but this will guarantee a database reset.

We need to add the following after we include the `database_cleaner` gem:

```ruby
config.before(:suite) do
  DatabaseCleaner.clean_with(:truncation)
end

config.before(:each) do
  DatabaseCleaner.strategy = :truncation
end

config.before(:each, :js => true) do
  DatabaseCleaner.strategy = :truncation
end

config.before(:each) do
  DatabaseCleaner.start
end

config.after(:each) do
  DatabaseCleaner.clean
end
```
### Beyond MVC: Decorators

Here we're going to manage complexity to keep our codebase clean.

First we'll explore decorators. Our `Todo` model has the method `#display_text`, the purpose for which is purely a presentation concern. So let's extract this into a decorator.

First we create the dir `app/decorators`, which is automatically loaded, since Rails automatically loads everything under `app`. Here we create a decorator in the file `todo_decorator.rb`.

```ruby
# todo_decorator.rb

class TodoDecorator
  attr_reader :todo

  def initialize(todo)
    @todo = todo
  end

  def display_text
    todo.name + tag_text
  end

  private

  def tag_text
    # Move `tag_text` from `Todo` to here, making sure all references to the
    # `todo` instance var are appropriately changed.
  end
end
```

In our view, use `TodoDecorator` whenever we need this display text.

```haml
# index.html.haml

# ...
= link_to TodoDecorator.new(todo).display_text, todo
# ...
```

If we use these decorators a lot, we can define a convenience method in our model:

```ruby
# todo.rb

def decorator
  TodoDecorator.new(self)
end
```

So now we just need to call the method whenever we need display logic:

```ruby
a_todo.decorator.display_text
```

We often need to call methods both on the decorator and on the model, and we don't really want to have to know when to insantiate a decorator when we want a decorator method. To accomplish that, we can pass decorators into the view from the controller:

```ruby
# todos_controller.rb

# ...
@todos = current_user.todos.map &:decorator
# ...
```

In order to do that, we need the decorator to respond to methods defined both on the decorator as well as on the model itself. To do that, we extend the `Forwardable` module in our decorator, defining delegators to the `todo` instance variable.

```ruby
class TodoDecorator
  extend Forwardable

  def_delegators :todo, :name_only?
  attr_reader :todo

  # ...
end
```

Decorators are useful when we persist models in the database, and display them differently on the actual page. We use decorators to define presentation logic to bridge this gap. Since we're defining our domain logic separately from our presentation logic, we can test them separately.

The `draper` gem allows us to easily define decorators in Rails projects. For example, we can use `delegate_all` in our class definition, which delegates all methods from the parent class. It uses convention-over-configuration, which allows us to just define a class, e.g., `PostDecorator` to have it understand that it is decorating the `Post` model.

We can push a lot of helper methods and inline view logic into decorators in order to separate concerns for a cleaner solution.

### Beyond MVC: Policy Objects

In our `TodosController#create` action, we have some complicated logic. We can improve this code by different kinds of objects.

I'll describe how the action currently works, since I think I might have skipped taking notes on a previous lesson:

1. When the Todo is created (`@todo.save_with_tags`), we have some nested logic which checks if the account is old or the user is a premium user.
2. If the user meets either of those criteria, we deduct 1 from the user's `credit_balance`. Else, deduct 2.
3. Save the user.
4. Then we check if the balance is less than 0. If so, we send a mailer notifying the user of insufficient credit. Else if the balance is less than 10, we send a mailer notifying the user of a low balance.
5. Redirect to root.

(1) and (2) represent a specific piece of knowledge in our application of the state of the current user. It indicates that we have different types of users, and we determine that type using the `if` statement.

We can use a policy object under `app/models`:

```ruby
# app/models/user_level_policy.rb

class UserLevelPolicy
  attr_reader :user

  def initialize(user)
    @user = user
  end

  def premium?
    user.created_at < Date.new(2010, 1, 1) || user.plan.premium?
  end
end
```

Then we create an object when we do the check:

```ruby
# todos_controller.rb

# ...
def create
  @todo = Todo.new todo_params
  if @todo.save_with_tags
    if UserLevelPolicy.new(current_user).premium?
      # ...
    end
    # ...
  end
end
# ...
```

We're using a policy to capture a specific piece of knowledge about the user. This policy object isn't that complicated, but we could have many more criteria. This allows us to make easy changes, and it can be referred to by other pieces of the application.

### Beyond MVC: Domain Objects

Domain objects are objects for domain models: models in the context of the application domain. We already have domain objects in our app, that is, those which inherit from `ActiveRecord::Base`. However, we often times have models that don't map to database tables.

For instance, the concept of user credits is important for our application--it's stored as an attribute in the `users` table. We always have to reach for the user, get the balance, and manipulate it. To persist it, we need to save it.

Instead, let's abstract the concept of credit into its own model which we can operate on directly without having to go through `users`. We don't need it to map to a database table. Since it's always attached to the user, we need to pass in the user on creation.

```ruby
# app/models/credit.rb

class Credit
  attr_reader :credit_balance

  def initialize(user)
    @credit_balance = user.current_credit_balance
  end
end
```

Now, we would like to do a few things with `Credit`. We want to manipulate the balance, save the credit, and query the credit balance.

Let's define the `-` operator and `save` method.

```ruby
# credit.rb

class Credit
  attr_accessor :user, :credit_balance

  def initialize(user)
    @credit_balance = user.current_credit_balance
    @user = user
  end

  def -(number)
    credit_balance = credit_balance - number
  end

  def save
    @user.current_credit_balance = @credit_balance
    @user.save
  end
end
```

So we add `@user` to keep track of the user, so we can then store the credit balance and save it.

Notice, now, that we have specific rules in our app for the balance being less than 0 or less than 10. These are obviously not arbitrary numbers, so we want to define a couple more methods to reflect that.

```ruby
# credit.rb

class Credit
  attr_accessor :user, :credit_balance

  def initialize(user)
    @credit_balance = user.current_credit_balance
    @user = user
  end

  def -(number)
    credit_balance = credit_balance - number
  end

  def save
    @user.current_credit_balance = @credit_balance
    @user.save
  end

  def depleted?
    credit_balance < 0
  end

  def low_balance?
    credit_balance < 10
  end
end
```

Now that we have our domain model, let's go back to the controller and use it.

```ruby
# todos_controller.rb

# ...
  def create
    @todo = Todo.new todo_params
    credit = Credit.new(current_user)

    if @todo.save_with_tags
      if UserLevelPolicy.new(current_user).premium?
        credit = credit - 1
      else
        credit = credit - 1
      end

      credit.save

      if credit.depleted?
        # Send insufficient credit email
      elsif credit.low_balance?
        # Send low credit email
      end

      # ...
    end
  end
# ...
```

Now, all the business logic based around credit is tucked away into a domain model. Therefore, we can add to the complexity of the handling of credit in a safe way. We can also simply extend from `ActiveRecord::Base` if we want credit to persist in the database as its own object. We can also test it more easily, alone rather than through the request/response cycle of a controller.

The point of domain objects is to decouple certain models from the way they are persisted.

### Beyond MVC: Service Objects

In our Todos controller, `create` action, we have a bunch of code that determines how we deduct credit from the user's account.

Just like how we can use domain objects to make a concept in the application domain more explicit, we can also use service objects to make a business process in the application domain more explicit. Deducting credits is an example of such a business process. We can make a service object for this.

We make a directory under `app` called `services`, where we create our service `credit_deduction.rb`.

```ruby
# app/services/credit_deduction.rb

class CreditDeduction

end
```

There are a few conventions for naming service object. We could use the verb form `DeductCredit`, which looks a little odd when we instantiate and use these objects. So we can use the noun form `CreditDeduction`. Others like to model them in the form of `CreditDeductor` or `CreditManager`, but we'll stick to what we have.

```ruby
# app/services/credit_deduction.rb

class CreditDeduction
  attr_accessor :credit

  def initialize(credit)
    @credit = credit
  end

  # We place the logic from the controller here.
  def deduct_credit
    if UserLevelPolicy.new(credit.user).premium?
      credit = credit - 1
    else
      credit = credit - 1
    end

    credit.save

    if credit.depleted?
      # Send insufficient credit email
    elsif credit.low_balance?
      # Send low credit email
    end
  end
end
```

In the video, Kevin passed in the `user` to the `initialize` method, but we don't need to do that, since `credit` has a `user` getter.

In our controller, we just need to create a new `CreditDeduction`.

```ruby
# todos_controller.rb

# ...
  def create
    @todo = Todo.new todo_params
    credit = Credit.new current_user

    if @todo.save_with_tags
      CreditDeduction.new(credit).deduct_credit

      redirect_to root_path
    else
      render :new
    end
  end
# ...
```

We can improve this by instantiating a `Credit` within `CreditDeduction`, passing in the `user` instead.

```ruby
# app/services/credit_deduction.rb

class CreditDeduction
  attr_accessor :credit, :user

  def initialize(user)
    @user = user
    @credit = Credit.new(user)
  end

  # We place the logic from the controller here.
  def deduct_credit
    if UserLevelPolicy.new(user).premium?
      credit = credit - 1
    else
      credit = credit - 1
    end

    credit.save

    if credit.depleted?
      # Send insufficient credit email
    elsif credit.low_balance?
      # Send low credit email
    end
  end
end
```

Then we just need to modify our action like so:

```ruby
# todos_controller.rb

# ...
  def create
    @todo = Todo.new todo_params

    if @todo.save_with_tags
      CreditDeduction.new(user).deduct_credit

      redirect_to root_path
    else
      render :new
    end
  end
# ...
```

### Beyond MVC: Object Composition, Object Oriented Design, and YAGNI

Let's look at what we've done:

Our controller collaborates with `CreditDeduction`, and that collaborates with `Credit` and `UserLevelPolicy`, as well as `AppMailer`.

We broke up a long block of procedural code down to a bunch of simpler collaborators that work together to accomplish the task. This is known as "object composition," or using simple objects with few responsibilities to compose complex tasks.

Rails provides an MVC framework, but as our application grows, the MVC itself is no longer enough to hold the complexity of the application. When that happens, people typically push down the complexity of the app to the Model layer to express the "skinny controller fat model" pattern. However, our models can become too bloated fairly quickly and assume too many responsibilities.

This is especially true with the models that are central to our applicaiton domain: "god objects." For example, the `User` model in many domains can have too many responsibilities and collaborate with too many other things. These objects become ones that everybody is afraid of touching, because when people mess with them they can break very easily.

To fix this pattern, we can take multiple simple objects.

The cost of object composition is indirection. When we use this pattern we often have to chase down workflows in different classes to know what exactly happens. This doesn't have to be too bad if things are well defined with good names.

Another antipattern with composition is YAGNI: you aren't gonna need it. Say if we wanted to abstract the parts of `deduct_credit` in the `CreditDeduction` class. This can be seen as premature, since those components are likely only to be used once. In these cases it's good to err on the side of not expecting to need those concerns abstracted away. It's best just to wait and make that abstraction later.

With this in mind, we may not even need to abstract the `UserLevelPolicy` into its own object, but rather have the `premium?` method on the `User` model itself. If in the future we come to the point where we have multiple user levels and the logic for determining them becomes more complex, that would be the time to abstract out such an object.

### Message Expectations

A message expectation in rspec is a matcher that expects an object to receive a certain message (e.g. a method call). If we have:

```ruby
logger.should_receive(:account_closed).with(account)

account.close
```

`logger` is a double or an object, and we're expecting the method `account_closed` to be invoked with the paramter `account`, which is defined outside of that block.

`account.close` is what we want to have trigger the message. Note that we put the expectation before the trigger.

We can also add a return value:

```ruby
receiver = double()
receiver.should_receive(:message).and_return(:return_value)
```

This stubs the method and makes the invocation of `message` return `:return_value`. This can all also be accomplished with the `expect` syntax.

We might also not be able to get a handle on a specific object, so we can have rspec expec any instance of a class to receive a message:

```ruby
Object.any_instance.should_receive(:foo).and_return(:bar)

o = Object.new
expect(o.foo).to eq(:bar)
```

The above test will pass.

If we want to test both at a message is received and verify that the return value of that is correct (remember that `and_return` will stub the method and invoking that method in subsequent lines will cause a false positive), we can use `and_call_original`.

```ruby
foo.should_receive(:bar).and_call_original
foo.bar.should eq(:baz)
```

We can also assert that a message is received a certain amount of times.

```ruby
foo.should_receive(:bar).once
```

### Mocking

So let's use message expectations to write tests for our credit deduction object composition.

The `TodosController` is collaborating with `CreditDeduction`, meaning it is delegating the `deduct_credit` task to an instance of the service object. When we do testing for this kind of object collaboration, we only need to verify that it is passing the message. We don't need to test what happens after the fact, since we have hopefully provided test coverage for the `CreditDeduction` task. W/R/T the `TodosController` class we only have to go so far as to assert that the object has indeed sent a message to the service object.

This technique is called mocking. So far we have only been making assertions on values or the state of the system. When we mock, rather than asserting on values, we assert on communication. When we use mocks to do testing, we mock all the collaborators of the unit we are testing, verifying that it is sending the correct messages to those collaborators.

In this case, we are going to make the call to `CreditDeduction.new` a test double, and make sure the double receives a message `deduct_credit`. Let's add a spec:

```ruby
# todos_controller_spec.rb

# ...
  describe 'POST create' do
    # ...
    it 'delegates to CreditDeduction to deduct credit' do
      credit_deduction = double('credit_deduction')
      expect(CreditDeduction).
        to_receive(:new).
        with(alice).
        and_return(credit_deduction)

      expect(credit_deduction).to_receive(:deduct_credit)

      post :create, todo: { name: 'cook', description: 'hello' }
    end
    # ...
  end
# ...
```

Another example: let's look at the `Credit` class. It looks like our instance methods fall under two categories. `#depleted?` and `#low_balance?` are query methods, which don't change the state of our object. To test these, we can set instance variables and test the output. The methods that mutate our object (such as `#-`) are tested similarly, checking the state after the method is invocated.

The `#save` method goes beyond the current `Credit` object. It relies on a collaborator, namely a `User` object. In order to do a true unit test of this object, we can use a test double for the collaborator object. We can then just test that the test double receives a certain message. In this case we want to make sure that the `user` instance variable receives `current_credit_balance=`, taking in `credit_balance` as a parameter, as well as `save`.

Mocks allow us to scope our tests to limit them to individual units. They also help us hone our object oriented design skills. If we find ourselves creating too many mocks, that means our unit has too many collaborators. Just by observing that pain will lead us to a better-designed system.

### Sandi Metz's Testing Talk

The three kinds of messages, the two flavors of each kind, and how we should test them:

- Incoming Query
  - Test by making assertions about what they send back. (Assert result)
  - Test the interface, not the implementation.
- Incoming Command
  - Test by makking assertions about _direct public_ side effects.
  - By 'direct' we mean the responsibility of the last ruby class involved.
  - Incoming messages are the only place where we make assertions about values.
- Sent to Self
  - Don't test! This does not add any value.
  - Tests that assert that we send methods to self bind us to a specific implementation.
- Outgoing Query
  - Don't test these!
  - Don't test that we've sent them!
  - The reason for this is that these messages are invisible to the rest of the app.
- Outgoing Command
  - These are a little different, since in most cases these message MUST be sent. It creates side effects upon which others depend.
    - Do not test these side effects.
    - This creates a dependency between the object and the distant side effect. It also is technically an integration test.
  - Test instead with mocks.
    - The test depends on a message, testing at the 'nearest edge'.
    - You do not have to run all the code between the message and the distant side effect.
    - This is fast and stable.
    - Break the rule when side effects are stable and cheap (close).

These tests allow us to write a minimal amount of tests while keeping 100% test coverage.

Trust collaborators, insist on simplicity.

#### On the fragility of mocks

API drift happens, and this is natural. We just have to implement mocks directly.

A mock is a test double, playing the role of some object in our real app. They promise that they implement a common API with the acutal objects. Our doubles must keep that promise, keeping them in sync with the API.

How do we do this? There are new libs on Github that allow us to use mocks and stubs in our tests and know with confidence that they won't stray from the actual API.

## Week 8

### Model Serialization

Serialization is the conversion of an object into a string to be sent to another system. We can only send strings over the wire. With any ActiveRecord object we can just use `#to_json`, which just gives keys and values for every column in the table.

Let's define a method `#as_json`:

```ruby
# video.rb

def as_json(options = {})
  # ...
end
```

This method will return a hash. Whenever we call `#to_json`, it will turn the hash returned by `#as_json` into a JSON string. There are a couple of ways we can do this. We can explicitly define a hash:

```ruby
# video.rb

def as_json(options = {})
  { title: title }
end
```

or we can call `#super`, passing in an `options` hash, for `only` or `except`, etc.:

```ruby
# video.rb

def as_json
  super(only: [:title])
end
```

The reason we want to serialize is that we want to communicate between the Rails app and the elasticsearch server. The elasticsearch server is most often located on a different machine, so we have to turn the video into a JSON format to send over the wire. We could update `#as_json`, which is good, since we only want to send the data we want to search on. The less data we send, the faster and more performant the query will be when we search against the elasticsearch server.

The issue is that this converts all `#to_json` calls on a `Video` object to this. We may also need to communicate with other systems or serve as an API server, which will require another set of attributes.

To get around this, we just need to define `#as_indexed_json`:

```ruby
# video.rb

def as_indexed_json(options = {})
  as_json only: [:title]
end
```

This is given to us by `Elasticsearch::Model`, but we can overwrite it. By default, it just calls `#as_json` with no options.

#### Associations

If we have an association in our search (video.reviews.body, for example), we need to make sure the elasticsearch index updates when a review is added:

```ruby
# review.rb

class Review < ActiveRecord::Base
  # ...
  belongs_to :video, touch: true
  # ...
end
```

This updates the `updated_at` field on the associated video, triggering elasticsearch callbacks.

#### Relevance Weights

If we want to assign weights to different fields, we need to change our options hash we pass into `__elasticsearch__.search`:

```ruby
search_definition = {
  query: {
    multi_search: {
      query: q,
      fields: ['title^100', 'description^50'],
      operator: 'and'
    }
  }
}
```

We boost with the `^` operator.
