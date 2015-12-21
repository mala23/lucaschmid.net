# Let's encrypt Express

Since _[Let's Encrypt][1]_ will be comming out soon, I thought I'd try it for this site. It runs on Node.js using Express on [Alpine Linux][2]. The guide should work for pretty much any Linux System, since both Node.js and _Let’s Encrypt_ are made to be compatible.

## Requesting the certificate

First of all lets get our certificate. I basically just followed the [README in Let's Encrypt's Github repo][3].

Install the utility. This will become easier one it's released. You'll then be able to use your package-manager.

\`\`\`
git clone https://github.com/letsencrypt/letsencrypt
cd letsencrypt
./letsencrypt-auto --help
\`\`\`

Then we can request the certificate. Here's what I did for this site

\`\`\`
./letsencrypt-auto certonly --standalone --email not\_an\_email\_address@lucaschmid.net -d lucaschmid.net
\`\`\`

This threw an error because I had IPv6 enabled. If [this issue][4] hasn't been resolved yet, you might need to do deactivate IPv6, before running the last command.

\`\`\`
sysctl -w net.ipv6.conf.all.disable\_ipv6=1
\`\`\`

Then after you have recieved the cert, you can enable it again.

\`\`\`
sysctl -w net.ipv6.conf.all.disable\_ipv6=0
\`\`\`

## Installing them for the Express App

Inside my app's directory I created a directory called `tls`. Then I created some symlinks for the certificate and the key.

\`\`\`
mkdir tls
cd tls
ln -s /etc/letsencrypt/live/lucaschmid.net/cert.pem
ln -s /etc/letsencrypt/live/lucaschmid.net/privkey.pem key.pem
\`\`\`

(I'm using Docker to run this site, so the symlinks won't work inside the container. To fix this, I had to make copies of the files instead of only symlinking them. This has the disadvantage, that letsencrypt can't manage them.)

Now we can integrate the `https` module into our Express server. Here's a simple example.

\`\`\`
var express = require('express')
var fs = require('fs')
var https = require('https')

var ports = process.env.NODE\_ENV === 'production'
  ? [80, 443]
  : [3442, 3443]

var app = express()

var server = https.createServer(
  {
	key: fs.readFileSync('./tls/key.pem'),
	cert: fs.readFileSync('./tls/cert.pem')
  },
  app
)

server.listen(ports[1])
app.listen(ports[0])

\`\`\`


[1]:	https://letsencrypt.org/
[2]:	https://alpinelinux.org/
[3]:	https://github.com/letsencrypt/letsencrypt/blob/master/README.rst
[4]:	https://github.com/letsencrypt/boulder/issues/1046