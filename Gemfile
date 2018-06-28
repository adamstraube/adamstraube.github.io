source "https://rubygems.org"

require 'json'
require 'open-uri'
versions = JSON.parse(open('https://pages.github.com/versions.json').read)
gem "github-pages", versions['github-pages']
#gem "github-pages", group: :jekyll_plugins

#gem "jekyll", "~> 3.3.0"
#group :jekyll_plugins do
#  gem "jekyll-paginate"
#  gem "jekyll-sitemap"
#  gem "jekyll-gist"
#  gem "jekyll-feed"
#  gem "jemoji"
#end
#gem "minimal-mistakes-jekyll"

gem "jekyll-compose", group: [:jekyll_plugins] 
gem "wdm", "~> 0.1.0" if Gem.win_platform?
gem 'jekyll-archives'
