# This image provides a base for building and running WildFly applications.
# It builds using maven and runs the resulting artifacts on WildFly 10.1.0 Final

FROM sinenomine/java-s390x:latest

MAINTAINER Ben Parees <bparees@redhat.com>

EXPOSE 8080 9990

ENV WILDFLY_VERSION=10.1.0.Final \
    MAVEN_VERSION=3.3.9 \
    STI_SCRIPTS_PATH=/usr/libexec/s2i

LABEL io.k8s.description="Platform for building and running JEE applications on WildFly 10.1.0.Final" \
      io.k8s.display-name="WildFly 10.1.0.Final" \
      io.openshift.expose-services="8080:http,9990:http" \
      io.openshift.tags="builder,wildfly,wildfly10" \
      io.openshift.s2i.destination="/opt/s2i/destination"

# Install Maven, Wildfly 10
RUN INSTALL_PKGS="tar unzip bc which lsof telnet" && \
    yum install -y $INSTALL_PKGS && \
    yum clean all -y 

RUN (curl -v https://www.apache.org/dist/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz | \
    tar -zx -C /usr/local) && \
    ln -sf /usr/local/apache-maven-$MAVEN_VERSION/bin/mvn /usr/local/bin/mvn

RUN mkdir -p /root/.m2 && \
    mkdir -p /wildfly && \
    (curl -v https://download.jboss.org/wildfly/$WILDFLY_VERSION/wildfly-$WILDFLY_VERSION.tar.gz | tar -zx --strip-components=1 -C /wildfly) && \
    mkdir -p /opt/s2i/destination

# Add s2i wildfly customizations
ADD ./contrib/wfmodules/ /wildfly/modules/
ADD ./contrib/wfbin/standalone.conf /wildfly/bin/standalone.conf
ADD ./contrib/wfcfg/standalone.xml /wildfly/standalone/configuration/standalone.xml

LABEL updated=15:17:10

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

RUN groupadd -g 1001 wildfly && \
    useradd -c "Wildfly User" -m -g wildfly -u 1001 wildfly && \
    chown -R wildfly:wildfly /wildfly /opt/s2i/destination && \
    chmod -R ug+rw /wildfly && \
    chmod -R g+rw /opt/s2i/destination

ADD ./contrib/settings.xml /home/wildfly/.m2/

RUN chown -R wildfly:wildfly /home/wildfly/.m2

USER 1001

LABEL io.openshift.s2i.scripts-url=image:///usr/libexec/s2i

CMD $STI_SCRIPTS_PATH/usage
