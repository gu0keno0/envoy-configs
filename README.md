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

3. Start Server side Envoy Sidecar For HTTP
# Start Envoy Proxy for HTTP with Trace logging level
./indis-configs/bin/envoy -c envoyconfig_benchmark_static_uds.yaml -l trace

# Curl Test if Envoy works
curl -k -v -X POST --data '@./payloads/tiny' https://localhost:18080/

# Start Fortio Client for HTTP
~/tmp/fortio load -no-reresolve -uniform -k -ping -qps 1000 -c 1 -n 10000 -payload-file ./envoy-configs/payloads/tiny https://localhost:18080

4. Start Server Side Envoy Sidecar For gRPC
# Start Envoy Proxy for gRPC
../indis-configs/bin/envoy -c ./envoy-server-grpc.yaml

# Start Fortio Client for gRPC
~/tmp/fortio load -no-reresolve -uniform -k -grpc -ping -qps 1000 -c 1 -n 10000 -payload-file ./envoy-configs/payloads/tiny  https://localhost:18080/

----

5. Test full HTTP service mesh
# Start HTTP Server
rm ~/tmp/fortio-http.sock ~/tmp/fortio-grpc.sock; ~/tmp/fortio server -http-port ~/tmp/fortio-http.sock -redirect-port 9081 -grpc-port ~/tmp/fortio-grpc.sock -payload-file ./payloads/tiny

# Start Envoy for HTTP Server
../indis-configs/bin/envoy -c ./envoy-server-http.yaml --base-id 0

# Start Envoy for HTTP Client
../indis-configs/bin/envoy -c envoy-client-http.yaml --base-id 1

# Start HTTP Curl Client 
curl -vvv -X POST --data '@./payloads/tiny' --unix-socket /home/jefjiang/tmp/envoy-client-http.sock http://localhost/

# Start HTTP Fortio Client
~/tmp/fortio load -no-reresolve -uniform -k -ping -qps 1000 -c 1 -n 100000 -payload-file ./envoy-configs/payloads/tiny -unix-socket /home/jefjiang/tmp/envoy-client-http.sock http://localhost

----

6. Test full gRPC service mesh
# Start gRPC Server
rm ~/tmp/fortio-http.sock ~/tmp/fortio-grpc.sock; ~/tmp/fortio server -http-port ~/tmp/fortio-http.sock -redirect-port 9081 -grpc-port ~/tmp/fortio-grpc.sock -payload-file ./payloads/tiny

# Start Envoy for gRPC Server
../indis-configs/bin/envoy -c ./envoy-server-grpc.yaml --base-id 0

# Start Envoy for gRPC Client
../indis-configs/bin/envoy -c envoy-client-grpc.yaml --base-id 1

# Start gRPC Fortio Client
~/tmp/fortio load -no-reresolve -uniform -k -grpc -ping -qps 1000 -c 1 -n 100000 -payload-file ./envoy-configs/payloads/tiny unix:/home/jefjiang/tmp/envoy-client-grpc.sock
