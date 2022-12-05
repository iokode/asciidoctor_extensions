require 'uri'
require 'net/http'
require 'json'
require 'date'

# Define the extension
class TweetBlockMacro < Asciidoctor::Extensions::BlockMacroProcessor
  # Implement the process method
  def process parent, target, attrs
    tweet_id = target

    # Validate that the ID is an integer value
    unless tweet_id =~ /\A\d+\z/
      raise "Invalid tweet ID: #{tweet_id}"
    end

    # Check if the 'TWITTER_BEARER_TOKEN' environment variable is set
    unless ENV.key?('TWITTER_BEARER_TOKEN')
      raise "Twitter bearer token is not set"
    end

    # Generate the HTML code for the tweet
    html = generate_tweet_html(tweet_id)

    # Create a new block containing the tweet HTML and add it to the document
    create_block parent, :pass, html, attrs
  end

  def generate_tweet_html tweet_id
    # Use the Twitter API to fetch the tweet by its ID
    tweet = fetch_tweet(tweet_id)
    tweet_data = tweet['data']
    tweet_author = tweet['includes']['users'][0]
    created_at = DateTime.parse(tweet_data['created_at'])
  
    # Generate the HTML code for the tweet
    html = <<~HTML
    <blockquote class="twitter-tweet" data-lang="en">
        <p lang="en" dir="ltr">#{tweet_data['text']}</p>&mdash; #{tweet_author['name']} (@#{tweet_author['username']})
        <a href="https://twitter.com/#{tweet_author['username']}/status/#{tweet_id}">#{created_at.strftime('%b %d, %Y')}</a>
    </blockquote>
    <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
    HTML

    html
  end

  def fetch_tweet tweet_id
    # construct the URL for the Twitter API request
    tweet_url = "https://api.twitter.com/2/tweets/#{tweet_id}?user.fields=id,name,username&tweet.fields=id,text,created_at&expansions=author_id"
    uri = URI.parse(tweet_url)
  
    # create the HTTP request and set the necessary headers
    request = Net::HTTP::Get.new(uri)
    request['Authorization'] = "Bearer " + ENV["TWITTER_BEARER_TOKEN"]
    request['Accept'] = 'application/json'
  
    # make the HTTP request and retrieve the response
    response = Net::HTTP.start(uri.host, uri.port, use_ssl: true) do |http|
      http.request(request)
    end
  
    if response.code != '200'
      if response.code == '404'
        raise "Error: Tweet not found with ID #{tweet_id}"
      else
        raise "Error: #{response.code} #{response.message}"
      end
    end
  
    tweet = JSON.parse(response.body)
    tweet
  end
end

# Register the extension
Asciidoctor::Extensions.register do
  block_macro TweetBlockMacro, 'tweet'
end
