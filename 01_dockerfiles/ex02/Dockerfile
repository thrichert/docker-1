FROM ruby:2.3

LABEL maintainer="trichert <trichert@student.42.fr>"
LABEL name=ft_rails

RUN useradd -m ruby

RUN apt-get update &&\
	apt-get upgrade &&\
	apt-get install -y --no-install-recommends \
			postgresql-client &&\


ONBUILD COPY . /opt/app

ONBUILD WORKDIR /opt/app

ONBUILD RUN bundle install
ONBUILD RUN rails db:migrate
ONBUILD RUN rails db:seed


