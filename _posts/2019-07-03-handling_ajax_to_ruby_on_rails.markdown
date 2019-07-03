---
layout: post
title:      "Handling AJAX inside of Ruby on Rails"
date:       2019-07-03 00:44:01 -0400
permalink:  handling_ajax_to_ruby_on_rails
---


First and foremost there needs to be a form to submit to the controller that we can then "hijack" with AJAX.

Within our ListsController we have our New Route 
`    def new
        @list = List.new
    end`
		
Next we will have our form:

`<h4>Create New List</h4>
<%= form_for [List.new] do |f|%>
  <%= f.label :title %>
  <%= f.text_field :title %>
  <%= f.submit 'Create List' %>
<% end %>
`

Next we will use a `<script>` tag to put AJAX on our page. 

`<script type="text/javascript" charset="utf-8">
$(document).on('turbolinks:load', function(){
  $('#new_list').submit(function(event) {
    event.preventDefault();
    
    let values = $(this).serialize();
  
    let posting = $.post('/lists', values)
    posting.done(function(data){
      $('#list_title').text(data['title'])
      $('#list_created_at').text(data['created_at'])
    });
  });
});
</script>
`

Since we have the turbolinks gem in our Gemfile we use `turbolinks:load` instead of `$(document).ready`. Next grab the id attribute of the form for the New List. Once the form is submitted we prevent the "event" of the submission from happening, thus the "hijacking" begins! 

Now we use the AJAX `serialize()` method on `this` which is the form itself. The `serialize()` method serializes the successful controls within the form, in this case the input from the `f.text_field :title` which is used to fill the `title` for the new List. We get this as a result:

`"utf8=%E2%9C%93&authenticity_token=HGTmkvFyTJR48JcNHY7357MW1oCMVSHRmiHD7Rk83Eh8dHuYl4AZJubMjeyrTWsubbeW2f3NWgwcBFN0%2F9gQsg%3D%3D&list%5Btitle%5D=Bootcamp"`

Next we send this to our Lists#create route to handle the POST request with our values.

To be able to handle this response we add `render json:@list, status:201` at the end of our Create route, the status:201 is to show that the request is successful as well as created. We render JSON to be able to handle the response of the AJAX request with the proper format, since html would send back the whole Create Page.

    `def create
        @list = List.find_or_create_by(list_params)
        @list.user_id = current_user.id
        if !current_user.lists.include?(@list) 
            current_user.lists << @list && @list.save
            render json: @list, status:201
        else
            redirect_to user_path(current_user)
        end
    end`
		
Upon the request being `.done()` we select the values we want out of our `data` argument

`data = {id: 18, title: "Happiness", created_at: "2019-07-03T04:36:55.349Z"}`

Then we add the results onto our page.

`<text id='list_created_at'></text>
<text id='list_title'></text>
`

This will populate the innerHTML of both `<text>` tags. Creating a new Asynchronous List on the page! 

And There you have it! 

