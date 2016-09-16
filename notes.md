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
