# frozen_string_literal: true

source "https://rubygems.org"

gem "jekyll-theme-chirpy", "~> 6.1"

group :test do
  gem "html-proofer", "~> 3.18"
end

# Lock jekyll-sass-converter to 2.x on Linux-musl
if RUBY_PLATFORM =~ /linux-musl/
  gem "jekyll-sass-converter", "~> 2.0"
end

# Windows file system watcher for faster live-reload
gem 'wdm', '>= 0.1.0' if Gem.win_platform?
