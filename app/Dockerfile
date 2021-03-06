FROM quay.io/ukhomeofficedigital/centos-base:v0.5.12

ENV USERMAP_UID 1000
ENV PYTHONPATH="$PYTHONPATH:/ADT"

# Setup folder structure and copy files
RUN mkdir -p \
             /tmp/ADT/scripts \
             /tmp/ADT/data/acl \
             /tmp/ADT/stage/acl \
             /tmp/ADT/quarantine/acl \
             /tmp/ADT/log \
             /tmp/ADT/bin \
             /ADT \
             /home/runner/.pm2 \
             /home/runner/.ssh

COPY packages.txt /tmp/ADT/scripts
COPY ecosystem.config.js /tmp/ADT/scripts
COPY docker-entrypoint.sh /
ADD bin /tmp/ADT/bin
ADD scripts /tmp/ADT/scripts

# Add user
RUN groupadd -r runner && \
    useradd --no-log-init -u $USERMAP_UID -r -g runner runner && \
    groupadd docker && \
    usermod -aG docker runner && \
    chown -R runner:runner /ADT && \
    chown -R runner:runner /home/runner/.pm2 && \
    chown -R runner:runner /home/runner/.ssh

# Install script dependencies
RUN yum update -y
RUN yum install -y wget make gcc openssl-devel zlib-devel pcre-devel bzip2-devel libffi-devel epel-release

# Install Python3.7.2 and pip modules
RUN cd /usr/bin && \
    wget https://www.python.org/ftp/python/3.7.2/Python-3.7.2.tgz && \
    tar xzf Python-3.7.2.tgz && \
    cd Python-3.7.2 && \
    ./configure --enable-optimizations && \
    make altinstall && \
    alternatives --install /usr/bin/python python /usr/local/bin/python3.7 1
RUN python -m pip install --no-cache-dir -r /tmp/ADT/scripts/packages.txt

# Fix yum installer with Python3.7 running as a global default
RUN sed -i '/#!\/usr\/bin\/python/c\#!\/usr\/bin\/python2.7' /usr/bin/yum && \
    sed -i '/#! \/usr\/bin\/python/c\#! \/usr\/bin\/python2.7' /usr/libexec/urlgrabber-ext-down

# Install PM2
RUN curl -sL https://raw.githubusercontent.com/nodesource/distributions/master/rpm/setup_11.x -o setup_11.x && bash setup_11.x && rm setup_11.x && \
    yum install -y nodejs && \
    npm install -g pm2

VOLUME ["/ADT"]
RUN chown -R runner:runner /ADT

RUN chown runner:runner /docker-entrypoint.sh && \
chmod +x /docker-entrypoint.sh

USER ${USERMAP_UID}

WORKDIR /ADT

ENTRYPOINT ["/docker-entrypoint.sh"]

# Start PM2
CMD pm2-docker /ADT/scripts/ecosystem.config.js  -- --config $ACL_SERVER $ACL_USERNAME $ACL_PASSWORD $ACL_LANDING_DIR $S3_BUCKET_NAME $S3_KEY_PREFIX \
    $S3_ACCESS_KEY_ID $S3_SECRET_ACCESS_KEY $SECONDARY_S3_BUCKET_NAME $SECONDARY_S3_ACCESS_KEY_ID $SECONDARY_S3_SECRET_ACCESS_KEY $CLAMAV_URL $CLAMAV_PORT \
    $ACL_RDS_HOST $ACL_RDS_DATABASE $ACL_RDS_USERNAME $ACL_RDS_PASSWORD $ACL_RDS_TABLE $SLACK_WEBHOOK

# Save PM2 configuration
RUN pm2 save
