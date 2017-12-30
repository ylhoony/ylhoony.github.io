---
layout: post
title:      "CLI Gem Project"
date:       2017-12-28 12:57:46 -0500
permalink:  cli_gem_project
---


For CLI gem project, I created Software Binder. With this, you can search the available softwares on the market under the certain category. I used [Capterra](https://www.capterra.com/) for the external data. Below was the steps that I briefly wrote down for planning. It was shorter at the beginning, and I started to break down with more details.

1. Greeting
2. Scrape software category from Capterra (create category objects with class)
3. Ask user to enter keyword or alphabet
⋅⋅* If letter, list all the software under the letter with index
⋅⋅* If keyword
⋅⋅⋅⋅a. List index + software categories with keyword if there is any
⋅⋅⋅⋅b. Tell there is no software category with that keyword, and ask again to enter keyword
⋅⋅* Exit - exit
 4. Scrape the categories
 6. Software categories are displayed
 7. Ask user to input the index number for the category
 8. Scrape the software lists
 9. List all the software name and rating with index under the selected category
 10. Ask if the user would like to search category again.
 11. Loop or exit
  
There is basic setups I needed to do before actual coding. 
1. Created Gem with [Bundler](https://bundler.io/v1.13/guides/creating_gem) 
2. Created Git repository on Github (and, connect to the local repository)

With bundler setup, loading files into the environment file was quite simple, but it was interesting that gemspec file is collecting all the fils from `lib` folder, then adding them into $LOAD_PATH. I will research and write more about loading files and entry point later on. 

```

lib = File.expand_path("../lib", __FILE__)
$LOAD_PATH.unshift(lib) unless $LOAD_PATH.include?(lib)
require "software_binder/version"
```


For classes and objects, I created Controller, Scraper, Category, and Student. 
Controller class comes with program flow, and accesses other classes to use methods or get data.
Scraper class is to scrape data from the external source, and in the meantime, it accesses Category or Student class to instantiate objects with scraped data.
Category and Student class is there to instantiate the objects and gives access to data.

The rest of the coding went without major issues actually, and I spent quite a time to get used to using pry and testing with console while doing this projects. 


Below are the issues that I had at the end - 
1. `undefined method 'attribute' for nil:NilClass (NoMethodError)`

Below was the wrong code that made me think several hours. 

```
  def self.scrape_softwares(category)
    html = open("https://www.capterra.com/#{category.slug}")
    doc = Nokogiri::HTML(html)

    doc.css(".listing").each do |element|
      software = SoftwareBinder::Software.new(category)
      software.name = element.css(".listing-name a").text.strip
      software.description = element.css(".listing-description").text.strip.gsub(/\s{2,}/,' ').gsub(" Learn more about #{software.name}",'')
      software.page_slug = element.css(".listing-description a").attr("href").value
      software.overall_rating = element.css(".reviews").nil? ? element.css(".reviews").attr("data-rating").value.split("/")[0]  : "0.0"
      software.reviews = element.css(".reviews").nil? ? element.css(".reviews").attr("data-rating").value.split(" - ")[1] : 0
    end
	end
```

Below is the fixed code - I was testing wrong code with pry, and did not use ternary operater properly.
 
```
   def self.scrape_softwares(category)
    html = open("https://www.capterra.com/#{category.slug}")
    doc = Nokogiri::HTML(html)

    doc.css(".listing").each do |element|
      software = SoftwareBinder::Software.new(category)
      software.name = element.css(".listing-name a").text.strip
      software.description = element.css(".listing-description").text.strip.gsub(/\s{2,}/,' ').gsub(" Learn more about #{software.name}",'')
      software.page_slug = element.css(".listing-description a").attr("href").value

      if !element.css(".reviews").empty?
        software.overall_rating = element.css(".reviews").attr("data-rating").value.split("/")[0]
        software.reviews = element.css(".reviews").attr("data-rating").value.split(" - ")[1]
      else
        software.overall_rating = "0.0"
        software.reviews = 0
      end
    end
  end
```

2. Releasing the gem, so I can use `gem install` 
With Bundler, I ran`rake build` and `rake release`. Before that, gemspec file needed some adjustment to find the executable file in the right directory. Also, authentication needs attention. running `gem push <pkg name>` might help to make suer that you can push to rubygems.org. `rake release` apparently hits both github and rubygems.

```
  spec.homepage      = "https://github.com/ylhoony/software-binder-cli-app"
  spec.license       = "MIT"
  spec.metadata["allowed_push_host"] = "https://rubygems.org"
	spec.bindir        = "bin"
  spec.executables   = "software_binder"
  spec.require_paths = ["lib"]
```





