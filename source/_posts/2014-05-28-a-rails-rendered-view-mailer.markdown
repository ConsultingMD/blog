---
layout: post
title: "A Rails Rendered View Mailer"
date: 2014-05-28 12:15:00 -0700
comments: true
categories:
---

We needed to email admins and product owners reports on certain resources on a regular basis and email an existing view in our admin portal to the responsible parties.We could have created custom emails for each report and duplicated the view logic. Instead we decided we'd figure out how to render in email the views we already had that people saw when they logged in. We wanted to be able to render that view and email it so we came up with the following scheme.

###ViewMailer

Everything in Ruby is a class. Your controllers are classes too.

We wanted to mock the real controller to be able to execute its methods in the context of a request dispatch. We started by creating a new instance of class that inherits from the real controller and includes various action_controller pieces to mock just enough of the dispatch path to be able to render a response without full rendering.
We then create a new global constant, our mocked version of the real controller.

```
def self.create_mocker_class(controller)
    new_class = Class.new(controller) do |klass|
      include ActionController::Rendering
      include ActionController::Redirecting
      include Rails.application.routes.url_helpers
      include  AbstractController::Callbacks
      @@controller = controller

      def content_type
        "text/html"
      end

      def env
        @_env ||= {
          "rack.url_scheme" => "http",
          "rack.input"      => "",
          "rack.session"    => "",
          "REQUEST_METHOD"  => "GET"
        }
      end

      def session
        {}
      end

      def request
        @_request ||= ActionDispatch::Request.new(env)
      end

      def current_user
        @@current_user
      end

      def current_user=(user)
        @@current_user = user
      end

      def params=(given_params)
        @_params = given_params
      end

      def params
        @_params
      end

      helper_method :will_paginate
      def will_paginate(o,x=nil)
        true
      end

      def self.name
        ViewMailer.mock_controller_name(@@controller)
      end
    end
    Object.const_set(mock_controller_name(controller), new_class)
  end
```

lastly we override mail method of the ActionMailer to create an instantiate a mock controller based on the params, set the current_user (if necessary for context) and call the action method on the controller and render the response to a string that gets passed to the super method to do the actual mailing

```
def mail(options, &block)
   params = Rails.application.routes.recognize_path(options[:path], method: (options[:method] || 'GET'))
   mock_controller = self.class.define_or_initialize_by_controller("#{params[:controller].camelize}Controller".constantize)
   mock_controller.params = params
   mock_controller.current_user = options[:current_user]
   mock_controller.send(params[:action])
   default_options = {:content_type => mock_controller.content_type}

   super(default_options.merge(options).merge(:body => mock_controller.render_to_string(params[:action].to_sym)), &block)
 end
```

now we can have nice mailers to email existing views (albeit without styling):

```
class ReportMailer < ViewMailer

  def tps_report
    mail(from: "peter@initech.com", to: "lumbergh@initech.com", path: reports_path(coversheet: true))
  end
end
```

Now, no need to worry about them tps reports not generating in time with a coversheet!

You can get the gem at:

https://rubygems.org/gems/view_mailer

and the source code is at:

https://github.com/ConsultingMD/viewmailer
