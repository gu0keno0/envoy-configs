# envoy-configs

1. Start Fortio Server, lisenning on UDS sockets for both HTTPS and GRPC
# Start server
rm ~/tmp/fortio-http.sock ~/tmp/fortio-grpc.sock; ~/tmp/fortio server -http-port ~/tmp/fortio-http.sock -redirect-port 9081 -grpc-port ~/tmp/fortio-grpc.sock -payload-file ./payloads/tiny
# Test if server is up
curl -vvv -X POST --data '@./envoy-configs/payloads/tiny' --unix-socket ~/tmp/fortio-http.sock http://localhost/ 
