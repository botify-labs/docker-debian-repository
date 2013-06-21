FROM ubuntu
MAINTAINER tcrevon@gmail.com

# make sure the package repository is up to date
RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
RUN apt-get update


# Install necessary packages
RUN apt-get install -qq -y sudo openssh-server nginx reprepro


# Import context
ADD ./context /tmp/context


# Configure a localshop user
# Prepare user and directories
RUN addgroup --system debianrepo
RUN adduser --system --shell /bin/bash --gecos 'debian repository dedicated user' --uid 5000 --disabled-password --home /home/debianrepo debianrepo
RUN adduser debianrepo debianrepo


# Set user debrepo as sudoer, and eventualy add authorized keys for
# the user to be be used through ssh
RUN cp /tmp/context/etc/sudoers /etc/sudoers
RUN mkdir /home/debianrepo/.ssh
RUN cp /tmp/context/home/debianrepo/.ssh/authorized_keys /home/debianrepo/.ssh/


# Prepare the nginx endpoint
RUN mkdir -p /var/www/repos/apt/debian


# Configure reprepro
RUN mkdir -p /var/www/repos/apt/debian/conf
RUN cp /tmp/context/var/www/repos/apt/debian/conf/distributions /var/www/repos/apt/debian/conf/distributions
RUN cp /tmp/context/var/www/repos/apt/debian/conf/options /var/www/repos/apt/debian/conf/options
RUN cp /tmp/context/var/www/repos/apt/debian/conf/override /var/www/repos/apt/debian/conf/override


# Open ports for nginx and ssh server
EXPOSE 80
EXPOSE 22


# workaround sshd rights seperation problem
RUN mkdir /var/run/sshd

CMD ["/usr/sbin/sshd", "-D"]
CMD ["/usr/sbin/nginx", "-g", "daemon off;"]
