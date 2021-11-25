FROM gogs/gogs:0.12.3

RUN apk add --no-cache jq

ENV GOGS_CUSTOM=""

COPY app.ini ./custom/conf/app.ini
COPY gogs-install.txt .
COPY init.sh .

# this uses the original start script to prep the data folders:
RUN chmod o+w ./custom/conf/app.ini && \
    chmod +x init.sh && ./init.sh

# replace with custom start script:
COPY start.sh ./docker/start.sh
RUN chmod +x ./docker/start.sh