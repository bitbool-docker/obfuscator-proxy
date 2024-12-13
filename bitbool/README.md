# obfuscator proxy

This is using the code from https://github.com/Yawning/obfs4 in order to build an obfuscating proxy, with main target being VPN usage. 
The target here is not privacy, nor for the client to bypass deep inspected networks. 
The target is to be able to connect *to* vpn server behing running behind deep inspection.

## Changes to the code

I added:
* A static port for the obfsproxy client (10080)
* The capability to provide the obfsproxy sevrer certificate at the obfsproxy client instead of the actual client software (socks client). This means that anyone can use the proxy in order to connect to the target server. Protect the proxy if this is not desired. 

## Build obfsproxy

```
docker run -it --rm -v $PWD:/app -w /app golang:1.23.4-alpine3.21 go build -o obfs4proxy/obfs4proxy ./obfs4proxy
```

## Build container

```
docker build -t registry.hub.docker.com/bitbool/obfs4proxy -f bitbool/Dockerfile .
```

# Instructions

Assuming:
* We want to obfuscate traffic to `TARGET_SERVICE:TARGET_PORT`
* obfsproxy server is exposed at `OBFS_SERVER_IP:OBFS_SERVER_PORT`
* obfsproxy client is exposed at `OBFS_CLIENT_IP:OBFS_CLIENT_PORT`

## Run obfsproxy server
```
docker run -it --rm -v $SOME_PATH:/var/lib/tor/pt_state/obfs4 -p 12345:12345 -e TOR_PT_SERVER_TRANSPORTS=obfs4 -e TOR_PT_ORPORT=$TARGET_SERVICE:$TARGET_PORT  bitbool/obfs4proxy
```
When obfsproxy server start it outputs its certificate, e.g.:
```
obfs-server-1  | SMETHOD obfs4 [::]:12345 ARGS:cert=LlzmEvXy2kal8lUXcI1Kq/TkZjjzIAdmP1YaSBMCYEeXI3myxomRRlp1SQyZLyuZ1xvIPw,iat-mode=0
```
Keep the `LlzmEvXy2kal8lUXcI1Kq/TkZjjzIAdmP1YaSBMCYEeXI3myxomRRlp1SQyZLyuZ1xvIPw` part, it is needed at the client, referred to as `OBFS4_SERVER_CERTIFICATE`

if `iat-mode` is not 0, then you need to also set at the client the var `OBFS4_SERVER_IAT_MODE` with the proper value.

You can use the `TOR_PT_SERVER_BINDADDR` variable to change the internal listening port if desired.


## Run obfsproxy client
```
docker run -it --rm -v $SOME_PATH:/var/lib/tor/pt_state/obfs4 -p 10080:10080 -e TOR_PT_CLIENT_TRANSPORTS=obfs4 -e OBFS4_SERVER_KEY="$OBFS4_SERVER_CERTIFICATE"   bitbool/obfs4proxy
```

## Configure openvpn

1. Change the `remote` address:port to the obfsproxy server's `OBFS_SERVER_IP:OBFS_SERVER_PORT`
2. Add a line pointing to the obfsproxy client.
```
socks-proxy OBFS_CLIENT_IP:OBFS_CLIENT_PORT
```

# Architecture

The obfsproxy server is supposed to run next to the service for which we want to obfuscate traffic. 

The obfsproxy client is supposed to run to a point after which we ant to obfuscate traffic (e.g. our premises)

Traffic is obfuscated between the obfsproxy server and client.


## Misc 

### OpenVPN socks auth
If you want to use the original obfsproxy with socks auth at the socks client you need to set the socks-proxy directive in openvpn as
```
socks-proxy OBFS_CLIENT_IP:OBFS_CLIENT_PORT path_to_creds_file
```
where `path_to_creds_file` contains:
```
cert=$OBFS4_SERVER_CERTIFICATE;iat-mode=
0
```
(notice the `;`, obfsproxy server show this at the output with `,`)

