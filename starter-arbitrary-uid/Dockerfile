FROM ubuntu:16.04
MAINTAINER Mirek

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

# default startup app
CMD /bin/bash
