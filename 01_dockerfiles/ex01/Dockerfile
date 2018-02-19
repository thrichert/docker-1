FROM debian

MAINTAINER trichert <trichert@student.42.fr>

EXPOSE 9987/udp
EXPOSE 30033
EXPOSE 10011

RUN useradd -m ts3

WORKDIR /home/ts3

RUN apt-get update &&\
	apt-get upgrade &&\ 
    apt-get -y install bzip2 wget ca-certificates &&\
	wget -O /home/ts3/teamspeak3.tar.bz2 http://dl.4players.de/ts/releases/3.0.13.6/teamspeak3-server_linux_amd64-3.0.13.6.tar.bz2 &&\
	ls -la /home/ts3/ &&\
	tar -xjvf /home/ts3/teamspeak3.tar.bz2

COPY ts3-config.sh /home/ts3/teamspeak3-server_linux_amd64/ts3-config.sh

USER ts3

ENTRYPOINT ["bash", "/home/ts3/teamspeak3-server_linux_amd64/ts3server_minimal_runscript.sh", "inifile=ts3-config.sh"]
