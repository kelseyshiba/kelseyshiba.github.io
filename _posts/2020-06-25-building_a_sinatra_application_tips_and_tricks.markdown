---
layout: post
title:      "Building A Sinatra Application Tips & Tricks"
date:       2020-06-25 18:03:45 +0000
permalink:  building_a_sinatra_application_tips_and_tricks
---


I learned so much completing this project, and realized there is so much left to learn. I wanted to document some of the technical things that I struggled with in the hopes that it would help others on their coding journey.

**Things I Struggled With:**
1. Setting up the application
2. Creating New Files
3. Adding/Deleting Columns, Resetting Objects
4. Understanding Relationships
5. Using Two Different Users to Login
6. Using Logic in the Views
7. Protecting your routes

**Setting Up the Application**
File structure really confused me upon the start of this project. I am a visual learner, so I setup this graphic for myself. 
![](https://drive.google.com/file/d/15PsDRSVAH9DT7Stwz81RsU_rSazCCSlX/view?usp=sharing)

https://drive.google.com/file/d/15PsDRSVAH9DT7Stwz81RsU_rSazCCSlX/view?usp=sharing

The biggest piece missing for me during the setup was that I need to include in my config.ru file:

```
use Rack::MethodOverride
```

Also, when and if you decide to use Sinatra Flash, be sure to do the following steps:

```
require 'sinatra/flash' in main controller
register Sinatra::Flash in main controller inside configure do
gem install sinatra-flash in your bash terminal
place gem 'sinatra-flash', '~> 0.3.0' in your Gemfile
```

You can also steal some HTML/Ruby to make your flash look nice above the yield in the layout with something like this:

```
<% flash.keys.each do |type| %>
      <div data-alert class="flash <%= type %> alert-box radius">
      <%= flash[type] %>
      <a href="#" class="close">&times;</a>
      </div>
  <% end %>
	
	add CSS with
	
	.flash {
	     background-color: yourchoice
	}
```

Use flash[:message] = "Whatever message you want at the top of your page"

**Creating New Files**
When creating a new file be sure to follow the routes that it needs in order for it to work before adding more code. 

For example, when adding a new controller, remember to trace it back to make it work. To make a controller, add the use ControllerName in your config.ru file.

When adding a new model, you'll need to trace its relationships, validations, and database. Be sure to add any belongs_to, has_many, has_secure_password to your objects, create the database with rake db:create_migration NAME=name_of_table and add any relevant columns, and add validates: email/name/password, presence: true.

When adding a new view, be sure to add the get request that renders that view, and add 1 line of HTML to be sure you can see the view after rendering. 

```
get '/' do

     erb :index
end

(in the view.erb)
<h1>Hello</h1>
```

**Adding/Deleting Columns**
I had some trouble figuring out whether or not I needed to seed / reseed my database after making changes. As it turns out, that was not necessary most of the time. What worked the best when adding or deleting columns, whether whole columns or just to make minor name changes, this code worked best.  Additionally, adding a reset command to the object that I was making changes to helped immensely.

```
class AddColumnsToAppointments < ActiveRecord::Migration
  def change
    remove_column :appointments, :time
    remove_column :appointments, :date
    add_column :appointments, :start, :datetime
    add_column :appointments, :end, :datetime
    Appointment.reset_column_information
  end
end
```

**Understanding Relationships**
This is a work in progress for me. When I initially setup the project it seemed like it would make sense in the real world for the following to be true:

Teachers would have many appointments, and many students through appointments
Appointments would belong to Students and Teachers
Students would have many appointments, and belong to teachers

When I worked through the true functionality of the application, the students did not need to belong_to teachers, because the teachers had many students through the appointments made and there was no need for the teachers to see the students they had without the need to view their appointments.  This was difficult to grasp but it made sense as that would almost be creating an unneccesary duplicate relationship. 


**Using Two Different Users to Login**
This concept was very hard for me to figure out. If I were giving advice to myself, I would advise myself to write a Users class and have Students and Teachers inherit from that class in order to login/signup.  However, it was an interesting challenge to have to work with.  Here is the helper methods that allowed me to have compelete functionality. 

```
def logged_in?
      !!session[:teacher_id] || session[:student_id]
    end

    def student_current_user
      @student_current_user ||= Student.find_by_id(session[:student_id])
    end

    def teacher_current_user
      @teacher_current_user ||= Teacher.find_by_id(session[:teacher_id])
    end

    def is_teacher?
      !!teacher_current_user
    end

    def is_student?
      !!student_current_user
    end
```

When using session[:user_id] the current_user method was finding both a student and a teacher using find_by_id (4) -- as 4 was the session[:user_id].  Making that distinction solved the issue and allowed me to create specific parts in the views using is_teacher? and is_student? logic. 

I originally solved it by adding the student object/teacher object to the session with session[:class] = student object, but it was advised that I never push an object into the session, which is good advice!

**Using Logic in the Views**

Professor Hansen advised that I take logic out the views, which cleans up the code and helps everyone understand what's happening in the views.  This was great advice and allowed me to rework several things. In the views, I wanted to sort lists of items, so I created methods to do so.  One of the methods I was able to put into the model to sort itself:

```
def self.sort_appointments
        self.all.sort do |a, b|
            a.week_number <=> b.week_number
        end
 end
```

However, there was a similar method that I had to use as a helper method inside of a controller because it needed access to another helper method in order to run.  Here is that method.

```
def teacher_appointments
      Appointment.all.select do |appt|
          appt.teacher_id == teacher_current_user.id
      end.sort do |a,b|
        a.week_number <=> b.week_number
      end
end
```

Both of these were very helpful in getting logic outside of the views. 

**Protecting Routes**
One of the best lines of code I learned from other students in my cohort was this line:

```
not_found do
    status 404
    erb :error
 end
```

This code will render an error page if someone enters a bad url such as:

127.0.0.1:9393/sldkfjalsjfdsla;j

Creating if/else logic to assist with bad data/input was very useful.  A pattern that I picked up, especially from patch or post requests that required creating something was to:

- access the object and set to variable
- check if that params are valid
- check if logged in 
- check the object belongs to the user 

Code sample: 

```
@variable = Variable.find_by_id(params[:id])
 if valid_params?
            if logged_in?
                if is_student? && @appointment.student_id == student_current_user.id
```

This was an incredible project to complete. It helped solidify object orientation, databases, and server knowledge. I'm excited for the next step in learning how to create more dynamic applications.

Code on!

Kelsey
