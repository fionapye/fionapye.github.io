# frozen_string_literal: true

source "https://rubygems.org"

gemspec

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
install_if -> { RUBY_PLATFORM =~ %r!mingw|mswin|java! } do
  gem "tzinfo", "~> 1.2"
  gem "tzinfo-data"
  gem 'wdm', '>= 0.1.0'
end

# Additional plugins
gem 'jekyll-toc'
gem "webrick"
