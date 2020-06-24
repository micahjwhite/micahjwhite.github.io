---
layout: post
title:      "Home Renovation with Sinatra"
date:       2020-06-24 09:22:07 -0400
permalink:  home_renovation_with_sinatra
---


I became a homeowner about four years ago, and in those four years, my wife has suggested at least twenty or thirty home-renovation projects. (We've only done three or four.) It seems every other week, we discuss knocking down a few walls, refinishing some stairs or the attic, or combining a few rooms into a single gargantuan space. Most of the time we laugh about it, but sometimes we reach out to get cost estimates for the projects that we know we want to eventually take on. I thought it would be nice if we had a way to keep track of our ideas (and their ballooning costs), and thus [The Renovator](https://github.com/micahjwhite/renovator)—a Sinatra app based on MVC structure—was born.

<iframe src="https://giphy.com/embed/hiqTKqU40YKI0" width="480" height="360" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/diy-hiqTKqU40YKI0">via GIPHY</a></p>

I used the [Corneal gem](https://github.com/thebrianemory/corneal) to get started, and I'm glad I did. It quickly set up the file structure I needed for this app. From there I created my migrations and tables and then set up the corresponding models (User and Project) and associations (a user has_many projects, and a project belongs_to a user). Then, I went to work on the fun part (that's not to say it wasn't fun up until this point): the controllers and views.

I set up three controllers—Application, Users, and Projects—and the standard ERB views that correspond to either a User or a Proejct—index, new, show, and edit. I also made use of the layout view that Corneal provides, and this was really helpful. Layout.erb allowed me to have a single file for elements that would appear on all pages of the application (e.g. the head metadata, navigation buttons, footer info, etc.). But it's real magic came from a single line of code:

```
<%= yield %>
```

Each HTTP request a user makes while using the app will load the layout file. During the processing of that file, when <%= yield %> is reached, the corresponding view file (based on the request made) will be loaded. Say a user makes a get request to '/', which, in my Application Controller, loads the file index.erb. The layout.erb view will begin loading, but once <%= yield %> is hit, index.erb will load before layout.erb continues, so in your browswer what you see, as a user, is a single page that was rendered by using the HTML from two separate files. Pretty cool!
# So Much Learning
I learned a lot while building this project. A few key learning points that come to mind are:
* If you're not careful, a user who is currently logged in will be able to view and edit another user's data. This plagued me for a bit before I realized that I needed to make quite liberal use of some helper methods in my Application Controller:

```
def logged_in?
      !!session[:user_id]
    end

    def current_user
      User.find(session[:user_id])
    end
```

These methods were really instrumental in getting the app to work correctly so that a user who is logged in doesn't have access to the data of other users. Here's an example from my Projects Controller that was instrumental in restricting data to only the current user of the application:

```
get '/projects/:id' do
        if !logged_in?
            redirect '/login' 
        else
            @project = Project.find(params[:id])
            if @project && @project.user == current_user
                erb :'/projects/show'
            else
                redirect '/login'
            end
        end
    end
```

* Getting flash messages to work took me some time, but I finally got it, and they are great. We see these so much on the web that it's nice to have an understanding of how they work.
* I need *much* more practice with CSS. There's so much to learn there. I kept getting sidetracked by wanting to make my app look nice, but I had to pull myself back from that impulse and remind myself to focus on the project requirements. Along the way, though, I learned a couple of neat CSS tricks, one of which includes the use of @keyframes to animate an element. I used this to make my flash messages increasingly become more transparent over the course of a few seconds until they disappeared entirely. This was fun to learn, and I'm looking forward to become more proficient on the front-end as time goes on.

I had a great time building this project, and it really reinforced my understanding of routes, controller actions, form data, and much more. I'd like to come back to it soon to beautify it a bit more. But for now, it's a functional home-renovation project cms that I can use anytime my wife walks into the room with her sledgehammer.

<iframe src="https://giphy.com/embed/tAeB6dptxnoli" width="480" height="446" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/construction-tAeB6dptxnoli">via GIPHY</a></p>





