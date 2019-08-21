FROM freeradius/freeradius-server:3.0.19-alpine

RUN apk add --no-cache --update postgresql-dev postgresql-client jq && \
    rm -rf /var/cache/apk/* /tmp/*

CMD ["sh", "init_command.sh"]
EXPOSE 1812/udp 1813/udp

COPY ./common/init_command.sh \
    ./common/utils.sh ./
COPY --chown=root:root ./openwisp_freeradius/raddb/ /etc/raddb/

ENV MODULE_NAME=freeradius \
    DB_NAME=openwisp_db \
    DB_USER=admin \
    DB_PASS=admin \
    DB_HOST=postgres \
    DB_PORT=5432 \
    DB_OPTIONS={} \
    FREERADIUS_KEY=testing123 \
    FREERADIUS_CLIENTS=0.0.0.0/0 \
    FREERADIUS_ORGANIZATION=default \
    DASHBOARD_URI=http://dashboard-internal \
    DEBUG_MODE=False