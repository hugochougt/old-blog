source "https://gems.ruby-china.org"

require 'json'
require 'open-uri'
versions = JSON.parse(open('https://pages.github.com/versions.json').read)

gem 'github-pages', versions['github-pages']
gem 'jekyll-paginate', versions['jekyll-paginate']
gem 'jekyll', versions['jekyll']
