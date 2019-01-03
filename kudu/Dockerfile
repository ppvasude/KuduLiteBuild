FROM oryxprod/build:20181220.1 as main

ENV DEBIAN_FRONTEND noninteractive

# Install dependencies
RUN echo "deb http://ftp.debian.org/debian jessie-backports main" >> /etc/apt/sources.list \
  && apt-get update \
  && apt-get install -y openssh-client --no-install-recommends \
  && apt-get install -y vim tree --no-install-recommends \
  && apt-get install -y tcptraceroute \
  && wget -O /usr/bin/tcpping http://www.vdberg.org/~richard/tcpping \
  && chmod 755 /usr/bin/tcpping 

# SQL Server gem support
RUN apt-get install -y unixodbc-dev freetds-dev freetds-bin 

COPY TunnelExtension.zip webssh.zip ssh /tmp/

# Install Kudu
RUN mkdir -p /opt/Kudu/local \
  && chmod 755 /opt/Kudu/local \
  && apt-get update \
  && apt-get  install -y unzip \
# Install pm2 and pm2-logrotate
  && mkdir -p /home/LogFiles \
# Install webssh
  && mkdir /opt/webssh \
  && unzip /tmp/webssh.zip -d /opt/webssh \
# Replace ssh with wrapper script for CIFS mount permissions workaround
  && mv /usr/bin/ssh /usr/bin/ssh.original \
  && mv /tmp/ssh /usr/bin/ssh \
  && chown root:root /usr/bin/ssh \
  && chmod 755 /usr/bin/ssh \
  && chmod -R 777 /home \
  && rm -rf /tmp/*

ENV DOTNET_RUNNING_IN_CONTAINER=true

# Enable correct mode for dotnet watch (only mode supported in a container)
ENV DOTNET_USE_POLLING_FILE_WATCHER=true

# Skip extraction of XML docs - generally not useful within an image/container - helps performance
ENV NUGET_XMLDOC_MODE=skip

RUN dotnet tool install -g dotnet-aspnet-codegenerator 
ENV PATH=$PATH:/root/.dotnet/tools

#Instal Kudu
RUN cd /tmp \
    && git clone https://github.com/Azure-App-Service/KuduLite.git \
    && cd ./KuduLite \
    && cd ./Kudu.Services.Web \
    && dotnet publish -c Release -o /opt/Kudu \
    && chmod 777 /opt/Kudu/Kudu.Services.Web.dll \
    && rm -rf /tmp/*             

COPY startup.sh /

RUN chmod 777 /startup.sh

#required for kudu
RUN ln -s /opt/nodejs/8.11.2/lib/node_modules/npm/bin/npm-cli.js /usr/bin/npm-cli.js

RUN npm install -g kudusync
RUN npm install pm2@latest -g

#required for kudusync
ENV PATH=$PATH:/opt/nodejs/8.11.2/bin 

EXPOSE 8181

ENTRYPOINT [ "/startup.sh" ]
CMD [ "1002", "kudu_group", "1001", "kudu_user", "localsite" ]
