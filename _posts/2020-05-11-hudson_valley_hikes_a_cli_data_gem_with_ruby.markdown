---
layout: post
title:      "Hudson Valley Hikes: A CLI Data Gem with Ruby"
date:       2020-05-11 09:26:04 -0400
permalink:  hudson_valley_hikes_a_cli_data_gem_with_ruby
---


Coding isn't easy. I mean, I knew it wouldn't be, but somehow I thought I'd have an easier time with my first project after studying and practicing Ruby for some number of weeks. (Four weeks? Six? Honestly who can say at this point. What day is it?) I had been spending anywhere from two to four hours a day working through the lessons and labs on Learn.co and also supplementing the Learn curriculum with some additional reading (currently: 'The Well-Grounded Rubyist' by David A. Black, which is quite good), but I quickly realized that building something from nothing was . . . well . . . intimdating.

I took a few days off to spend more time with my kids and clear my mind for the challenge. When I finally came back to it, I was . . . well . . . still intimidated. But I cracked on and started to brainstorm. What should I make? What would I like to use? I had a few ideas and wanted to be sure that the websites I'd use to pull data from for my ideas would be scrapable. Guess what? Most weren't—or at least they weren't with the skills I had acquired so far. This sucked up a couple more days. Damn, I thought, I need to make progress, but time kept on moving. Then my family and I went on a hike in New York's Hudson Valley, where we live, and it hit me—I'd make an app that scrapes details and user reviews for the top hiking spots in the Hudson Valley and other nearby areas. I'd call it Hudson Valley Hikes. I got right to work.

Many of the sites I wanted to use weren't easily scrapable, but I landed on TripAdvisor, which seemd to provide me with what I needed. OK, that works, I thought, the rest should be easy!

It wasn't.

I quickly realized I didn't understand some truly basic things, like creating a new gem, customizing the gemspec file, etc. Thankfully, though, Flatiron's Live Project Build videos, with Beth Schofield, were great and really helped me in this area (and others).

The first thing I did after setting up all my files was to make detailed notes about two aspects of the project:
1. Flow
2. Classes and their relationships

I thought about how, from a user's perspective, the app would work and wrote down each step of the user's experience. It wasn't until after I built the app that I created an actual flowchart. I should have made one beforehand, but I actually learned a lot about the flow, and made changes to it, while creating the app. You can see the flow [here](https://drive.google.com/file/d/1P8khATeeQpxftSmHNA-VL5Zo4zFJik8O/view?usp=sharing) if you're interested.

Next I worked out my classes. There would be three:
1. **CLI** - manage the user interface
2. **Hike** - initialize hike objects and store the data for each hike instance
3. **Scraper** - gather hike data (name, location, number of reviews, and select reviews from hikers) from the TripAdvisor website 


# The Scraper Class
I started with the Scraper class to make sure I could scrape appropriately and, after iterating over the relevant HTML items I needed, I assigned each piece of data to a variable that would serve as a parameter in the Hike#initialize method once I built my Hike class. It all worked out well, and I learned some new regex and string methods along the way (the *strip* method is a handy one). One problem that I encountered was that my scraper returned two excerpts of hiker reviews, in quotes, without a space between them. I wanted to keep the quotes, but I was having a hard time adding a space in between them. Eventually, after much trial and error and googling, I sorted it out. You can see my Scraper class below:

```
class HudsonValleyHikes::Scraper

    def self.hudson_valley_hikes
        puts "********** Finding you the top hikes! **********"
        doc = Nokogiri::HTML(open("https://www.tripadvisor.com/Attractions-g2663029-Activities-c61-t87-Hudson_River_Valley_New_York.html"))
        hikes = doc.css("div.listing")
        hikes.each do |hike|
            name = hike.css("h2").text
            location = hike.css("var.parent-name-and-distance span").text.gsub(/\d.*$/, "")
            number_of_reviews = hike.css("span.more a").text.strip
            reviews = hike.css("div.review_snippet").text.gsub('.”“', '.” “')
        HudsonValleyHikes::Hike.new(name, location, number_of_reviews, reviews) 
        end
    end
end
```

# The Hike Class

I knew I needed my Hike class to initialize hike objects with the data pulled from my scrape, so that's where I went next. It was pretty straightforward.

```
class HudsonValleyHikes::Hike

    attr_reader :name, :location, :number_of_reviews, :reviews

    @@all = []

    def initialize(name, location, number_of_reviews, reviews) 
        @name = name
        @location = location
        @number_of_reviews = number_of_reviews
        @reviews = reviews
        @@all << self
    end

    def self.all
        @@all
    end
end
```

Each hike instance is instantiated with a name, a location, a number of reviews, and reviews (from hikers)—although later I noted that not all top hikes on the TripAdvisor site had data for the @reviews instance variable. More on that, and how I handled it, below.

At this point, though, I was feeling good because my hike objects were coming alive!

![](https://media.giphy.com/media/d3Kq5w84bzlBLVDO/giphy.gif)


# The CLI Class

OK, sow how would the app actually work? What would happen first? What would the user see? What options would the user have to select from? What would happen if the user mistyped something in their selection? How would the user exit the app? These are questions I pondered as I began working on the CLI class. I thought about the flow of the app and defined a .call method in which I listed the methods I knew I'd need to build. I named the names as simply but descriptively as possible, so hopefully you can get an idea of the flow just from seeing the method calls within the .call method:

```
    def call
        render_ascii_art
        greet_user
        get_user_interest
        HudsonValleyHikes::Scraper.hudson_valley_hikes
        list_hikes
        select_hike
        select_another_hike
        goodbye
    end
```

Hudson Valley Hikes works as follows:
* First, the program puts some ASCII art. (This was the last thing I added—see next section). T
* Then, it greets the user and asks if the user would like to either A) go on a hike or B) exit the app.
* Next, it grabs the user's input to the hike-or-exit question.
* Depending on the user's input, the program then either A) scrapes the web for the top hiking spots and puts them in a numbered list, or B) says goodby and exits.
* Once a user sees the list of hikes, the program advises the user to enter a number to learn more about one of the hikes or enter 'e' to exit.
* If the user enters a number, the user is then presented with the scraped data stored for each hike object. This incudes the hike name, location, number of reviews, and (if applicable - see below) some hiker review excerpts. For example, if the user entered '1' to see the first hike, they would then see:

```
Name: The Wallkill Valley Rail Trail

Location: New Paltz, NY

Number of Reviews: 94 reviews

What Hikers Say:“If you like trails it's a butifull trail we our whole family loves to go hiking and biking and enjoy the nature so it's a place everyone would like it the nice scenery and it's a quiet place too so sure to go walk...” “Lovely place to hike alone with family or bike have picnics stop and look at what nature has to offer I would recommend ot to all my friends and family”
```

(On TripAdvisor, a few of the hikes didn't have review excerpts available. This was frustrating, but rather than cut those hikes out entirely, I included them without that data so that the user can still see the name, location, and number of reviews. I struggled with this for a while because I assumed that if the scraper wasn't grabbing any data for this, then my @reviews instance variable for these objects would return nil. Lo and behold, though, that wasn't the case. They returned an empty string. Thanks, binding.pry!) 

* The user could then choose to see the list of hikes again in order to learn more about the other hikes, or the user could exit the program.
* When the user finally exits the program, the program thanks the user for visiting Hudson Valley Hikes.

Here's the full code for the CLI class:

```
require 'pry'

class HudsonValleyHikes::CLI
    
    def call
        render_ascii_art
        greet_user
        get_user_interest
        HudsonValleyHikes::Scraper.hudson_valley_hikes
        list_hikes
        select_hike
        select_another_hike
        goodbye
    end

    def render_ascii_art
        banner = File.read("./lib/hudson_valley_hikes/art.txt")
        puts ""
        puts banner
        puts ""
    end

    def greet_user
        puts "\nWelcome to Hudson Valley Hikes! Are you interested in taking a hike today?"
        puts "\nEnter 'y' to see the region's top hiking spots, or enter 'e' to exit."
    end

    def get_user_interest
        input = gets.strip.downcase 
        if input == "y" || input == "yes" 
            list_hikes
        elsif input == "e" || input == "exit" 
            goodbye
        else
            puts "Sorry, I didn't understand that. Please enter 'y' for yes or 'e' for exit."
            get_user_interest
        end
    end

    def list_hikes
        puts ""
        HudsonValleyHikes::Hike.all.each.with_index(1) do |hike, index|
            puts "#{index}. #{hike.name}"
        end
    end

    def select_hike
        puts "\nIf you'd like to know more about one of these hikes, please enter a number. If you'd rather not, enter 'e' to exit the program."
        input = gets.strip
        until input.to_i.between?(1, HudsonValleyHikes::Hike.all.length)
            if input == "e" || input == "exit"
                goodbye
            else
                puts "Sorry, I don't understand."
                select_hike
            end
        end
        index = input.to_i - 1
        hikes = HudsonValleyHikes::Hike.all
        if hikes[index].reviews == ""
            puts "\nName: #{hikes[index].name}\n\nLocation: #{hikes[index].location}\n\nNumber of Reviews: #{hikes[index].number_of_reviews}"
        else
            puts "\nName: #{hikes[index].name}\n\nLocation: #{hikes[index].location}\n\nNumber of Reviews: #{hikes[index].number_of_reviews}\n\nWhat Hikers Say:#{hikes[index].reviews}" 
        end
        select_another_hike
    end

    def select_another_hike
        puts "\nWould you like to see the list of the Hudson Valley's top hikes again? Enter 'y' for yes or 'e' to exit."
        input = gets.strip.downcase
        case 
        when input == "y" || input == "yes" 
            list_hikes
            select_hike
        when input == "e" || input == "exit"
            goodbye
        end
    end

    def goodbye
        puts "Thanks for visiting Hudson Valley Hikes!"
        exit!
    end
end
```
# For Fun
Lastly, I thought I little ASCII art might jazz up the interface when a user starts the app. I went with this:
```
          /\
         /**\
        /****\   /\
       /      \ /**\
      /  /\    /    \        /\    /\  /\      /\            /\/\/\  /\
     /  /  \  /      \      /  \/\/  \/  \  /\/  \/\  /\  /\/ / /  \/  \
    /  /    \/ /\     \    /    \ \  /    \/ /   /  \/  \/  \  /    \   \
   /  /      \/  \/\   \  /      \          /   /    \
__/__/_______/___/__\___\__________________________________________________

                  _                              _ _                     _ _             
  /\  /\_   _  __| |___  ___  _ __   /\   /\__ _| | | ___ _   _    /\  /(_) | _____  ___ 
 / /_/ / | | |/ _` / __|/ _ \| '_ \  \ \ / / _` | | |/ _ \ | | |  / /_/ / | |/ / _ \/ __|
/ __  /| |_| | (_| \__ \ (_) | | | |  \ V / (_| | | |  __/ |_| | / __  /| |   <  __/\__ \
\/ /_/  \__,_|\__,_|___/\___/|_| |_|   \_/ \__,_|_|_|\___|\__, | \/ /_/ |_|_|\_\___||___/
                                                          |___/                          
```

# Final Thoughts
I learned a ton throughout this process, and after I got over the initial hurdle of narrowing my focus and finding a site I could scrape, I made progress pretty quickly. The act of creating this also really solidified a number of object-oriented Ruby concepts for me. I'm really looking forward to building on what I've learned so far. Now, who wants to go on a hike? :)

![](https://media.giphy.com/media/3oxRmGNqKwCzJ0AwPC/giphy.gif)
