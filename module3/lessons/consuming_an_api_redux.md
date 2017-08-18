---
layout: page
title: Consuming an API Redux
length: 180
tags: apis, rails, faraday
---

## Consuming an API

### Summary

What we are going to be working on today is creating an app that reaches out
and consumes data from an external API, and then displays and formats that
data on a web page. The API we will be using is the ProPublica API, and we will
be using it to grab a list of Representatives from Congress.


We will accomplish that by starting with a user story.


```
As a user
When I visit "/"
And I select "Colorado" from the dropdown
And I click on "Locate Members from the House"
Then my path should be "/search" with "state=CO" in the parameters
And I should see a message "7 Results"
And I should see a list of 7 the members of the house for Colorado
And they should be ordered by seniority from most to least
And I should see a name, role, party, and district for each member
```

As you can see, it lines out all that we will do. Let's get started.


### Setup

We start by spinning up our rails app. We are going to call it House Salad.

Because we're getting information about the House of Representatives and we're
gonna toss it around. Kind of.

```sh
$ rails new house-salad -T -d postgresql
$ cd house-salad
```

Great, now we should get in the gems we want. Add this to your `Gemfile`.

```
gem 'faraday'
gem 'bootstrap-sass'

group :development, :test do
  gem 'pry'
  gem 'rspec-rails'
end
```

Now, you'll want to bundle to install the gems you've added, install rspec,
and create your databases.

```sh
$ bundle
$ rails s rspec:install
$ rails db:create
$ rails db:migrate
```

Yes, we haven't created any migrations, but running rails db:migrate
will generate the `schema.rb` so we don't get an error when we start running
tests.

### Testing, All Day, Every Day

So we've got the basic setup.

Let's create our first test files.

```sh
$ mkdir spec/features/
$ touch spec/features/user_can_search_by_state_spec.rb
```

Now let's open up that file and translate our user story into a test.

```
require 'rails_helper'


feature "user can search for house members" do

  scenario "user submits valid state name" do
    # As a user
    # When I visit "/"
    visit '/'

    select "Colorado", from: :state
    # And I select "Colorado" from the dropdown
    click_on "Locate Members of the House"
    # And I click on "Locate Members from the House"
    expect(current_path).to eq(search_path)
    # Then my path should be "/search" with "state=CO" in the parameters
    expect(page).to have_content("7 Results")
    # And I should see a message "7 Results"
    expect(page).to have_css(".member", count: 7)
    # And I should see a list of 7 the members of the house for Colorado

    within(first(".member")) do
      expect(page).to have_css(".name")
      expect(page).to have_css(".role")
      expect(page).to have_css(".party")
      expect(page).to have_css(".district")
    end
    # And they should be ordered by seniority from most to least
    # And I should see a name, role, party, and district for each member

  end
end
```

This test should look pretty straightforward.

And so we run our tests. We should get an error concerning our routes.

Let's edit our `routes.rb` and add a root route that goes to the index action
in a welcome controller.

```
  root "welcome#index"
```

Running the test again, it's not going to be happy that we don't have a
welcome controller. Let's make one.

```
$ rails g controller welcome
```

Once we do that, let's add the index action straight away.

```
class WelcomeController < ApplicationController
  def index
  end
end
```

Now it's going to complain about not finding the template, so we have some
work to do.

First step, we are going to rename `app/assets/stylesheets.css` to
`app/assets/stylesheets.scss` and we are going to delete its contents to add
this:

```
@import "bootstrap-sprockets";
@import "bootstrap";

.search-field { margin-top: 7px;}
```

Now, if we remember the user story, the application is expecting us to select
Colorado out of a drop box. We can safely imply that we need to have all of
the other states in this drop box as well. We don't want this to go in the
view, so we can take advange of the application helper.

Add this to `app/helpers/application_helper.rb`

```
def us_states
    [
      ['Alabama', 'AL'],
      ['Alaska', 'AK'],
      ['Arizona', 'AZ'],
      ['Arkansas', 'AR'],
      ['California', 'CA'],
      ['Colorado', 'CO'],
      ['Connecticut', 'CT'],
      ['Delaware', 'DE'],
      ['District of Columbia', 'DC'],
      ['Florida', 'FL'],
      ['Georgia', 'GA'],
      ['Hawaii', 'HI'],
      ['Idaho', 'ID'],
      ['Illinois', 'IL'],
      ['Indiana', 'IN'],
      ['Iowa', 'IA'],
      ['Kansas', 'KS'],
      ['Kentucky', 'KY'],
      ['Louisiana', 'LA'],
      ['Maine', 'ME'],
      ['Maryland', 'MD'],
      ['Massachusetts', 'MA'],
      ['Michigan', 'MI'],
      ['Minnesota', 'MN'],
      ['Mississippi', 'MS'],
      ['Missouri', 'MO'],
      ['Montana', 'MT'],
      ['Nebraska', 'NE'],
      ['Nevada', 'NV'],
      ['New Hampshire', 'NH'],
      ['New Jersey', 'NJ'],
      ['New Mexico', 'NM'],
      ['New York', 'NY'],
      ['North Carolina', 'NC'],
      ['North Dakota', 'ND'],
      ['Ohio', 'OH'],
      ['Oklahoma', 'OK'],
      ['Oregon', 'OR'],
      ['Pennsylvania', 'PA'],
      ['Puerto Rico', 'PR'],
      ['Rhode Island', 'RI'],
      ['South Carolina', 'SC'],
      ['South Dakota', 'SD'],
      ['Tennessee', 'TN'],
      ['Texas', 'TX'],
      ['Utah', 'UT'],
      ['Vermont', 'VT'],
      ['Virginia', 'VA'],
      ['Washington', 'WA'],
      ['West Virginia', 'WV'],
      ['Wisconsin', 'WI'],
      ['Wyoming', 'WY']
    ]
end
```

That gives us a nice array of arrays of the states and their abbreviations.

Because we placed this in the application_helper.rb file, we now have access
to `us_states` anywhere in our app.

Now that we have this, we are going to edit our
app/views/layouts/application.html.erb.

```
<!DOCTYPE html>
<html>
  <head>
    <title>HouseSalad</title>
    <%= csrf_meta_tags %>

    <%= stylesheet_link_tag    'application', media: 'all' %>
    <%= javascript_include_tag 'application' %>
  </head>

  <body>
    <nav class="navbar navbar-default" role="navigation">
      <div class="navbar-header">
        <button class="navbar-toggle" data-target="#bs-example-navbar-collapse-1" data-toggle="collapse" type="button">
          <span class="sr-only">Toggle navigation</span>
          <span class="icon-bar"></span>
          <span class="icon-bar"></span>
          <span class="icon-bar"></span>
        </button>
        <%= link_to "HouseSalad", root_path, class: "navbar-brand" %>
      </div>
      <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
        <ul class="nav navbar-nav">
          <%= form_tag :search, method: :get, class: "form-inline" do %>
            <div class="form-group search-field">
              <%= select_tag :state, options_for_select(us_states) %>
              <%= submit_tag "Locate Members of the House", class: "btn btn-primary" %>
            </div>
          <% end %>
        </ul>
      </div>
    </nav>

    <%= yield %>
  </body>
</html>
```

The idea is that the navigation is going to be in this nav bar which will
persist across all of the views, so after viewing the Representatives
of a single state, we will be able to immediately go to another state.

So now we need to run our test once again. And it's going to freak out. Our
form wont be happy about where we are trying to send information. So we are
going to have to add a route.

```
get "/search", to: "search#index"
```

And of course, we need to generate a controller and add to it an index action

```
# app/controllers/search_controller.rb
class SearchController < ApplicationController
  def index
  end
end

```

When we run the tests now, we're not getting the results on the page we are
expecting.

It is only seeing our navbar, so let's first get ourselves a search/index view.

```
# app/views/search/index.html.erb

<%= @members.count %> Results
<% @members.each do |member| %>
<ul class="member">
  <li class="name"><%= member.name %></li>
  <li class="role"><%= member.role %></li>
  <li class="party"><%= member.party %></li>
  <li class="district"><%= member.district %></li>
</ul>
<% end %>

```

Now we have everything set up except we aren't creating a @members array
to send to our view.

We need to do some work in our search controller. Since we are working with
the index action, in this action we should contact the API, and get the
information we are looking for.

The following isn't going to be pretty, but we will be refactoring later.