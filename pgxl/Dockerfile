FROM ubuntu:14.04
MAINTAINER Xiaodong Wang

USER root

### USER
RUN adduser pom --disabled-password --gecos "" --home /pom --shell /bin/bash
RUN adduser pom sudo
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

### BASE PACKAGES
RUN apt-get update && \
    apt-get install -y byobu cmake vim software-properties-common openssh-server openssh-client rsync && \
    apt-get install -y haproxy pgxnclient curl flex bison zlib1g-dev libreadline6-dev gcc make supervisor git python-dev

### LOCALE
RUN apt-get install -y locales && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:fr
ENV LC_ALL en_US.UTF-8
ENV LANGUAGE en_US:en

### PGXL
WORKDIR /root
RUN wget -q -O - http://files.postgres-xl.org/postgres-xl-9.5r1.4.tar.gz | tar xzvf - |tail
WORKDIR /root/postgres-xl-9.5r1.4
RUN ./configure --with-python #--prefix /opt/pgxl
# ./configure --enable-cassert --enable-debug CFLAGS="-ggdb -Og -fno-omit-frame-pointer" --with-python --prefix /opt/pgxl
RUN make -j 4 |tail && make install
WORKDIR /root/postgres-xl-9.5r1.4/contrib
RUN mv test_decoding/specs test_decoding/.specs && make -j 4 |tail && make install

#ENV PATH /opt/pgxl/bin:$PATH
ENV PGDATA /pom/pgxc

### LD
RUN echo "/usr/local/lib" >> /etc/ld.so.conf
#RUN echo "/opt/pgxl/lib" >> /etc/ld.so.conf
RUN ldconfig

### CONF
RUN mkdir -p /var/log/supervisor
COPY etc/supervisor/conf.d/supervisor.conf /etc/supervisor/conf.d/
COPY etc/sysctl.conf /etc/sysctl.conf
COPY etc/environment /etc/environment

###GIS
RUN sudo add-apt-repository ppa:ubuntugis/ppa &&\
	sudo apt-get update
RUN  sudo apt-get install -y  libgdal1-dev libgeos-dev libproj-dev libgdal-dev
WORKDIR /root
RUN wget -q -O - http://download.osgeo.org/postgis/source/postgis-2.3.1.tar.gz | tar xzvf - |tail
WORKDIR /root/postgis-2.3.1
RUN ./configure --with-pgconfig=/usr/local/pgsql/bin/pg_config
RUN make && make install
### SSH
RUN echo "   StrictHostKeyChecking no" >> /etc/ssh/ssh_config
RUN sudo /etc/init.d/ssh start && sudo /etc/init.d/ssh stop

### USER
ADD . /app
RUN chown -R pom:pom /app /pom
USER pom
WORKDIR /pom
ENV HOME /pom

ENTRYPOINT ["/app/entrypoint.sh"]

EXPOSE 5432

CMD ["supervisor"]
