# Setup

## Requirements

- Ruby
- g++ (needed when install Jekyll)
- Bundler
- Jekyll
- wget (needed by the theme when doing `jekyll serve` if my memory is correct)
- Jupyter (require `pipx`)
- imagemagick (needed to resolve errors when doing `jekyll serve`)

### Ruby, g++, wget, pipx, imagemagick
```
sudo apt install ruby-full g++ wget pipx imagemagick
```

### Bundler and Jekyll
Bundler and Jekyll are ruby gems.
Before install them, add the following to `.bashrc`.
```
# Install Ruby Gems to ~/gems
export GEM_HOME="$HOME/gems"
export PATH="$HOME/gems/bin:$PATH"
```
Then install Bundler and Jekyll `gem install bundler jekyll`.

### Jupyter
To install Jupyter, `pipx` is needed. (What is pipx?)
```
pipx install jupyter --include-deps
pipx ensurepath
```

## Copy the essential files of the theme

- _bibliography/
- _data/
- _includes/
- _layouts/
- _news/
- _plugins/
- _posts/
- _projects/
- _sass/
- assets/
- bin/
- lighthouse_results/
- readme_preview/
- _config.yml
- Gemfile


## Build and Customize
After copying the essential files, the site can be built now. Run `bundle exec jekyll serve --lsi`. Then customize as needed.
