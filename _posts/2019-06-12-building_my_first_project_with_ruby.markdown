---
layout: post
title:      "Building my First Project with Ruby"
date:       2019-06-12 05:20:22 -0400
permalink:  building_my_first_project_with_ruby
---


As a movie lover, I weirdly enjoy seeing how movies are performing. You can usually tell how movies are doing by the box office results from their opening weekend. It’s also interesting to see how long movies stay in the Top 10 at the box office. For example, at the time of writing this, Avengers: Endgame is in its 7th week and is number 8 on the list. I’m curious to see how long it’ll stay in the Top 10. It’s currently the top grossing movie of 2019. 

For my first coding project, I created a Ruby gem that provides a CLI to view the Top 10 movies at the weekend box office. The information is scraped from IMDb’s website using Nokogiri. 

### Scraper class

I started by creating a Scraper class, with multiple class methods to separate what and where I scrape, that parses the HTML on IMDb’s web page to pull information specified using CSS selectors. This can be a bit tricky, depending on the web page that is being scraped. The Top 10 movies are placed in a table on IMDb’s page (I’ll call this page, with the Top 10 movies, my index page). The thing with tables is that they are specified with table tags, such as <tr> (to indicate a table row) and <td> (to indicate a table cell), without unique identifiers/names. When this is the case and there are multiple tables on the page, Nokogiri pulls all the information within those tags. So, the more specific your CSS selectors are, the more accurate the information you scrape will be.

Since the page I scraped had multiple <tr> tags that were outside the table I needed, I had to remove any excess or irrelevant information. Because the page only reports the Top 10 movies at the weekend box office, I know I only needed ten rows of data. I used the [slice](https://ruby-doc.org/core-2.6.3/Array.html#method-i-slice) method to extract the ten rows of data needed by using ` array[1..10] ` on my nested node. Before slicing the array, I used Pry to determine where to slice. Upon using Pry to view the scraped information, I found that passing the CSS selector ("td.titleColumn a") in as an argument to Nokogiri .css method ` row.css("td.titleColumn a").text ` returned an empty string. I was sure I used the correct CSS selector to retrieve the movie title, so I returned the first element in the collection provided by Nokogiri. 
```
#(Element:0x12fb240 {
  name = "tr",
  children = [
    #(Text "\n                    "),
    #(Element:0x12faf0c { name = "th" }),
    #(Text "\n                    "),
    #(Element:0x12fac50 { name = "th", children = [ #(Text "Title")] }),
    #(Text "\n                    "),
    #(Element:0x12fa5fc { name = "th", children = [ #(Text "Weekend")] }),
    #(Text "\n                    "),
    #(Element:0x12fa034 { name = "th", children = [ #(Text "Gross")] }),
    #(Text "\n                    "),
    #(Element:0x1303a6c { name = "th", children = [ #(Text "Weeks")] }),
    #(Text "\n                    "),
    #(Element:0x130351c { name = "th" }),
    #(Text "\n                ")]
  }) 
```
Turns out, the first element is the table header (but still preceded with a <tr> tag) and why an empty string was returned when I used the CSS selector to retrieve the movie title text. ` row.css("td.titleColumn a").text ` 
The second element in the collection was the first movie on the list, so I selected the desired movie elements with ` array[1..10] `

After grabbing all the rows with movies, I iterated through each movie element and set the value of the attributes I want to pull to a variable. I create symbols to represent my attributes and set the symbols to their respective variables that contain the value in a hash as key/value pairs. Then, I use the shovel method to add the hash to the empty array I created at the beginning of my scraping method. 

Next, I created a class method to scrape each movie page by passing in the path url of each movie I scraped on the index page as an argument. Again, I use Nokogiri and CSS selectors to retrieve attributes on the movie page. The attributes are placed in a hash and assigned as values to their corresponding symbol keys.

Note: I only created class methods in my Scraper class because I don’t need to store any information. I’m scraping the information and passing it to the Movie class. 

### Movie class

I started my Movie class by creating attribute accessors for all my movie attributes. That way, I have a way to set and get my attributes. 

I created a class variable (@@all) set to an empty array to store all my Movie instances/objects and a class method (.all) to return the array of Movie objects. 

I created an initialize method to accept an argument when a new instance is created (when .new is called). When creating a new Movie instance, the argument passed in is a hash that contains key/value pairs of attributes obtained by the Scraper method. Mass assignment and metaprogramming are used so the initialize method can take an argument of a hash and iterate through each key/pair value in the hash to set the symbol (key in hash) to the attribute value (value in hash) by calling .send on the instance. Then, the instance is added to the class variable by shoveling self (the instance) into the class variable (@@all). 

Next, I created a class method (.create_from_collection) that is passed the array containing hashes of movie attributes scraped from the index page. Each element in the array is iterated over to instantiated a new Movie object with the hash containing the movie attributes passed in as an argument. 

The last method in the Movie class is an instance method (#add_movie_details) and takes in a hash of movie attributes scraped from a movie’s page as an argument. This method works like the initialize method, where mass assignment and metaprogramming are utilized with the .send method to set the key (as a symbol) to its value pair.

### CLI class

To view all the scraped information and each Movie objects’ attributes, a CLI was created to display the list of Top 10 Movies. The user is prompted to choose a number associated with the movie’s rank to view more details on the movie. After a movie’s details are displayed, the user is again prompted to choose to view another movie, go back to the list, or to exit the application. If an invalid entry is given, an error message will show and the user will be asked to provide a valid input. The CLI will continue to prompt the user for input until the user exits the application. This loop was created using the while loop `while input.downcase != "exit"` with conditional if statements.
