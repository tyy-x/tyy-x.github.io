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

## Copy files of the theme

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
- _config.yml
- Gemfile


## Build and Customize
After copying the essential files, the site can be built now. Run `bundle exec jekyll serve --lsi`. Then customize as needed.


# Deploy

## Copy workflows and other needed files
- .github/workflows/axe.yml
- .github/workflows/broken-links-site.yml
- .github/workflows/broken-links.yml
- .github/workflows/deploy.yml
- .github/workflows/lighthouse-badger.yml
- lighthouse-results/ (I think this will be generated automatically by
    `lighthouse-badger-action` after deploy, so this may not be really needed. Not verified though)
- purgecss.config.js (needed in `Delpoy site` workflow)


## Modify `lighthouse-badger` workflow

- set `URLS` to the urls that is to be checked, which in this case is my website: `https://tyy-x.github.io`
- set `TOKEN_NAME` to the name of my repository's `secret`
    - in order to create a secret, first create a `PAT`
- change `REPO_BRANCH: "${{ github.repository }} master"` to `REPO_BRANCH: "${{ github.repository }} main"`

## Remove and untrack Gemfile.lock
This step may not be needed.

## Change repo settings
- go to `Settings -> Actions -> General -> Workflow permissions` and give `Read and write permissions`
    to GitHub Actions

## Push to GitHub
- push to GitHub
- wait until GitHub Actions finish

## Change publishing source branch
- change publishing source branch from `main` to `gh-pages` (how is gh-pages generated automatically???)

Done.
