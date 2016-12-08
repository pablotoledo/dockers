FROM ubuntu

MAINTAINER jtoledog "jpablotoledo92@gmail.com"

# User for machine
ARG username=machine
ARG password=rails
ARG projectName=firstProject

# Configuring ports
EXPOSE 3000
EXPOSE 22

# Instaling basic software
RUN apt-get update && apt-get install -y openssh-server supervisor apt-utils sudo less man software-properties-common wget nano build-essential apt-transport-https libssl-dev libreadline-dev git-core curl zlib1g-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties libffi-dev

# Create user
RUN useradd -m ${username}
RUN echo "${username}:${password}" | chpasswd
RUN adduser ${username} sudo
RUN echo "${username} ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# Configuring supervisord
RUN echo [supervisord] >> /etc/supervisor/conf.d/supervisord.conf
RUN echo nodaemon=true >> /etc/supervisor/conf.d/supervisord.conf
RUN echo [program:sshd] >> /etc/supervisor/conf.d/supervisord.conf
RUN echo 'command=bash -c "/usr/sbin/sshd -D; sudo service ssh start"' >> /etc/supervisor/conf.d/supervisord.conf
RUN echo [program:rails] >> /etc/supervisor/conf.d/supervisord.conf
RUN echo user="${username}" >> /etc/supervisor/conf.d/supervisord.conf
RUN echo directory=/home/"${username}"/"${projectName}"/ >> /etc/supervisor/conf.d/supervisord.conf
RUN echo 'command=bash -c "bundle install && rails s -b 0.0.0.0"'  >> /etc/supervisor/conf.d/supervisord.conf

#Enviroment Variables
USER ${username}
ENV USERDIR=/home/${username}
ENV PATH=${USERDIR}/.rbenv/shims:${USERDIR}/.rbenv/plugins/ruby-build/bin:${USERDIR}/.rbenv/bin:$PATH
ENV RAILS_VERSION 4.2.6

#Installing Ruby
RUN cd; pwd ; git clone https://github.com/rbenv/rbenv.git ~/.rbenv; 
RUN cd; echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc;
RUN cd; echo 'eval "$(rbenv init -)"' >> ~/.bashrc; exec $SHELL;
RUN cd; git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build; 
RUN cd; echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc; exec $SHELL
RUN echo $PATH

RUN rbenv install 2.3.1
RUN rbenv global 2.3.1
RUN rbenv rehash
RUN rbenv init -; gem install bundler
RUN curl -sL https://deb.nodesource.com/setup_4.x | sudo -E bash -
RUN sudo apt-get install -y nodejs

# Installing rails
RUN sudo apt-get update && sudo apt-get install -y mysql-client postgresql-client sqlite3 --no-install-recommends && sudo rm -rf /var/lib/apt/lists/*
RUN gem install rails --version "$RAILS_VERSION"

# Create project
RUN cd; rails new "${projectName}"; cd "${projectName}"; bundle install; rake db:migrate

RUN sudo chsh -s /bin/bash

# to execute supervisor must be root user
USER root

CMD ["/usr/bin/supervisord"]
