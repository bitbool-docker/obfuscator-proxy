FROM golang:1.23.4-alpine3.21 as builder

COPY . /app

WORKDIR /app

RUN go build -o obfs4proxy/obfs4proxy ./obfs4proxy


FROM alpine:3

COPY --from=builder /app/obfs4proxy/obfs4proxy /usr/local/bin/obfs4proxy 

ENV TOR_PT_MANAGED_TRANSPORT_VER=1
ENV TOR_PT_STATE_LOCATION=/var/lib/tor/pt_state/obfs4
#ENV TOR_PT_EXIT_ON_STDIN_CLOSE=1

RUN mkdir -p $TOR_PT_STATE_LOCATION

WORKDIR $TOR_PT_STATE_LOCATION

RUN echo -e "#!/bin/sh\nexec /usr/local/bin/obfs4proxy -enableLogging=true -logLevel=\$OBFS4_LOG_LEVEL \$OBFS4_UNSAFE_LOGGING" > /obfs4proxy.sh; chmod a+x /obfs4proxy.sh

CMD ["/obfs4proxy.sh"]

ENV OBFS4_LOG_LEVEL=INFO
ENV OBFS4_UNSAFE_LOGGING=-unsafeLogging

#for server
#ENV TOR_PT_SERVER_TRANSPORTS=obfs4
#ENV TOR_PT_ORPORT=my_vpn_server:port ## target service

#for client
#ENV TOR_PT_CLIENT_TRANSPORTS=obfs4
#ENV OBFS4_SERVER_KEY=$OBFS4_SERVER_CERTIFICATE
ENV OBFS4_SERVER_IAT_MODE=0

ENV TOR_PT_SERVER_BINDADDR=obfs4-0.0.0.0:12345


