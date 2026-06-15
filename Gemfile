source "https://rubygems.org"

# Safe zone: Core compiler tools we need locally
gem "jekyll"
gem "webrick"

# Isolated zone: The heavy plugin suite causing the eventmachine crash
group :jekyll_plugins do
  gem "github-pages"
end