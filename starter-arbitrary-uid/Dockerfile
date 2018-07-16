# FROM registry.access.redhat.com/rhel7
# FROM centos:7
FROM ubuntu:16.04
MAINTAINER Mirek

# ### Atomic/OpenShift Labels - https://github.com/projectatomic/ContainerApplicationGenericLabels
# LABEL name="acme/starter-arbitrary-uid" \
#       maintainer="refarch-feedback@redhat.com" \
#       vendor="Acme Corp" \
#       version="3.7" \
#       release="1" \
#       summary="Acme Corp's Starter app" \
#       description="Starter app will do ....." \
# ### Required labels above - recommended below
#       url="https://www.acme.io" \
#       run='docker run -tdi --name ${NAME} \
#       -u 123456 \
#       ${IMAGE}' \
#       io.k8s.description="Starter app will do ....." \
#       io.k8s.display-name="Starter app" \
#       io.openshift.expose-services="" \
#       io.openshift.tags="acme,starter-arbitrary-uid,starter,arbitrary,uid"

# ### Atomic Help File - Write in Markdown, it will be converted to man format at build time.
# ### https://github.com/projectatomic/container-best-practices/blob/master/creating/help.adoc
# COPY help.md /tmp/

# ### add licenses to this directory
# COPY licenses /licenses

### Add necessary Red Hat repos here
# RUN REPOLIST=rhel-7-server-rpms,rhel-7-server-optional-rpms \
### Add your package needs here
#     INSTALL_PKGS="golang-github-cpuguy83-go-md2man" && \
#     yum -y update-minimal --disablerepo "*" --enablerepo rhel-7-server-rpms --setopt=tsflags=nodocs \
#       --security --sec-severity=Important --sec-severity=Critical && \
#     yum -y install --disablerepo "*" --enablerepo ${REPOLIST} --setopt=tsflags=nodocs ${INSTALL_PKGS} && \
# ### help file markdown to man conversion
#     go-md2man -in /tmp/help.md -out /help.1 && \
#     yum clean all

# set up home folder and add it to path
ENV HOME=/opt/app-root
ENV PATH=${HOME}/bin:${PATH}

# copy the uid_entrypoint script
COPY bin/ ${HOME}/bin/

# set up user for build execution and application runtime
RUN chmod -R u+x ${HOME}/bin && \
    chgrp -R 0 ${HOME} && \
    chmod -R g=u ${HOME} /etc/passwd

# switch to user, because containers should NOT run as root as a good practice
# note that Openshift best practice is to use a user number not user name
ARG MY_UID=10001
USER ${MY_UID}

# set up work dir
WORKDIR ${HOME}

# run script for user name recognition at runtime 
# with an arbitrary UID - for OpenShift deployments
ENTRYPOINT [ "uid_entrypoint" ]

# create volumes inside the container
# VOLUME ${HOME}/logs 
# VOLUME ${HOME}/data

# default startup app
CMD /bin/bash
