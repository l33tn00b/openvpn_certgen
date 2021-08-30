# openvpn_certgen
apt-get install easy-rsa
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz /etc/openvpn/
sudo gunzip /etc/openvpn/server.conf.gz
cd +
cd ~
mkdir openvpn
cd openvpn/
/usr/share/easy-rsa/easyrsa init-pki
cp /usr/share/easy-rsa/vars.example pki/vars
nano pki/vars
/usr/share/easy-rsa/easyrsa build-ca
/usr/share/easy-rsa/easyrsa gen-req openvpn-server nopass
/usr/share/easy-rsa/easyrsa sign-req server openvpn-server
/usr/share/easy-rsa/easyrsa gen-req openvpn-client1 nopass
cd pki
zip -j client1.zip ./issued/openvpn-client1.crt ./private/openvpn-client1.key ca.crt
