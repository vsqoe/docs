# OpenLiteSpeed-Setup

## 1. Installation

Choose either of the following methods for setting up OpenLiteSpeed:

1. **Installation from source**: Clone the [github.com/vsqoe/openlitespeed](https://github.com/vsqoe/openlitespeed) repository in your project-directory and follow the following [article](https://docs.openlitespeed.org/installation/source/) for installation from source.

2. **Installation from package-repository**: Follow the following [article](https://docs.openlitespeed.org/installation/repo/) for installation from package-repository.

## 2. Certificate

Follow this [guide](https://medium.com/@bmcrathnayaka/manually-generating-free-ssl-certificates-with-lets-encrypt-certbot-02d9240b85a3) to get a free Let’s Encrypt SSL certificate.

## 3. HTTPS-Setup

1. Generate admin-credentials through `sudo /usr/local/lsws/admin/misc/admpass.sh`.
2. Access the admin-console via `http://localhost:7080/login.php`.
3. Follow this [guide](https://docs.openlitespeed.org/security/ssl/) to setup SSL at listener and virtual-host-level. This is crucial for HTTP3. *Make sure to add our certificate and private key file paths in SSL configuration.*