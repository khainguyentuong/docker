# Set the base image
FROM java:8-jdk

# File Author / Maintainer
MAINTAINER Khai Nguyen <khainguyentuong@gmail.com>
 
# Overdrive enabled environment variables
ENV ROLE master
ENV SSH_PASS Khoan2008
ENV ENV prod
ENV TERMTAG JENKINS

# Run Update, and install Jenkins
RUN apt-get clean && \
apt-get update && \
apt-get -y upgrade && \
apt-get -y install sudo git wget openssh-server net-tools&& \
apt-get clean && \
rm -fr /var/lib/apt/lists/*

# Remove locales other than english
RUN for x in `ls /usr/share/locale | grep -v en_GB`; do rm -fr /usr/share/locale/$x; done && \
for x in `ls /usr/share/i18n/locales/ | grep -v en_`; do rm -fr /usr/share/i18n/locales/$x; done

# Recreate a needed directory so that java headless installs
RUN mkdir -p /usr/share/man/man1/ && \
touch /usr/share/man/man1/rmid.1.gz.dpkg-tmp

# Add Jenkins Repository
RUN wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add - && \
echo "deb http://pkg.jenkins-ci.org/debian binary/" >> /etc/apt/sources.list

RUN apt-get clean && \
apt-get update && \
apt-get -y install jenkins && \
apt-get clean && \
rm -fr /var/lib/apt/lists/*

# Make SSH Directory, Instruct Jenkins not to prompt for host key verification and set perms
RUN mkdir /var/lib/jenkins/.ssh && \
echo "Host *\n\tStrictHostKeyChecking no\n" >> /var/lib/jenkins/.ssh/config

RUN mkdir -p /var/cache/jenkins/war && \
mkdir /var/log/jenkins || exit 0 && \
cd /var/cache/jenkins/war && \
jar -xvf /usr/lib/jenkins/jenkins.war && \
chmod a+w ./

# Reset Permissions
RUN chown -R jenkins:jenkins /var/cache/jenkins && \
chown jenkins:jenkins /var/log/jenkins && \
chmod -R 775 /var/cache/jenkins && \
chmod -R 777 /var/log/jenkins && \
chown -R jenkins:jenkins /var/lib/jenkins && \
chown -R jenkins:jenkins /var/lib/jenkins/.ssh && \
chmod -R 0700 /var/lib/jenkins/.ssh && \
chmod -R 0600 /var/lib/jenkins/.ssh/*

# Add Database Scripts, and Runconfig
ADD runconfig.sh /tmp/.runconfig.sh

# Set boot items
RUN chmod +x /tmp/.runconfig.sh && \
echo "/tmp/./.runconfig.sh" >> /root/.bashrc && \
echo "[ -f /tmp/.runconfig.sh ] && rm -fr /tmp/.runconfig.sh" >> /root/.bashrc && \
echo "service jenkins start" >> /root/.bashrc

CMD /bin/bash
USER ${user}

# Expose ports to other containers only
EXPOSE 22
EXPOSE 8080
EXPOSE 50000

# Install plug-ins
COPY plugins.sh /usr/local/bin/plugins.sh
COPY install-plugins.sh /usr/local/bin/install-plugins.sh
COPY plugins.txt /usr/share/jenkins/plugins.txt
RUN /usr/local/bin/plugins.sh /usr/share/jenkins/plugins.txt
