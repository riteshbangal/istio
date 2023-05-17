$ kubectl create secret generic external-url-tls --from-file=cert.pem=path/to/certificate.pem --from-file=key.pem=path/to/private-key.pem


$ kubectl create secret generic mywebapp-external-tls-secret --from-file=cert.pem=path/to/certificate.pem --from-file=key.pem=path/to/private-key.pem

[ec2-user@ip-172-31-40-63 ~]$ openssl x509 -inform der -in client.crt -out client.pem
[ec2-user@ip-172-31-40-63 ~]$ openssl rsa -in client.key -out client.pem


$ curl -ksI --cert client.crt --key client.key https://ec2-3-125-207-155.eu-central-1.compute.amazonaws.com/ | grep HTTP


$ kubectl create secret tls external-url-tls --key=path/to/private-key.pem --cert=path/to/certificate.pem

$ kubectl create secret tls mywebapp-external-tls-secret --key=client.key --cert=client.crt

---------------------------------------------------  mywebapp-mtls-credentials-secret ---------------------------------------------------
The name of the secret that holds the TLS certs for the client including the CA certificates. Secret must exist in the same namespace with the proxy using the certificates. The secret (of type generic)should contain the following keys and values: key: <privateKey>, cert: <clientCert>, cacert: <CACertificate>. Here CACertificate is used to verify the server certificate. Secret of type tls for client certificates along with ca.crt key for CA certificates is also supported. Only one of client certificates and CA certificate or credentialName can be specified.

NOTE: This field is applicable at sidecars only if DestinationRule has a workloadSelector specified. Otherwise the field will be applicable only at gateways, and sidecars will continue to use the certificate paths.


Generate a CA Certificate:
[ec2-user@ip-172-31-40-63 ~]$ openssl req -x509 -newkey rsa:4096 -keyout ca.key -out ca.crt -subj "/CN=MyWebAppCA"

# Enter PEM pass phrase: mywebapp

Generate the Server Certificate:
[ec2-user@ip-172-31-40-63 ~]$ openssl genrsa -out server.key 4096
[ec2-user@ip-172-31-40-63 ~]$ openssl req -new -key server.key -out server.csr -subj "/CN=my-web-app"
[ec2-user@ip-172-31-40-63 ~]$ openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365


Generate the Client Certificate:
[ec2-user@ip-172-31-40-63 ~]$ openssl genrsa -out client.key 4096
[ec2-user@ip-172-31-40-63 ~]$ openssl req -new -key client.key -out client.csr -subj "/CN=my-macbook-pro-client"
[ec2-user@ip-172-31-40-63 ~]$ openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days 365



Recreate a Kubernetes Secret:
$ kubectl delete secret  mywebapp-mtls-credentials-secret
$ kubectl create secret generic mywebapp-mtls-credentials-secret \
    --from-file=ca.crt \
    --from-file=server.crt \
    --from-file=server.key \
    --from-file=client.crt \
    --from-file=client.key
