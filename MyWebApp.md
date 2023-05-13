Create a Mutual TLS (mTLS) web app on an EC2 instance using Nginx/httpd, you can follow these steps:

##### Launch an EC2 Instance:

- Log in to the AWS Management Console.
- Go to the EC2 service.
- Click on "Launch Instance" to create a new EC2 instance. Use elastic ip, to have a static public IP.
- Choose an appropriate Amazon Machine Image (AMI) based on your requirements.
- Select an instance type, configure networking, and add storage as per your needs.
- Configure security groups to allow inbound traffic on ports 80 (HTTP) and 443 (HTTPS).
- Launch the instance and choose an existing key pair or create a new one to securely connect to the instance.


##### Connect to the EC2 Instance:

- Connect to your EC2 instance using SSH. You can use a tool like PuTTY (Windows) or SSH command-line tool (Unix-based systems).
- Once connected, you will have access to the command line of your EC2 instance.
$ chmod 400 my-web-app.pem
$ ssh -i my-web-app.pem ec2-user@3.125.207.155


##### Install a web server:
If you're using an Amazon Linux 2 AMI:
$ sudo yum update -y
$ sudo yum install httpd -y
Create a file named index.html in the Apache document root directory (/var/www/html/) with some content.
$ sudo service httpd start

Go to http://ec2-3-125-207-155.eu-central-1.compute.amazonaws.com


##### Install and Configure Nginx:
$ sudo yum update
$ sudo yum install nginx
$ sudo service nginx start

Go to http://ec2-3-125-207-155.eu-central-1.compute.amazonaws.com

##### Generate SSL/TLS Certificates: 
To enable mTLS, you need SSL/TLS certificates for your server. You can obtain certificates from a certificate authority (CA) or create self-signed certificates for testing purposes.
Create self-signed certificates for testing purposes. Generate a private key and a certificate signing request (CSR) using OpenSSL. Run the following command:
[ec2-user@ip-172-31-40-63 ~]$ openssl req -newkey rsa:2048 -nodes -keyout server.key -out server.csr

Submit the CSR to a CA or self-sign the certificate if you're using self-signed certificates. Follow the instructions provided by your CA or refer to OpenSSL documentation for self-signing.
[ec2-user@ip-172-31-40-63 ~]$ openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
Certificate request self-signature ok
subject=C = NL, ST = Holland, L = Voorburg, O = Visa, OU = Dev, CN = Test, emailAddress = test@rcb.com


##### Configure Nginx with mTLS:
Copy the server certificate file (e.g., server.crt) and the private key file (e.g., server.key) to a location on your EC2 instance.
Open the Nginx configuration file for editing using a text editor. The default file is located at /etc/nginx/nginx.conf.
Add the following server block to enable HTTPS and mTLS authentication:
server {
    listen 443 ssl;
    server_name your_domain.com;
    ssl_certificate /path/to/your/server.crt;
    ssl_certificate_key /path/to/your/server.key;

    # Enable mTLS
    ssl_client_certificate /path/to/your/client.crt;
    ssl_verify_client on;

    location / {
        # Configure your app's settings here
    }
}
Please take a look to the nginx.conf file.

[ec2-user@ip-172-31-40-63 ~]$ sudo service nginx restart

