---
layout: post
title:  "NYC Today, My First CLI Gem "
date:   2017-01-24 21:45:03 +0000
---

![Welcome Screen](http://i.imgur.com/TZrkkXX.png?1)

It's about time I got back to this blog! It's been a while, but I've been busy in the meantime, not least of all recently with work on the topic of this post: my first Ruby data gem, a CLI app I call [NYC Today](https://github.com/vicision/nyc-today-cli-app). If you'd like to check it out yourself, enter `gem install nyc_today` in your terminal to install it, and then enter `nyc_today` to run the gem.

This is the first of the portfolio projects for the Learn.co program, and requires scraping a public website with the Nokogiri gem and using the gathered data to create Ruby objects. A command line interface (CLI) menu is also needed to interact with these objects, and it has to go at least one level deep by allowing a choice from a menu and then displaying more information about that choice. As a longtime fan of live music who lives in the NYC area, undoubtedly one of the best cities in the world for live music, I thought that an app listing today's concerts and their info would be a fun, ideal project for me.

My first choice of site to scrape for these listings was [OhMyRockness](http://www.ohmyrockness.com/), a website that primarily focuses on independent music in NYC. I started this project by watching and/or participating in some of Learn.co's code-alongs and lectures relating to CLI gems and studying some projects made by students in the past, getting a feel for how I'd need to build my own and using Bundler (following the steps [here](http://railscasts.com/episodes/245-new-gem-with-bundler?view=asciicast)) to lay the groundwork. Once I felt like I had a solid foundation, with an `NycToday::Scraper` class, an `NycToday::Event` class and an `NycToday::CLI` class for interacting with the scraper and event objects, I started coding the methods I'd need for scraping the site. And that's where I ran into my first roadblock.

I inspected the code for the site, made a note of the CSS selectors I would need to use in my scraper to get the data I wanted, and built the scraper accordingly. However, when I tested it out, it returned nothing. Being relatively new to scraping, I assumed I was doing something wrong. After all, the data was all right there in the code for me to see. So I tweaked and tested, tweaked and tested over and over again, but nothing I did seemed to make a difference. I had my suspicions, and after speaking with an instructor they were confirmed--it turned out that this site used AJAX to load its content after the page itself, so I was going to be unable to scrape it using Nokogiri. 

So what to do now? I was pretty committed to my show listing concept at this point and had built some of the basics of the program already so I was hesitant to scrap anything. So, I went looking for another site that I could use in the same way. After some searching, I found out that [Brooklyn Vegan](http://nyc-shows.brooklynvegan.com/events/today), the long-running music blog, has its own event listing section. It ended up being pretty comprehensive, and better yet, it didn't use AJAX! Now we were cooking. 

As I started looking over the Brooklyn Vegan [event listings](http://), it quickly became apparent that I was going to be dealing with a lot of events, of a lot of different types, and on a lot of different pages. This was not going to be quite as simple as I'd hoped to begin with, but I felt up to the challenge. It might even be fun!

And it was fun, though it was also difficult--but that just made finding a solution to the issues I was facing that much more rewarding. It's a great feeling to build something actually useful and usable by yourself from the ground up. So here's how it works, and some of the obstacles I ran into along the way.

The first challenge was the sheer number of pages in the listing. Not only were there 25 events on every page, there were several pages. I decided the best way to handle this was by looping and scraping each page in succession. Here's what I did.

First, I set a constant to the main URL for the event listings and another to an empty array that I'd be filling with the scraped pages: 

```
  @@main_url = "http://nyc-shows.brooklynvegan.com"
  @@pages = []
```

Next, I built a method to scrape the pages and turn them into a set of Nokogiri nodes that I'd be able to parse and gather event attributes from. 

```
  def self.get_pages
    num = 1
    while num <= 10
      page_url = @@main_url + "/events/today?page=#{num}"
      page = Nokogiri::HTML(open(page_url))
      @@pages << page
      num += 1
    end
  end
```

I opted for the simplest solution to the number of event pages--I checked how many pages of events there were on the busiest nights of the week, namely Friday and Saturday. Neither had more than 10 pages of events at most, so I coded the loop to start at page 1 and stop at page 10. Even on New Year's Eve (which also happened to be a Saturday) the daily event listing never exceeded 10 pages, so I felt that this was a safe, if inelegant, solution. If I were to decide to update the gem in the future I would probably try to code so that it checks the pages for events and stops the loop at the point which they end, but for now this works great. 

Since the URLs of the event listing pages included a page number, I was able to interpolate the loop counter `num` into the URL ending the pages shared in common, add this to the `@@main_url`, scrape with Nokogiri, and push the resulting `page` into the `pages` array. 

Now I needed to create my event instances and assign their attributes using CSS selectors. Here's a simplification of how I did that:

```
  def self.scrape_events
    get_pages
    @@pages.each do |page|
      page.css(".event-card").each do |event|
			event_hash = {}
        url_end = event.css("a").attribute("href").value
				event_hash[:name] = event.css(".ds-listing-event-title-text").text
				event_hash[:event_link] = @@main_url + url_end
				event_hash[:time] = event.css(".dtstart").text
				event_hash[:venue] = event.css(".ds-venue-name").text
				event_hash[:event_type] = event.attr("class")
        NycToday::Event.new(event_hash)
      end
    end
  end

```

As you can see, the `.scrape_events` method first calls the `.get_pages` method described above, then iterates over each page, creating an `event_hash` to which it assigns the various keys and values that become the event attributes when each event instance is initialized at the end, using the following `#initialize` method, a constant `@@all` to collect the event instances, and an `attr_accessor` for each of the event attributes: 

```
  attr_accessor :name, :venue, :time, :time_stamp, :event_link, :event_info, :event_type, :price
	
  @@all = []

  def initialize(hash)
    hash.each do |key, value|
      self.send("#{key}=", value)
    end
    @@all << self
  end
```

(Thanks to Learn instructor Luke Ghenco and [this mob programming session](https://www.youtube.com/watch?v=2CahNsC5xCs) for showing me this process, by the way! A good explanation of the `send` method can be found [here](https://teamtreehouse.com/community/what-does-the-ruby-send-method-do).)

So, now that I have all these event instances, what next? We need some way to interact with them. In thinking about how I'd want to do that, I realized that just having hundreds of random events listed to read through was not an ideal situation, even thought that's basically how they were listed on the event site. Their sorting criteria apparently is based on popularity or upvotes, but I didn't want to incorporate that into my app. I thought that grouping the events by category and then listing them by time made more logical sense. 

![Menu](http://i.imgur.com/NuUXbtf.png)

When a user inputs a number from the above menu, the following code in the `CLI` class is run:

```
  def choose_type
    input = gets.strip
    if input.to_i > 0 && input.to_i <= NycToday::Event.event_types.count
      @@type_choice = input.to_i-1
    elsif input.downcase == "exit"
      goodbye
    end
	end
```

The number chosen is saved as a constant `@@type_choice`, which is then used by the following two methods:

```
  def page_count
    NycToday::Event.event_sets(@@type_choice).count
  end
```

```
  def category
    NycToday::Event.event_types[@@type_choice]
  end
```

The first method above calls this method in the `Event` class which takes `@@type_choice` as its input, uses it to select from the event types, and then groups all the events of that type in an array:

```
  def self.event_sets(input)
    choice = event_types[input]
    event_group = []
    all.each do |event|
      event.event_type.downcase == choice.downcase ? event_group << event : nil
    end
    @@sets = event_group.sort_by!{|e|e.time_stamp}
		sort_events
  end
```

It then sets a constant `@@sets` to that array, sorted by time, and calls `sort_events`:

```
  def self.sort_events
    midnight_fix.sort_by!{|e|e.time_stamp}
    @@sets = @@sets.push(*midnight_fix).each_slice(5).to_a
  end
```

The method called first above,`midnight_fix`, simply adjusts the events in the array so that events at midnight or later are at the end of the array, which when sorted by the `time_stamp` attribute reads them as `00:00 AM`and so on, and places them before earlier events. It then slices the array into a number of smaller arrays, which the user then sees displayed as pages in the interface, as below. 

![Event List](http://i.imgur.com/HU8rrhg.png)

When the user inputs a number for an event, the CLI runs the following as part of the `selection` method:

```
      event = NycToday::Event.sets[@@set_no][input.to_i-1]
      NycToday::Scraper.scrape_event_page(event) unless event.event_info != nil
      more_info(event)
```

This sets the variable `event` to the particular event instance that the user wants to know more about, and then scrapes that event's individual event page for the event's price (if listed) and any further information about the event, such as a description or a bio for the performer, and lists it as seen below using the CLI's `more_info` method. (I had originally coded the events list scraper to call the `scrape_event_page` method for each event at the same time it scraped the list, but that led to an occasionally huge time lag when first launching the program, and I realized there was no need to scrape an event's page until a user selected it, at which point the info there would be assigned to attributes and wouldn't scrape the page if the attributes were already set).

![Event Info](http://i.imgur.com/o8wFzOO.png)

The user is then able to return to the event listing and go forward in the list to later events or back to earlier ones, or return to the main event category menu to look select a different type of event.

And that's pretty much it! It seems so much simpler described here than it actually was in practice. There were numerous hurdles and setbacks throughout the process, and overcoming them and moving on to the next step was a great feeling every time, and a great learning experience. I'm especially happy to have made an app that I'll probably actually use pretty frequently in my day-to-day life. Hopefully others will find it useful too!



















