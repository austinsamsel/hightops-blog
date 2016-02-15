---
title: Getting Started with Cucumber on Rails
date: 2015-08-31 17:09 UTC
tags: software development
---

<p>This is a "getting started" with Cucumber and BDD testing in rails. We're literally jumping straight into it. I'm assuming you've already read up on some testing concepts and you've found your way here in order to understand what a workflow with Cucumber might look like and how we think about problems along the way. If you like, you can grab the finished code from <a href="https://github.com/austinsamsel/rails-cuke-todo">GitHub</a>.</p>

<h3>Step 1 - Set up.</h3>

<p>Create a new app</p>

```ruby
$ rails new todos-tdd-cucumber --skip-test-unit
```

<p>Add Cucumber to your Gemfile. We'll include database_cleaner, which comes highly recommended as per <a href="https://github.com/cucumber/cucumber-rails">cucumber docs</a></p>

```ruby
group :test do
  gem 'cucumber-rails', require: false
  gem 'database_cleaner'
end

$ bundle install
```

<p>Run cucumber installer.</p>

```ruby
$ rails generate cucumber:install
```

<p>This gives us a new directory called <em>features</em> which is where we'll write tests.</p>

<h3>Step 2 - Create first feature: Add a Task.</h3>

<p>Create the feature. Using (Gherkin)[https://github.com/cucumber/cucumber/wiki/Gherkin] - an easy to read syntax that works as both documentation and as tests. Your clients can read it and you can write it together. Even if you're not working with a client (or a client that wants to read it), its still worthwhile to use it because it forces you to "think like the user" as you write your features. Rather than mapping out your database and building your models, you'll develop your features with an "outside in" approach, starting from the homepage, and working with what you would see in the browser.</p>

```ruby
$ touch features/todos.feature
```

<p>Your feature should express what you'd like to build and conclude with the business value behind it.</p>

```ruby
#todos.feature

Feature: Todos
  In order to get things done
  As a todo-list freak
  I want to be able to create a list of tasks.
```

<p>Great. Let's write our first scenario. Each scenario is composed of steps that begin with keywords like <em>Given</em> <em>And</em> <em>When</em> and finally <em>Then</em>.</p>

```ruby
#todos.feature
...

Scenario: Create a task
  Given I am on the home page
  And I go to "Add new task"
  When I fill in "Task Field" with "My first todo!"
  And I press "Submit"
  Then I should see "My first todo!"
```

<p>Run cucumber for the first time as a rake task.</p>

```ruby
$ rake cucumber
```

<p>Copy and paste the console output into our step definitions file.</p>

```ruby
$ touch features/step_definitions/todos.rb
```

<!-- -->


<p></p>

```ruby
#features/step_definitions/todos.rb

Given(/^I am on the home page$/) do
  pending # express the regexp above with the code you wish you had
end

Given(/^I go to "(.*?)"$/) do |arg1|
  pending # express the regexp above with the code you wish you had
end

When(/^I fill in "(.*?)" with "(.*?)"$/) do |arg1, arg2|
  pending # express the regexp above with the code you wish you had
end

When(/^I press "(.*?)"$/) do |arg1|
  pending # express the regexp above with the code you wish you had
end

Then(/^I should see "(.*?)"$/) do |arg1|
  pending # express the regexp above with the code you wish you had
end
```

<p>Whenever we want some feedback from cucumber, we'll run <em>$ rake cucumber</em>  And we'll see that it tells us our first step is pending.</p>

<p>Fill out the first step definition with the following.</p>

```ruby
#features/step_definitions/todos.rb

Given(/^I am on the home page$/) do
  visit "/"
end
...
```

<p>This tells Cucumber we expect to be on the home page to start things off.</p>

<p>Cucumber tells us</p>

```ruby
No route matches [GET] "/" (ActionController::RoutingError)
```

<p>What we need is a route to our homepage. Even if we build a route, we know we'll need a view and a controller (plus an index) to get it started...</p>

<p>Let's add a route.</p>

```ruby
# config/routes.rb
...
root 'tasks#index'
...
```

<p>Run cucumber again.</p>

```ruby
uninitialized constant TasksController (ActionController::RoutingError)
```

<p>Cucumber tells us we need a controller for our tasks. This is really TDD. Just taking one step, then another, letting the tests dictate each next step in development.</p>

<p>Let's create a controller.</p>

```ruby
$ rails g controller Tasks home --no-helper --no-assets
```

<p>This will also give us the view we need as well at <em>app/views/tasks/home.html.erb</em></p>

<p>When we run cucumber again, our first step passes! Now our next step is pending. So let's write the code for that.</p>

<p>Now we need to write the code for our next feature. We want to go to a page to add a new task. So we'll need a link users can click. We'll update the argument so its human readable, the_link and reference it in the code.</p>

```ruby
Given(/^I go to "(.*?)"$/) do |the_link|
  click_link the_link
end
```

<p>Cucumber tells us:</p>

```ruby
Unable to find link "Add new task" (Capybara::ElementNotFound)
```

<p>Now we have a failing test. It's time to update our application code with a link to add a new task.</p>

```ruby
#app/views/tasks/home.html.erb
&lt;%= link_to "Add new task", new_task_path %&gt;
```

<p>Cucumber tells us:</p>

```ruby
undefined local variable or method `new_task_path' for #&lt;#&lt;Class:0x007fcc04b53060&gt;:0x007fcc04b51dc8&gt; (ActionView::Template::Error)
```

<p>We don't have a route for new. So let's change our code in our routes to:</p>

```ruby
config/routes.rb
...
root 'tasks#home'
resources :tasks
...
```

<p>Cucumber tells us:</p>

```ruby
The action 'new' could not be found for TasksController (AbstractController::ActionNotFound)
```

<p>So we need to update our controller with a new action.</p>

```ruby
#app/controllers/task_controller.rb
class TasksController &lt; ApplicationController
  def new
  end
end
```

<p>Cucumber tells us:</p>

```ruby
Missing template tasks/new, application/new with {:locale=&gt;[:en], :formats=&gt;[:html], :variants=&gt;[], :handlers=&gt;[:erb, :builder, :raw, :ruby, :coffee, :jbuilder]}. Searched in:
```

<p>We need a view template for our new action.</p>

```ruby
$ touch app/views/tasks/new.html.erb
```

<p>Rake tells us this step passes!</p>

<p>Let's write the code for our next step.</p>

```ruby
# features/step_definitions/todos.rb
...
When(/^I fill in "(.*?)" with "(.*?)"$/) do |input, value|
  fill_in input, with: value
end
...
```

<p>We updated the argument variables from <em>arg1, arg2</em> to <em>input, value</em> to be more readable.</p>

<p>When we run cucumber, we get:</p>

```ruby
Unable to find field "Task Field" (Capybara::ElementNotFound)
```

<p>Cucumber is looking for the form to add a new task. Let's create the form. First let's add Simple Form to our gemfile.</p>

```ruby
#Gemfile
...
gem 'simple_form'
...

$ bundle install

$ rails g simple_form:install
```

<!-- -->


```ruby
#app/views/tasks/home.html.erb
&lt;%= simple_form_for(@task) do |f| %&gt;
  &lt;%= f.input :name %&gt;
  &lt;%= f.button :submit, "Submit" %&gt;
&lt;% end %&gt;
```

<p>Run Cucumber and we get:</p>

```ruby
undefined method `model_name' for nil:NilClass (ActionView::Template::Error)
```

<p>We need to update our controller action that will allow us to create a new task.</p>

```ruby
#app/controllers/tasks_controller.rb
...
def new
  @task = Task.new()
end
...
```

<p>We need to create a model.</p>

```ruby
$ rails g model Task name:string
$ rake db:migrate
```

<p>Then we get...</p>

```ruby
Unable to find field "Task Field" (Capybara::ElementNotFound)
```

<p>Capybara is having trouble finding the field for which we'd like to add in the name of our task. It's time to take a second look at our original step definition and enter a revision. If you run your rails server and inspect the source code, our rails form field looks like this: <code class="language-ruby">&lt;input class="string optional" type="text" name="task[name]" id="task_name" /&gt;</code> Cucumber is able to find the field by name or ID. So let's update our feature with the id as the form field identifier:</p>

<p>Our step definition passes.</p>

<p>Let's write the code for our third step definition.</p>

```ruby
#features/step_definitions/todos.rb
...
When(/^I press "(.*?)"$/) do |the_button|
  click_button the_button
end
...
```

<p>Run cucumber and we get:</p>

```ruby
The action 'create' could not be found for TasksController (AbstractController::ActionNotFound)

We need a create action in our controller.

#app/controllers/task_controller.erb
...
def create
  @task = Task.new
end
...
```

<p>Run Cucumber.</p>

```ruby
Missing template tasks/create, application/create with {:locale=&gt;[:en], :formats=&gt;[:html], :variants=&gt;[], :handlers=&gt;[:erb, :builder, :raw, :ruby, :coffee, :jbuilder]}. Searched in:
```

<p>By default Rails is trying to take us to a new page... Since this is just a one page app, let's keep everything on the home page. And so our app is secure, I'm including in params at this time.</p>

```ruby
#app/controllers/task_controller.erb
...
def create
  @task = Task.new(task_params)
  if @task.save
    redirect_to root_url
  end
end

private
  def task_params
    params.require(:task).permit(:name)
  end
...
```

<p>Our step passes. Now we should verify that we can see our new task. Let's finish out the code for the fourth step. Where we test to make sure we can actually see the new task posted.</p>

```ruby
#features/step_definitions/todos.rb
...
Then(/^I should see "(.*?)"$/) do |task|
  assert page.has_content?(task)
end
...
```

<p>Run Cucumber:</p>

```ruby
Failed assertion, no message given. (Minitest::Assertion)
```

<p>This isn't that informative. But considering we haven't done anything to actually list tasks, let's start with the view.</p>

```ruby
#app/views/tasks/home.html.erb
...
&lt;% for task in @tasks %&gt;
  &lt;li&gt;&lt;%= task.name %&gt;&lt;/li&gt;
&lt;% end %&gt;
```

<p>Run Cucumber:</p>

```ruby
undefined method `each' for nil:NilClass (ActionView::Template::Error)
```

<p>We don't have anything in the controller to feed the view any data. So let's update that.</p>

```ruby
#app/controllers/task_controller.rb
...
def home
  @tasks= Task.all
end
...
```

<p>Then all our tests pass.</p>

<h3>Step 3: Create a failing test</h3>

<p>What happens when the user does something weird? Do we want users to create empty tasks? No. So let's test to make sure that invalid tasks are not posted.</p>

<p>Let's write a feature:</p>

```ruby
#features/todos.feature
...
Scenario: Create an invalid task
  Given I am on the home page
  And I go to "Add new task"
  When I fill in "task_name" with ""
  And I press "Submit"
  Then I should be told "Can't be blank"
  And I fill in "Name" with "My first todo!"
  And I press "Submit"
  Then I should see "My first todo!"
```

<p>There's some repetition here in the steps. We're only going to have to write one new step definition. The rest is repeat. The new step definition is <code class="language-ruby">Then I should be told "Can't be blank"</code></p>

<p>When we run Cucumber, it gives us our new step definition to implement. We'll copy and paste it into our todos.rb</p>

```ruby
#features/step_definitions/todos.rb
...
# invalid post
Then(/^I should be told "(.*?)"$/) do |arg1|
  pending # express the regexp above with the code you wish you had
end
```

<p>And let's update this step definition with our test code.</p>

```ruby
#features/step_definitions/todos.rb
...
# invalid post
Then(/^I should be told "(.*?)"$/) do |error_message|
  assert page.has_content?(error_message)
end
```

<p>Cucumber tells us:</p>

```ruby
Failed assertion, no message given. (Minitest::Assertion)
```

<p>Not that informative. But we know we don't have any validations on our task model. So let's update that.</p>

```ruby
#app/models/task.rb
class Task &lt; ActiveRecord::Base
  validates :name, presence: true
end
```

<p>Cucumber tells us:</p>

```ruby
Missing template tasks/create, application/create with {:locale=&gt;[:en], :formats=&gt;[:html], :variants=&gt;[], :handlers=&gt;[:erb, :builder, :raw, :ruby, :coffee, :jbuilder]}. Searched in:
```

<p>It looks like Rails is trying to reroute us. We want to stay on the same page and see the error. Let's update the controller action with a scenario detailing what happens when a task is not saved.</p>

```ruby
#app/controllers/tasks_controller.rb
...
def create
  @task = Task.new(task_params)
  if @task.save
    redirect_to root_url
  else
    render :new
  end
end
...
```

<p>Cucumber tells us:</p>

```ruby
Failed assertion, no message given. (Minitest::Assertion)
```

<p>We still aren't being told of the failure. If we run <em>rails s</em> to start the server and see what's going on when we submit a blank task, we're told "can't be blank". It is working! Cross referencing with our feature, we previously wrote, "Can't be blank". So we need to update our feature with the proper capitalization.</p>

```ruby
#features/todos.feature
...
# old
# Then I should be told "Can't be blank"
Then I should be told "can't be blank"
...
```

<p>Now all our tests pass.</p>

<h3>Step 4: Completing Tasks</h3>

<p>Let's write the scenario for marking a task as complete.</p>

```ruby
Scenario: Mark a task as completed
  Given I have the following tasks:
    | name          | completed |
    | My First Todo | false     |
    | My 2nd Todo   | false     |
  When I am on the home page
  And I follow "Edit" associated with "My 1st Todo"
  And I check the "Done" checkbox
  And I press "Submit"
  Then I should see "My 1st Todo" as completed.
```

<p>You'll notice a new format here. We created some sample data within Cucumber that can run. It's a simple and fast way to start writing more complex tests.</p>

<p>We'll run cucumber and grab the output and paste it in our <em>step_definitons/todos.rb</em> file.</p>

<p>In order to work with the data in the fixtures, we'll modify our first step definition:</p>

```ruby
Given(/^I have the following tasks:$/) do |table|
  for hash in table.hashes
    Task.create(hash)
  end
end
```

<p>Then we're told:</p>

```ruby
unknown attribute 'completed' for Task. (ActiveRecord::UnknownAttributeError)
```

<p>We never created the a field for "completed" so let's add that now.</p>

```ruby
$ rails g migration AddCompletedToTasks completed:boolean
$ rake db:migrate
```

<p>Our second step passes. Now we need to write another step definition:</p>

```ruby
When(/^I follow "(.*?)" associated with "(.*?)"$/) do |arg1, arg2|
  first('li').click_link('Edit')
end
```

<p>This will find the first "Edit" link on the page, which will be associated with our first todo. Capybara will tell us it can't find it:</p>

```ruby
Unable to find link "Edit" (Capybara::ElementNotFound)
```

<p>We'll add an edit link into our view.</p>

```ruby
#app/views/tasks/home.html.erb
...
&lt;% for task in @tasks %&gt;
  &lt;li&gt;&lt;%= task.name %&gt; • &lt;%= link_to 'Edit', edit_task_path(task) %&gt;&lt;/li&gt;
&lt;% end %&gt;
```

<p>Then Cucumber tells us we don't have a controller action:</p>

```ruby
The action 'edit' could not be found for TasksController (AbstractController::ActionNotFound)
```

<p>We'll add the action to our tasks controller:</p>

```ruby
#app/controllers/tasks_controller.rb

def edit
end
```

<p>Now we're missing our view</p>

```ruby
Missing template tasks/edit, application/edit with {:locale=&gt;[:en], :formats=&gt;[:html], :variants=&gt;[], :handlers=&gt;[:erb, :builder, :raw, :ruby, :coffee, :jbuilder]}. Searched in:
```

<p>We'll add it in. Again just doing the bare minimum at each step.</p>

```ruby
$ touch app/views/tasks/edit.html.erb
```

<p>Our step passes. We now need to write the code for the next step definition to test completing a task.</p>

```ruby
When(/^I check the "(.*?)" checkbox$/) do |arg1|
  page.find('input[type=checkbox]').set(true)
end
```

<p>Cucumber:</p>

```ruby
Unable to find css "input[type=checkbox]" (Capybara::ElementNotFound)
```

<p>All we have is a blank page. Its time to add in a form to edit the task on our edit page. It's the same form as on the new view, except with a new field to mark todos as completed.</p>

```ruby
#app/views/tasks/edit.html.erb

&lt;%= simple_form_for(@task) do |f| %&gt;
  &lt;%= f.input :name %&gt;
  &lt;%= f.input :completed, as: :boolean, checked_value: true, unchecked_value: false %&gt;
  &lt;%= f.button :submit, "Submit" %&gt;
&lt;% end %&gt;
```

<p>We get:</p>

```ruby
undefined method `model_name' for nil:NilClass (ActionView::Template::Error)
```

<p>There's nothing in the controller to help rails find the task. So let's update the edit method:</p>

```ruby
#app/controllers/task_controller.rb
def edit
  @task = Task.find(params[:id])
end
```

<p>Cucumber tells us:</p>

```ruby
The action 'update' could not be found for TasksController (AbstractController::ActionNotFound)
```

<p>Let's add an update action to our tasks controller.</p>

```ruby
#app/controllers/task_controller.rb

def update
end
```

<p>Cucumber says:</p>

```ruby
Missing template tasks/update, application/update with {:locale=&gt;[:en], :formats=&gt;[:html], :variants=&gt;[], :handlers=&gt;[:erb, :builder, :raw, :ruby, :coffee, :jbuilder]}. Searched in:
```

<p>We don't want to create a view for update... so let's try updating the action to include some rerouting... After a successful update we want to go to the home page and if there's been an error, we want to stay on the edit page.</p>

```ruby
#app/controllers/task_controller.rb

def update
  if @task.update
    redirect_to root_url
  else
    render :edit
  end
end
```

<p>One more problem, rails can't find the task we're trying to update... so let's copy over the same find code from the edit action...</p>

```ruby
#app/controllers/task_controller.rb

def update
  @task = Task.find(params[:id])
  if @task.update
    redirect_to root_url
  else
    render :edit
  end
end
```

<p>And now...</p>

```ruby
wrong number of arguments (0 for 1) (ArgumentError)
```

<p>We need to fix our strong parameters in our private task_params method.</p>

```ruby
#app/controllers/task_controller.rb

def task_params
  params.require(:task).permit(:name, :completed)
end
```

<p>We also need to pass our params into the update method.</p>

```ruby
#app/controllers/task_controller.rb

def update
  @task = Task.find(params[:id])
  if @task.update(task_params)
    redirect_to root_url
  else
    render :edit
  end
end
```

<p>The step passes. Before we go on, let's refactor the code so we aren't repeating the same line of code in the edit and update methods in the controller. Remove the following line from the edit and update actions.</p>

```ruby
@task = Task.find(params[:id])
```

<p>Then create a new private method:</p>

```ruby
#app/controllers/task_controller.rb

def find_task
  @task = Task.find(params[:id])
end
```

<p>And at the top of our controller we'll call our new method with a before_action hook:</p>

```ruby
#app/controllers/task_controller.rb

before_action :find_task, only: [:edit, :update]
```

<p>If we run cucumber again and see it still working, we were just able to "fearlessly" refactor.</p>

<p>Finally, let's finish this last step. We need to communicate to the user that the task is completed.</p>

```ruby
Then(/^I should see "(.*?)" as completed\.$/) do |task|
  assert first('li').parent.has_css?('.completed')
end
```

<p>This code checks if our list item has the css class <em>.completed</em>. The argument has_css? looks within the list item to its children, when we wanted to check the list item itself. So we used <em>.parent</em> to return the search to the list item element itself.</p>

<p>Let's update the home page view with some logic that determines if the task is marked as completed, and if it is, to add the class, "completed"</p>

```ruby
#app/views/tasks/home.html.erb

&lt;li class="&lt;%= task.completed == true ? "completed" : "" %&gt;"&gt;&lt;%= task.name %&gt; • &lt;%= link_to 'Edit', edit_task_path(task) %&gt;&lt;/li&gt;
```

<p>You may want to move this logic out into a helper method, but for now, this will adequately accomplish what we want.</p>

<p>Then let's add a new CSS file:</p>

```CSS
$ touch app/assets/stylesheets/tasks.css
```

<p>And add the code:</p>

```CSS
#app/assets/stylesheets/tasks.css

li.completed{
  text-decoration: line-through;
}
```

<p>Then we can run cucumber and verify that all our tests are passing. It kind of sucks that we have a line running through even the edit and delete actions, but oh well - this isn't about being pretty, but just showing how this can all work.</p>

<h3>Step 5: Removing Tasks</h3>

<p>When a user has completed a bunch of tasks, after a time, it makes sense to that our user may no longer want to view the tasks. So we need a way to delete tasks.</p>

<p>So let's write the following scenario:</p>

```ruby
Scenario: Delete a task
  Given I have the following tasks:
    | name          | completed |
    | My First Todo | false     |
    | My 2nd Todo   | false     |
  When I am on the home page
  And I press "Delete" associated with "My First Todo"
  Then I should no longer see "My First Todo"
```

<p>Let's write our first step. We want to tell Cucumber to click the delete link:</p>

```ruby
When(/^I click on the "(.*?)" next to "(.*?)"$/) do |arg1, arg2|
  first('li').click_link('Delete')
end
```

<p>Cucumber tells us:</p>

```ruby
Unable to find link "Delete" (Capybara::ElementNotFound)
```

<p>Let's add a delete link to the Home page view.</p>

```ruby
#app/views/tasks/new.html.erb

&lt;% for task in @tasks %&gt;
  &lt;li class="&lt;%# task.completed == true ? "completed" : "" %&gt;"&gt;
    &lt;%= task.name %&gt;
    • &lt;%= link_to 'Edit', edit_task_path(task) %&gt;
    • &lt;%= link_to 'Delete', task, method: :delete, data: { confirm: 'Are you sure?' } %&gt;
  &lt;/li&gt;
&lt;% end %&gt;
```

<p>Cucumber tells us:</p>

```ruby
The action 'destroy' could not be found for TasksController (AbstractController::ActionNotFound)
```

<p>Let's add the necessary controller action:</p>

```ruby
#app/controllers/task_controller.rb

def destroy
end
```

<p>Cucumber tells us:</p>

```ruby
Missing template tasks/destroy, application/destroy with {:locale=&gt;[:en], :formats=&gt;[:html], :variants=&gt;[], :handlers=&gt;[:erb, :builder, :raw, :ruby, :coffee, :jbuilder]}. Searched in:
```

<p>Let's fill out the action with:</p>

```ruby
#app/controllers/task_controller.rb

def destroy
  @task.destroy
end
```

<p>And also include the destroy action in the before_action hook:</p>

```ruby
#app/controllers/task_controller.rb

before_action :find_task, only: [:edit, :update, :destroy]
```

<p>We keep getting the same message:</p>

```ruby
 Missing template tasks/destroy, application/destroy with {:locale=&gt;[:en], :formats=&gt;[:html], :variants=&gt;[], :handlers=&gt;[:erb, :builder, :raw, :ruby, :coffee, :jbuilder]}. Searched in:
```

<p>So let's add a redirect</p>

```ruby
#app/controllers/task_controller.rb

def destroy
  @task.destroy
  redirect_to root_url
end
```

<p>Our step passes.</p>

<p>Let's write the last step here.</p>

```ruby
Then(/^I should no longer see "(.*?)"$/) do |task|
  !assert page.has_content?(task)
end
```

<p>This is similar to when we were first checking to see if the task would appear on the page when we created a new task. Here we want to test that it doesn't show. So I slightly modified the code with a bang, to assert the opposite.</p>

<p>Cucumber tells us:</p>

```ruby
And I press "Delete" associated with "My First Todo"
```

<p>Nice! We created a Todo app using Cucumber that allows us to create tasks, validates against invalid tasks, mark tasks as complete, and delete tasks. This this type of functionality is pretty common and can be applied in a lot of scenarios.</p>

<p>If you have any questions, or notice anything that can be improved, please let me know!</p>
