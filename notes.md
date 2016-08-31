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
