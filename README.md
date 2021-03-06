# SafeDNS Authenticator plugin for Certbot

## Quickstart
```bash
docker run -it -v $(pwd)/safedns.ini:/safedns.ini -v /etc/letsencrypt:/etc/letsencrypt -v /var/lib/letsencrypt:/var/lib/letsencrypt ukfast/certbot-dns-safedns:latest <certbot args>
```

## Setup

```bash
apt install certbot python3-pip
pip3 install --upgrade certbot-dns-safedns
```

## Execution

```bash
certbot certonly --authenticator dns_safedns
```

> **Warning**: certbot might tell you that it doesn't have permissions to write to its log file. However, if you run certbot as sudo, you won't have access to the safedns plugin if you didn't install the plugin as sudo.

This will result in the following error from certbot:

```bash
Could not choose appropriate plugin: The requested dns_safedns plugin does not appear to be installed
```

To get around this just do:

```bash
sudo pip3 install --upgrade certbot-dns-safedns
sudo certbot certonly --authenticator dns_safedns
```

If you get any python cryptography errors, such as:

```bash
ContextualVersionConflict: ...
```

just make sure to upgrade your pyopenssl.

```bash
sudo pip install --upgrade pyopenssl
```

### Credentials and Config Options

Use of this plugin can be simplified by using a configuration file containing SafeDNS API credentials, obtained from your MyUKFast [account page](https://my.ukfast.co.uk/applications/index.php). See also the [SafeDNS API](https://developers.ukfast.io/documentation/safedns) documentation.

An example ``safedns.ini`` file:

```ini
dns_safedns_auth_token = 0123456789abcdef0123456789abcdef01234567
dns_safedns_propagation_seconds = 20
```

The path to this file can be provided interactively or using the `--dns_safedns-credentials` command-line argument. Certbot records the path to this file for use during renewal, but does not store the file's contents.

> **CAUTION:** You should protect these API credentials as you would the password to your MyUKFast account. Users who can read this file can use these credentials to issue arbitrary API calls on your behalf. Users who can cause Certbot to run using these credentials can complete a ``dns-01`` challenge to acquire new certificates or revoke existing certificates for associated domains, even if those domains aren't being managed by this server.

Certbot will emit a warning if it detects that the credentials file can be accessed by other users on your system. The warning reads "Unsafe permissions on credentials configuration file", followed by the path to the credentials file. This warning will be emitted each time Certbot uses the credentials file, including for renewal, and cannot be silenced except by addressing the issue (e.g., by using a command like `chmod 600` to restrict access to the file).

### Examples

To acquire a single certificate for both `example.com` and `*.example.com`, waiting 900 seconds for DNS propagation:

```bash
certbot certonly \
  --authenticator dns_safedns \
  --dns_safedns-credentials ~/.secrets/certbot/safedns.ini \
  --dns_safedns-propagation-seconds 900 \
  -d 'example.com' \
  -d '*.example.com'
```

## Build

The package for the safedns plugin is hosted on pypi here: <https://pypi.org/project/certbot-dns-safedns/>

To build and upload the package from source, first ensure you've increased the version number in ```setup.py```.

Delete the ```build``` ```dist``` and ```.egg-info``` dirs if they are present from a previous build.

Then run:

```bash
python3 setup.py sdist bdist_wheel
```

## Deployment

```bash
python3 -m twine upload dist/*
```

> **Warning**: Use the username: `__token__`, along with the token registered on pypi.
