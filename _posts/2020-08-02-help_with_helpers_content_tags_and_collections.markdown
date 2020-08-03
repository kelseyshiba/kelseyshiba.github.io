---
layout: post
title:      "HELP with Helpers, Content Tags, & Collections"
date:       2020-08-03 00:23:16 +0000
permalink:  help_with_helpers_content_tags_and_collections
---

## A Rails Project Reflection

Trying to find one place for all of this information seemed really difficult. I tried blogs and google and Rails documentation, but none of them seemed to lay certain things out in a way that made it clear what all of these new things were doing in my Rails application. 

I learned so much from completing this project. I feel lucky to have made some progress, and I hope this will help future programmers with their projects as well. 

I think one of the most challenging things was trying to understand helpers and content tags within those helpers. The look of content tags is pleasing to the eye when used over a ton of HTML inside the view, and if those are further abstracted from the view and made into helper methods, the views are able to be more direct and clear about what they are trying to do, especially when collaborating.

## Images
There is a wonderful resource I used when adding images to my project.  I would like to show how it abstracts the view by showing the full HTML code and moving backwards from there.

**HTML**

```<img src="app/assets/google-logo.jpg" alt="fb-logo" class="card-img-top">
```

**ERB (View File)**
```<img src="<%= asset_path 'fb-logo.png'%>" alt="fb-logo" class="card-img-top">
```

**Helper Abstracted**
```   def show_google_img
```     image_tag(asset_path('google-logo.jpg'), {class: 'card-img-top'})
 ```  end
 
** New ERB (View File)**
 ```<%= show_google_img %>
 ```
 
 Here is the extracted output for each:
 
image_tag 
```<img class="card-img-top" src="/assets/google-logo-```8ccc923de01fa9a0668090cb43bf4a3b907d2d2a27b57793189fd0c32536b8d6.jpg">

html img src with asset_path
```<img src="/assets/google-logo-8ccc923de01fa9a0668090cb43bf4a3b907d2d2a27b57793189fd0c32536b8d6.jpg" ```alt="google-logo" class="card-img-top">

html
```<img src="/app/assets/images/google-logo.jpb" alt="google-logo" class="card-img-top">


All Three in the View Together
     ```1.<%= show_google_img %>
     ```2.<img src="<%= asset_path 'google-logo.jpg'%>" alt='google-logo' class='card-img-top'>
     ```3.<img src="/app/assets/images/google-logo.jpg" alt='google-logo' class='card-img-top'>
		 ```

You can see that #1 makes things very skinny and clear for the view page!



## Buttons and Links
I used a lot of buttons throughout my Rails project. I often wondered how much work it would be to make sure the buttons and links were both routing to the correct places. I also needed several links or buttons in one place to make sure only the admin could see some of them.  

In order to keep the GET request, I made some of the links into content tags, and used Bootstrap classes to make them into buttons so they would look nice to the user.  

Here is the code:
```content_tag(:a, 'All Gigs', {href: gigs_path, class: 'btn btn-dark'})
```

**Tips**
I was often missing the comma after the first argument (":a") and that would throw an error that was difficult to figure out. Also, passing the options as a hash, sometimes two separate hashes became a good thing to try. As you can see above-- the href and class keys are passed in the same hash with a comma separating them. 

Here's what they look like:

Blog1


**Delete**

Beware doing a content_tag(:a, ...) for the delete buttons. It sends a GET request even though they appeared to be a button through the class styling (btn btn-dark), but they did not send the POST to the delete method. Most of them needed to be re-written like so:

```
button_to('Delete User', user_path(user), method: :delete, data: {confirm: 'Delete This Musician?'}, class: 'btn btn-danger') + tag(:br)
```

The other cool part was including the "data: {confirm: 'Delete'}" which created an additional confirmation pop-up box before a user was allowed to fully delete an item. 

Blog3 Photo

## Many Links
There is probably a better way to do this, but in order to display many links inside of a helper method, I used content tags and "tag(:br) + ". This added a <br> between each of the links and the "+" sign allowed all of them to show. Without the "+" sign, it would only read the last one inside of the method. 


```def show_admin_links(current_user)
       if current_user.admin
			      content_tag(:a, 'All Invoices', {href: invoices_path, class: 'btn btn-dark float-left'}) +
            content_tag(:a, 'Create New Invoice', {href: new_invoice_path, class: 'btn btn-dark float-right'}) + tag(:br) +
            content_tag(:a, 'All Gigs', {href: gigs_path, class: 'btn btn-light'}) +
            content_tag(:a, 'My Gigs', {href: user_gigs_path(current_user), class: 'btn btn-light'}) + tag(:br)
			 end
	end
```

This allowed me to display these buttons solely if the user was an administrator.

**Content Tag Advantage**
I think one of the main reasons I love content tag helpers is because you can use your path helpers inside of them (i.e. gigs_path, user_gigs_path, etc.) One I got to know these paths, it was easy for my brain to recall that helper instead of the full URL. 

## Nesting Tags
I found some great code to help me display errors from my controllers, which I used flash[:notice] for. In my views, the only code now needed was this:

```
<%= display_flash_notice(flash[:notice]) %>
```

In my helper method, this is the code using content_tags that were nested:

```
   def show_flash_notice(flash)
         if flash
             content_tag(:div, class: "alert alert-danger alert-dismissibile fade show") do
                content_tag(:p, "#{flash}", class: 'text-danger') + 
                content_tag(:button, class: "close", type: "button", data: "alert") do
                end
             end
         end
   end
```

This was the HTML code that was rewritten:

```
<div class="alert alert-danger alert-dismissibile" role="alert">
     <strong><%= flash[:error] %></strong>
         <button type="button" class="close" data-dismiss="alert">
            <span>&times;</span>
         </button>
	</div>
```

I removed the <span> from the HTML code that generates an "X" to close the box as that is only piece I could not get working.

I think it is arguable that both of these are lengthy code. However, I think it is easier to understand what is happening within the helper method, and I think it is good to be able to know how to do these things if you want to follow the conventions of a programmer or employer that prefers them. I also think this could be refactored to actually nest the content_tags inside of the other content_tags, but this seemed to read clearer to me this way and output the HTML that was desired.

Visual Output:

Blog 2

## The Beloved Collection Select
It took me forever to become more comfortable with any of the different collection select methods. I feel a lot more familiar with how they look now. It was difficult to add classes to some of the different ways I needed to use these form helpers. Here are a few examples using Bootstrap classes.

```
<%= f.collection_select :gig_id, Gig.all, :id, :title, {include_blank: 'Please Choose the Title'}, {class: 'dropdown-item'} %><br>
```

Notice how this particular method needed the two (2) separated hashes in order to render correctly. 

Visual Output

Blog4



I also wanted only three (3) choices for one of the parameters on a gig for its booking status. Here's how I handled that and the styling inside the form helper:

```
<%= f.select :booking_status, options_for_select([['Pending'], ['Complete'], ['Deleted']]), {include_blank: 'Please Select'}, {class: 'dropdown-item'} %>
```

Visual Output

Blog5


## Summary

I have a lot to learn and this project was extremely difficult and an incredible learning experience. Please submit any comments or adjustments to developerkelseyshiba@gmail.com. Thank you!

