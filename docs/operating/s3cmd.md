## S3cmd Setup

Install S3cmd CentOS7
```sh 
yum install epel-release -y 
yum update -y 
yum install s3cmd -y
```

Set Default Configuration

$ s3cmd --configure

s3cmd requests your access and secret keys. Find these values in your customer portal.

Access Key: exampleNMWQSG599TB3A
Secret Key: exampleCL2s4EgQRhnXafSBHCsjlsz1XVfJBeE4V

Type ENTER to accept the default region. Vultr ignores this value.

Default Region [US]:

Enter s3.azunce.xyz for the S3 Endpoint.

S3 Endpoint [s3.amazonaws.com]: s3.azunce.xyz

Enter %(bucket)s.s3.azunce.xyz for the DNS-style template.

DNS-style bucket+hostname:port template for accessing a 
bucket [%(bucket)s.s3.amazonaws.com]: %(bucket)s.s3.azunce.xyz

Optional: Encryption Password

GPG encryption protects objects while stored at Vultr. Setting this password does not automatically encrypt objects; it only makes the option available later. Linux users can usually accept the default path to GPG. macOS users may need to install GPG first, then locate the path with which gpg.

Encryption password: example
Path to GPG program [/usr/bin/gpg]:

Enter ENTER to use HTTPS protocol. Vultr Object Storage requires HTTPS.

Use HTTPS protocol [Yes]:

Optional: If your network requires an HTTP Proxy, enter that here. Otherwise press ENTER.

HTTP Proxy server name:

Press Y + ENTER to test the s3cmd configuration.

Test access with supplied credentials? [Y/n] y
Please wait, attempting to list all buckets...
Success. Your access key and secret key worked fine :-)

Press Y + ENTER to save the .s34cfg file.

Save settings? [y/N] y
Configuration saved to '~/.s3cfg'

## Using S3cmd

Make a bucket.

s3cmd mb s3://mybucket

Remove a bucket.

s3cmd rb s3://mybucket

List the buckets.

s3cmd ls

List the objects in bucket.

s3cmd ls s3://mybucket

Upload a file for private access.

s3cmd put photo.jpg s3://mybucket/photo.jpg

Upload a file for public access.

s3cmd put -P photo.jpg s3://mybucket/photo.jpg

Download a file.

s3cmd get s3://mybucket/photo.jpg

Delete a file.

s3cmd rm s3://mybucket/photo.jpg

Change file permission to public access.

s3cmd setacl s3://mybucket/photo.jpg --acl-public

Change file permission to private access.

s3cmd setacl s3://mybucket/photo.jpg --acl-private

Enable public directory listing for a bucket.

s3cmd setacl s3://mybucket/ --acl-public

Disable public directory listing for a bucket.

s3cmd setacl s3://mybucket/ --acl-private
