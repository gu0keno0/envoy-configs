# envoy-configs

1. Start Fortio Server, lisenning on UDS sockets for both HTTPS and GRPC
# Start server
rm ~/tmp/fortio-http.sock ~/tmp/fortio-grpc.sock; ~/tmp/fortio server -http-port ~/tmp/fortio-http.sock -redirect-port 9081 -grpc-port ~/tmp/fortio-grpc.sock -payload-file ./payloads/tiny
# Test if server is up
curl -vvv -X POST --data '@./envoy-configs/payloads/tiny' --unix-socket ~/tmp/fortio-http.sock http://localhost/

2. Start Fortio Client (WITHOUT Envoy!)
# Start gRPC client, connecting to local UDS socket for sending gRPC load
~/tmp/fortio load -no-reresolve -uniform -k -grpc -ping -qps 1000 -c 1 -n 10000 -payload-file ./envoy-configs/payloads/tiny  unix:/home/jefjiang/tmp/fortio-grpc.sock 
# Start HTTP client (HTTP1.1), connecting to local UDS socket for sending HTTP1.1 load
~/tmp/fortio load -no-reresolve -uniform -k -ping -qps 1000 -c 1 -n 10000 -payload-file ./envoy-configs/payloads/tiny -unix-socket /home/jefjiang/tmp/fortio-http.sock http://localhost
