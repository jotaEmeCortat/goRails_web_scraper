# Build a Web Scraper in Ruby on Rails 7

[https://gorails.com/series/build-a-web-scraper-in-rails](https://gorails.com/series/build-a-web-scraper-in-rails)

## Creating a New Rails app

```bash
rails new web_scraper
```

After creating the class Scraper. This class will be responsible for fetching the HTML document and parsing it.

```ruby
require "net/http"

class Scraper
  attr_reader :document

  def initialize(url)
    response = Net::HTTP.get(URI(url))
    @document = Nokogiri::HTML(response)
  end

  def text(selector:)
    document.at_css(selector).text
  end

  def present?(selector:)
    document.at_css(selector).present?
  end
end
```

```bash
rails c
# Loading development environment (Rails 7.0.10)
irb(main):001> Scraper.new("https://www.adafruit.com/product/6407").text(selector:"[itemprop='availability']")
=> "in stock"
```
