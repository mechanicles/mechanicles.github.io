---
layout: post
title:  "Gotchas with Rails System Testing"
---

I’m not so familiar with Rails system testing, but recently we started to write system tests in our app. We do write model
tests and controller tests which cover most of the business logic. Our app started growing a lot, and we found 
in some pages there were silly mistakes we had in our app which were related to JavaScript/Ajax, and which indirectly 
affected some of our users.

Then our team decided to write down system tests for the pages where we use JavaScript/Ajax a lot.

We followed the [Rails documentation](http://guides.rubyonrails.org/testing.html#system-testing) for system testing and
added all required setup for it. We added simple system tests for those pages, and it helped to trace JavaScript
related issues. Bingo!

As we use [Turbolinks](https://github.com/turbolinks/turbolinks) in our Rails app which intercepts 
all clicks on `<a href>` links to the same domain, i.e., it converts every link into an Ajax request, but this 
caused some serious issues in our system tests.

Those issues were:

1. System tests didn't wait for Ajax request, so our post Ajax assertions were
   failing.
2. Sometimes we did get this error `Timeout::Error: execution expired`.
3. We were getting `Net::ReadTimeout` issues intermittently.
4. Initial system test was taking too much time to run when Rails starts the server (In our case Puma).

For the **1st issue**, We have followed this excellent [blog post](https://robots.thoughtbot.com/automatically-wait-for-ajax-with-capybara)
which helped us to resolve post Ajax assertions checking. We also have modified our flow for system testing, we have created 
a separate private method for user sign in. Our main sign-in page uses Ajax request in the background because of the 
Turbolinks. We created a method called `user_sign_in`, and we call it before every system test where current user context is
required.

```ruby
  def setup
    @john = users(:john)
  end

  def teardown
    sign_out @john
  end

  def test_some_system_test
    user_sign_in
    ...
  end

  private

    def user_sign_in
      visit root_url # sign in url
      assert_selector("div", text: "Log-in to your account")

      fill_in("user[email]", with: @john.email)
      fill_in("user[password]", with: "welcome1")

      click_on "Sign in"

      wait_for_ajax
    end
```

In the `user_sign_in` method, we have called `wait_for_ajax`
helper method. This method we have defined in the file called `test/support/wait_for_ajax.rb`. For more info about this
`wait_for_ajax` method, please follow this [blog post](https://robots.thoughtbot.com/automatically-wait-for-ajax-with-capybara).

```ruby
  module WaitForAjax

    def wait_for_ajax
      Timeout.timeout(Capybara.default_max_wait_time) do
        loop until finished_all_ajax_requests?
      end
    end

    def finished_all_ajax_requests?
      page.evaluate_script('jQuery.active').zero?
    end

  end
```

Include this file in `test/application_system_test_case.rb` file and then you are
done. This flow also fixed our **2nd issue**.

For the **3rd** and **4th issue**, I have added this gem [minitest-retry](https://github.com/y-yagi/minitest-retry) in
Gemfile and called this gem in `test/application_system_test_case.rb` file.

```ruby
require "test_helper"
require "selenium/webdriver"
require 'minitest/retry'
Minitest::Retry.use!

class ApplicationSystemTestCase < ActionDispatch::SystemTestCase

  include Devise::Test::IntegrationHelpers
  include WaitForAjax

  driven_by :selenium, using: :chrome, screen_size: [1200, 800]
end

```

By adding all these changes. Our system tests are running fine without any
issues. System tests speed also got improved.

Please let me know if you have good ideas for these fixes. It would be helpful for our project as well as for
Rails communities.

Happy Hacking :)
