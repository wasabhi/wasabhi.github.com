---
layout: post
title: "Customised redirect after sign in with devise"
date: 2011-12-15 22:44
comments: true
categories: 
- rails
- devise
---

Devise is a very popular Ruby gem used in Rails for authenticating access, and allowing Rails applications to quickly set up common access behaviours like, "Forgot Password?".

It took me a little while to find out how to redirect a devise user to a specific path after signing in, or registering (signing up) for an account.

The point of the exercise, is to be able to create a your own registration flow which doesn't correspond to the usual/normal approach adopted by devise.

Typically, a registration flow in devise is:

* sign up with an email address
* you're sent a confirmation email, and cannot progress until you:
* follow the confirm link in the email, where you are prompted to:
* set a password and then you're in!

I wanted to create a different series of steps

* sign up with an email address
* you're sent a confirmation email, and then:
* in parallel progress to other steps needed to subscribe for a service (eg payment)
* you can follow the confirm link in the email at any time

So, first we should write a test:

``` ruby rspec registrations controller test
    before do
      @request.env['devise.mapping'] = Devise.mappings[:user]
    end

    describe "when I register" do
      it "should redirect to a payment page" do
        post :create, :user => { :email => 'some@email.com' }
        response.should redirect_to(payment_path)
      end
    end
```

The following block achieves a simple yet powerful thing. It loads up the devise
scope for the user into the request, allowing routes and devise modules to be
available for the test to run properly.  Without this, the devise routes are not
available, and the test fails with a message asking whether a devise_scope for
the user has been set at all in the routes.

``` ruby load the devise scope for the user
    before do
      @request.env['devise.mapping'] = Devise.mappings[:user]
    end
```

The remainder of the test posts to the create action of the registrations
controller, after which assert that the response should be a redirect to the
required payment path.

To make the test pass:

``` ruby overide a specific method in the registrations controller
class RegistrationsController < Devise::RegistrationsController

  protected

  def after_inactive_sign_up_path_for(resource)
    payment_path
  end 
end
```

This method is useful for redirecting users who have registered, and have not
yet confirmed their account.

Once they have confirmed their account, should you choose to redirect them to a
specific path, then again write this test:

``` ruby rspec confirmations controller test
    before do
      @request.env['devise.mapping'] = Devise.mappings[:user]
      @user = User.create! Factory.attributes_for(:user)
    end

    describe "when I confirm my account" do
      it "should redirect to a status page" do
        post :confirm_account, 
             :user => { :confirmation_token => @user.confirmation_token }
        response.should redirect_to(status_path)
      end
    end
```

I'm using FactoryGirl above to create valid attributes for a user.  I set up the
devise scope just as before, and then I post the confirmation token to the
confirm_account action of the confirmations controller asserting that this
should redirect the user to a status_path.

To achieve this I modify the confirmations controller thus:

``` ruby confirmations controller
class ConfirmationsController < Devise::ConfirmationsController

  [ ... some methods removed ... ]

  protected
  def after_sign_in_path_for(resource)
    status_path
  end 
end
```

