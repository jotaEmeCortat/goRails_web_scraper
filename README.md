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

## Storing Web Scraper Results in ActiveRecord models

Generate a scaffold for the `Page` model.

```bash
rails g scaffold Page url check_type selector match_text
```

And generate a model for the `Result`

```bash
rails g model Result page:references success:boolean
```

Files changed in this section:

```ruby
# config/routes.rb
root "pages#index"
```

```ruby
# app/models/page.rb
class Page < ApplicationRecord
  has_many :results

  validates :url, presence: true
  validates :check_type, presence: true
  validates :selector, presence: true
  validates :match_text, presence: {if: ->{ check_type == "text" }}

  def run_check
    scraper = Scraper.new(url)
    result = case check_type
             when "text"
               scraper.text(selector: selector).downcase == match_text.downcase
             when "exists"
               scraper.present?
             when "not_exists"
               !scraper.present?
             end
    results.create(success: result)
  end
end
```
