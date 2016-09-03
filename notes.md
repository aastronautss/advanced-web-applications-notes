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

