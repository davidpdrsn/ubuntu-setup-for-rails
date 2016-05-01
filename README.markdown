Ubuntu setup script
===================

This is how I normally setup a VPS running Ubuntu for hosting an app made with Ruby on Rails.

## Create the user

- `ssh root@<server ip address>`
- Save root password in 1Password
- `adduser <username>`
- `visudo`, duplicate line that begins with "root" but replace with new username

## Setup SSH authentication

Run VPS

```bash
ssh <username>@<ip address>
ssh-keygen -t rsa
```

Run locally

```
cat ~/.ssh/id_rsa.pub | ssh <username>@<ip address> "mkdir -p ~/.ssh && cat >>  ~/.ssh/authorized_keys"
```

Then [disable password authentication](http://askubuntu.com/questions/435615/disable-password-authentication-in-ssh)

## Setup Ruby

Following [this guide](https://www.digitalocean.com/community/tutorials/how-to-install-ruby-on-rails-with-rbenv-on-ubuntu-14-04)

```bash
# rbenv
sudo apt-get update
sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties libffi-dev

# Ruby
cd
git clone git://github.com/sstephenson/rbenv.git .rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bash_profile
source ~/.bash_profile
rbenv install -v 2.3.0
rbenv global 2.3.0
echo "gem: --no-document" > ~/.gemrc
gem install bundler
rbenv rehash

# Rails
gem install rails
rbenv rehash

# Node (for asset pipeline)
sudo add-apt-repository ppa:chris-lea/node.js
sudo apt-get update
sudo apt-get install nodejs
```

## Configure git

```bash
git config --global user.name "David Pedersen"
git config --global user.email "david.pdrsn@gmail.com"
cat ~/.ssh/id_rsa.pub
# paste the key [here](https://github.com/settings/ssh)
ssh git@github.com
```

## Fix locale issues

Add the following to ~/.bash_profile

```
export LC_CTYPE=en_US.UTF-8
export LC_ALL=en_US.UTF-8
```

## Install Postgres

```bash
# might require exiting and logging in again
sudo apt-get update -qq && apt-get install -y --no-install-recommends libpq-dev
sudo -i -u postgres
# type password of unix user
psql
\q
sudo -u postgres createuser <app name> -s
sudo -u postgres psql
\password <app name>
CREATE DATABASE <app name>_production OWNER blog;
# give user the same name as the repo on GitHub
# store username and password in 1Password
sudo -i -u cardistryio
psql -d <app name>_production
\conninfo
\q
exit
```

## Install Nginx

Following [this guide](https://gorails.com/deploy/ubuntu/14.04)

```bash
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7
sudo apt-get install -y apt-transport-https ca-certificates
sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger trusty main > /etc/apt/sources.list.d/passenger.list'
sudo apt-get update
sudo apt-get install -y nginx-extras passenger
sudo service nginx start
# open ip address in browser and make sure you see nginx welcome page
sudo vim /etc/nginx/nginx.conf
# uncomment passenger lines and replace second one with:
passenger_ruby /home/<username>/.rbenv/shims/ruby;
```

## Setup Rails app for deployment

See [cardistryio](https://github.com/davidpdrsn/CardistryIO) for an example of how to do this.

- Setup capistrano
- Setup Capfile
- Setup deploy.rb
- Setup config/production.rb
- Link database.yml
- Link application.yml

## Setting up monitoring of system
