FROM	debian


EXPOSE	5432 5431
EXPOSE	6379
EXPOSE	22 80/tcp
EXPOSE	10022
# update and standard install
RUN			apt-get update && apt-get upgrade -y && \
			apt-get install -y sudo \
			build-essential zlib1g-dev libyaml-dev libssl-dev libgdbm-dev libre2-dev libreadline-dev libncurses5-dev libffi-dev curl openssh-server checkinstall libxml2-dev libxslt-dev libcurl4-openssl-dev libicu-dev logrotate rsync python-docutils pkg-config cmake \
			git-core

# install ruby && bundler
RUN			sudo apt-get remove ruby1.8 &&\
			mkdir /tmp/ruby && cd /tmp/ruby &&\
			curl --remote-name --progress https://cache.ruby-lang.org/pub/ruby/2.4/ruby-2.4.3.tar.gz &&\
			tar xzf ruby-2.4.3.tar.gz && cd ruby-2.4.3 && ./configure --disable-install-rdoc &&\
			make && sudo make install &&\
			sudo gem install bundler --no-ri --no-rdoc

# install go && bundler

RUN			sudo rm -rf /usr/local/go &&\
			curl --remote-name --progress https://storage.googleapis.com/golang/go1.8.3.linux-amd64.tar.gz && \
			echo '1862f4c3d3907e59b04a757cfda0ea7aa9ef39274af99a784f5be843c80c6772  go1.8.3.linux-amd64.tar.gz' | shasum -a256 -c - && \
  			sudo tar -C /usr/local -xzf go1.8.3.linux-amd64.tar.gz && \
  			sudo ln -sf /usr/local/go/bin/{go,godoc,gofmt} /usr/local/bin/ && \
  			rm go1.8.3.linux-amd64.tar.gz

# install NodeJS && Yarn
RUN			curl --location https://deb.nodesource.com/setup_8.x | sudo bash - &&  sudo apt-get install -y nodejs && \
			curl --silent --show-error https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add - && \
			echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list && \
			sudo apt-get update && \
			sudo apt-get install yarn

# create GitLab user
RUN			sudo adduser --disabled-login --gecos 'GitLab' git
RUN			echo git:password | chpasswd --crypt-method=SHA512

# install Postgresql
RUN			sudo apt-get install -y postgresql postgresql-client libpq-dev postgresql-contrib &&\
			sudo -u postgres /etc/init.d/postgresql start && \
			sudo -u postgres psql -p 5432 -d template1 -c "CREATE USER git CREATEDB PASSWORD 'password';" && \
			sudo -u postgres psql -p 5432 -d template1 -c "CREATE EXTENSION IF NOT EXISTS pg_trgm;" && \
			sudo -u postgres psql -p 5432 -d template1 -c "CREATE DATABASE gitlabhq_production OWNER git;"

# install Redis
RUN 		sudo apt-get install -y redis-server && \
			sudo cp /etc/redis/redis.conf /etc/redis/redis.conf.orig && \
			sed 's/^port .*/port 0/' /etc/redis/redis.conf.orig | sudo tee /etc/redis/redis.conf && \
			echo 'unixsocket /var/run/redis/redis.sock' | sudo tee -a /etc/redis/redis.conf && \
			echo 'unixsocketperm 770' | sudo tee -a /etc/redis/redis.conf && \
			mkdir /var/run/redis && \
			chown redis:redis /var/run/redis && \
			chmod 755 /var/run/redis && \
			sudo service redis-server restart && \
			sudo usermod -aG redis git

# install GitLab !
WORKDIR		/home/git

RUN			sudo -u git -H git clone https://gitlab.com/gitlab-org/gitlab-ce.git -b 10-6-stable gitlab

WORKDIR		/home/git/gitlab
# configuration Database
RUN			sudo -u git -H cp config/gitlab.yml.example config/gitlab.yml &&\
			sudo -u git -H cp config/secrets.yml.example config/secrets.yml &&\
			sudo -u git -H chmod 0600 config/secrets.yml && \
			sudo -u git -H chown -R git log/ && \
			sudo -u git -H chown -R git tmp/ && \
			sudo -u git -H chmod -R u+rwX,go-w log/ && \
			sudo -u git -H chmod -R u+rwX tmp/ && \
			sudo -u git -H chmod -R u+rwX tmp/pids/ && \
			sudo -u git -H chmod -R u+rwX tmp/sockets/ && \
			sudo -u git -H mkdir public/uploads/ && \
			sudo -u git -H chmod 0700 public/uploads && \
			sudo -u git -H chmod -R u+rwX builds/ && \
			sudo -u git -H chmod -R u+rwX shared/artifacts/ && \
			sudo -u git -H chmod -R ug+rwX shared/pages/ && \
			sudo -u git -H cp config/unicorn.rb.example config/unicorn.rb && \
			sudo -u git -H cp config/initializers/rack_attack.rb.example config/initializers/rack_attack.rb && \
			sudo -u git -H git config --global core.autocrlf input && \
			sudo -u git -H git config --global gc.auto 0 && \
			sudo -u git -H git config --global repack.writeBitmaps true && \
			sudo -u git -H git config --global receive.advertisePushOptions true && \
			sudo -u git -H cp config/resque.yml.example config/resque.yml &&\
			sudo -u git -H cp config/database.yml.postgresql config/database.yml && \
			sudo -u git -H chmod o-rwx config/database.yml &&\
			sudo -u git -H sed -i 's/secure password/password/' config/database.yml

# install Gems
RUN			sudo -u git -H bundle install --deployment --without development test mysql aws kerberos &&\
			gem install rails &&\
			apt-get update && apt-get install -y golang &&\
			sudo -u git -H bundle exec rake gitlab:shell:install REDIS_URL=unix:/var/run/redis/redis.sock RAILS_ENV=production SKIP_STORAGE_VALIDATION=true

# install GitLab-workhorse
RUN			sudo -u git -H bundle exec rake "gitlab:workhorse:install[/home/git/gitlab-workhorse]" RAILS_ENV=production && \
			service postgresql restart && service redis-server restart && yes yes | sudo -u git -H bundle exec rake gitlab:setup -q RAILS_ENV=production

# init DB and Activate advanced features (Gitaly / logrotate)
RUN			sudo cp lib/support/init.d/gitlab /etc/init.d/gitlab &&\
			sudo update-rc.d gitlab defaults 21 &&\
			sudo -u git -H bundle exec rake "gitlab:gitaly:install[/home/git/gitaly]" RAILS_ENV=production &&\
			sudo chmod 0700 /home/git/gitlab/tmp/sockets/private && \
			sudo chown git /home/git/gitlab/tmp/sockets/private &&\
			sudo cp lib/support/logrotate/gitlab /etc/logrotate.d/gitlab &&\

# Restart and check application status
RUN service postgresql restart && sudo -u git -H bundle exec rake gitlab:env:info RAILS_ENV=production

# Compile Gettext PO files
RUN			sudo -u git -H bundle exec rake gettext:compile RAILS_ENV=production

# Compule Assets
RUN			sudo -u git -H yarn install --production --pure-lockfile &&\
			sudo -u git -H bundle exec rake gitlab:assets:compile RAILS_ENV=production NODE_ENV=production
# start Gitlab instance
RUN			service postgresql restart && service redis-server restart && sudo service gitlab start

# Install Nginx && configuration (+ ssh)
RUN			apt-get update && apt-get install -y nginx-full &&\
			rm /etc/nginx/sites-enabled/default &7\
			sudo cp lib/support/nginx/gitlab /etc/nginx/sites-available/gitlab && \
			sudo ln -s /etc/nginx/sites-available/gitlab /etc/nginx/sites-enabled/gitlab &&\
			sudo sed -i 's/YOUR_SERVER_FQDN/gitlab.trichert.com/' /etc/nginx/sites-available/gitlab &&\
			sed -i -e '30,62d' /etc/nginx/sites-available/gitlab &&\
			sed -i '51s+gitlab_access;+;+' /etc/nginx/sites-available/gitlab &&\
			sudo nginx -t &&\
			sed -i '582s/# ssh_port: 22/ssh_port: 30022/' /home/git/gitlab/config/gitlab.yml && \
		    mkdir /run/sshd

ADD config /home/git/.ssh/

CMD /usr/sbin/sshd && service postgresql start && service redis-server start && service gitlab start && service nginx start && bash

EXPOSE 80
EXPOSE 30022