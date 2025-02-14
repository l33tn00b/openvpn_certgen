# openvpn_certgen
Setting up the CA and one client.
We're on an older version of easy-rsa, so revoking certificates needs to be done using the old syntax, see: https://github.com/OpenVPN/easy-rsa/blob/master/doc/EasyRSA-Renew-and-Revoke.md  
We'd love to be on a later version (>3.2), but nooo..

# Basic setup
Editing pki/vars: e.g. for setting certificate expiry dates. 
```
apt-get install easy-rsa
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz /etc/openvpn/
sudo gunzip /etc/openvpn/server.conf.gz
cd ~
mkdir openvpn
cd openvpn/
/usr/share/easy-rsa/easyrsa init-pki
cp /usr/share/easy-rsa/vars.example pki/vars
nano pki/vars
```

# Generate CA and Server certificate
still be in openvpn directory!  
```
/usr/share/easy-rsa/easyrsa build-ca
/usr/share/easy-rsa/easyrsa gen-req openvpn-server nopass
/usr/share/easy-rsa/easyrsa sign-req server openvpn-server
```

# Generate Diffie-Hellman Parameter File
```
/usr/share/easy-rsa/easyrsa gen-dh
```

# Generate Client Certificate
and zip it for download
```
/usr/share/easy-rsa/easyrsa gen-req openvpn-client1 nopass
/usr/share/easy-rsa/easyrsa sign-req client openvpn-client1

cd pki
zip -j client1.zip ./issued/openvpn-client1.crt ./private/openvpn-client1.key ca.crt
```
NOTE:
When not renewing a certificate but issuing a new key/crt, the server's client configuration directory must be updated!

# Using the Keys/Certificates  
Make sure to set ownership of files to openvpn user!

# Renew Certificate (easy-rsa > 3.2) (not for us)
- Command expire <NAME>   
- Command sign-req <TYPE> <NAME>
- If required, Command revoke-expired can be used to revoke an expired certificate

# Revoke Certificate
```
/usr/share/easy-rsa/easyrsa revoke openvpn-server
```
NOTE:
- After generating crl it must be copied to openvpn config directory (e.g. `/etc/openvpn/server`)
- server config file must be edited to use crl file (include `crl-verify crl.pem` in config file)
- access rights must be taken into account (i.e. when running openvpn as non-root user, file ownership must (usually) be set to `openvpn:openvpn`)
  - `chown openvpn:openvpn crl.pem`
- The CRL WILL EXPIRE! When it expires, clients will no longer be able to connect! So you'll probably
  - want to make sure to periodically update the crl (e.g. via cron shell script)
  - and/or choose a longer expiry period:
    - `nano ~/openvpn/pki/vars`
    - `set_var EASYRSA_CRL_DAYS        3650`
    - `easyrsa gen-crl`
