[![Build Status](https://travis-ci.org/arempter/gargoyle-s3proxy.svg?branch=master)](https://travis-ci.org/arempter/gargoyle-s3proxy)
[![codecov.io](http://codecov.io/github/arempter/gargoyle-s3proxy/coverage.svg?branch=master)](https://codecov.io/gh/arempter/gargoyle-s3proxy?branch=master)
[![](https://images.microbadger.com/badges/image/arempter/gargoyle-s3proxy:master.svg)](https://microbadger.com/images/arempter/gargoyle-s3proxy:master)
[![](https://images.microbadger.com/badges/version/arempter/gargoyle-s3proxy:master.svg)](https://microbadger.com/images/arempter/gargoyle-s3proxy:master)

# Gargoyle S3Proxy

Template for s3 proxy based on Akka Http.

gargoyle-s3proxy acts as a security layer between s3 user (eg. application using aws sdk) and s3 backend (eg. ceph RadosGW).
It interacts with
* [Session API](https://github.com/kr7ysztof/auth-api) to get users (or KeyCloak)
* [Ranger](https://ranger.apache.org/) to get bucket policies. On ranger, custom [ranger plugin](https://github.com/bolkedebruin/rangers3plugin) is used to define policies. 
* actual S3 Backend

## usage
To run it:
* Set Environment vars
```
-DPROXY_HOST=proxy_host - name or IP of the proxy server 
-DPROXY_PORT=port       - port to listen on
-DAWS_ACCESS_KEY=       - admin credentials on Ceph RadosGW
-DAWS_SECRET_KEY=       - admin secret on Ceph RadosGW   
-DS3_ENDPOINT=          - RadosGW (or S3 backend) endpoint, eg. http://localhost:8080
```
* sbt run

## Local Testing

To test the proxy locally, we need a running S3 API. For this we use a CEPH docker container with Rados Gateway.
The `docker-compose.yml` defines this, run it using:

    docker-compose up

## Setting Up AWS CLI

It is possible to set up the AWS command-line tools for working with Ceph RadosGW and Gargoyle. The following instructions assume that you have `virtualenv_wrapper` installed.

1. Create an environment for this work:

       % mkvirtualenv -p python3 gargoyle

2. Install the AWS command-line tools and the endpoint plugin:

       % pip install awscli awscli-plugin-endpoint

3. Configure profiles and credentials for working with Gargoyle or the RadosGW directly:

       % mkdir -p ~/.aws
       % cat >> ~/.aws/credentials << EOF
       [radosgw]
       aws_access_key_id = accesskey
       aws_secret_access_key = secretkey

       [gargoyle]
       aws_access_key_id = accesskey
       aws_secret_access_key = secretkey
       EOF
       % cat >> ~/.aws/config << EOF
       [plugins]
       endpoint = awscli_plugin_endpoint

       [profile gargoyle]
       output = json
       region = localhost
       s3 =
           endpoint_url = http://localhost:8080/
       s3api =
           endpoint_url = http://localhost:8080/
       sts =
           endpoint_url = http://localhost:7080/

       [profile radosgw]
       output = json
       region = localhost
       s3 =
           endpoint_url = http://localhost:8010/
       s3api =
           endpoint_url = http://localhost:8010/
       EOF

4. Configure the default profile and reactivate the virtual environment:

       % cat >> ${WORKON_HOME:-$HOME/.virtualenvs}/gargoyle/bin/postactivate << EOF
       AWS_DEFAULT_PROFILE=gargoyle
       export AWS_DEFAULT_PROFILE
       EOF
       % cat >> ${WORKON_HOME:-$HOME/.virtualenvs}/gargoyle/bin/predeactivate << EOF
       unset AWS_DEFAULT_PROFILE
       EOF
       % deactivate
       % workon gargoyle

By default S3 and STS commands will now be issued against the proxy service. For example:

    % aws s3 ls

Commands can also be issued against the underlying RadosGW service:

    % aws --profile radosgw s3 ls

The default profile can also be switched by modifying the `AWS_DEFAULT_PROFILE` environment variable.
