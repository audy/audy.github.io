source 'https://rubygems.org'

require 'json'
require 'open-uri'

ruby '2.3.0'

versions = JSON.parse(open('https://pages.github.com/versions.json').read)

gem 'github-pages', versions['github-pages']
