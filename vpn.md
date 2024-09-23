## Setting up OpenVPN with Easy-RSA

#### Scenario:
You want to set up a **VPN server** using OpenVPN to allow multiple clients to connect securely to your private network. For secure communication, you need to create a **Certificate Authority (CA)**, issue a **server certificate**, and generate **client certificates**.

We'll use **Easy-RSA** to handle the creation of the necessary certificates and keys.

---

### Steps for Setting up OpenVPN with Easy-RSA:

#### Prerequisites:
1. A Linux server (e.g., Ubuntu 20.04 or CentOS) with **root access**.
2. OpenVPN and Easy-RSA installed on the server.
3. Clients that will connect to the OpenVPN server (e.g., Windows, macOS, Linux devices).

---

### Step 1: Install OpenVPN and Easy-RSA

On your VPN server, install both **OpenVPN** and **Easy-RSA**.

#### Ubuntu/Debian:
```bash
sudo apt update
sudo apt install openvpn easy-rsa
```

#### CentOS/RHEL:
```bash
sudo yum install epel-release
sudo yum install openvpn easy-rsa
```

---

### Step 2: Set Up the PKI Environment with Easy-RSA

Easy-RSA is used to generate the **Certificate Authority (CA)**, server certificates, and client certificates.

#### 1. Initialize the Easy-RSA Directory
```bash
make-cadir ~/easy-rsa
cd ~/easy-rsa
```

This command creates a working directory for Easy-RSA and moves into it.

#### 2. Initialize the Public Key Infrastructure (PKI)
Initialize the PKI structure where the keys and certificates will be stored:
```bash
./easyrsa init-pki
```

#### 3. Build the Certificate Authority (CA)
Create your own **Certificate Authority (CA)**, which will be used to sign certificates.
```bash
./easyrsa build-ca
```

This will prompt you for some information like the common name (use something identifiable like "MyVPN CA"). You will also be asked to create a **CA private key password** (keep this safe, as it is needed to issue certificates).

---

### Step 3: Generate the Server Certificate and Key

Now, generate the server certificate and private key, which will be used by the OpenVPN server to identify itself.

#### 1. Create the Server Request
Run the following to generate a certificate signing request (CSR) for the server:
```bash
./easyrsa gen-req server nopass
```

- The `server` is the name of the request.
- `nopass` means no password is required for the server private key.

#### 2. Sign the Server Certificate
Sign the server's CSR with the CA to create the server certificate:
```bash
./easyrsa sign-req server server
```

When prompted, confirm that you’re signing the server request.

#### 3. Generate Diffie-Hellman Parameters
The Diffie-Hellman key exchange allows secure key negotiation over an insecure channel:
```bash
./easyrsa gen-dh
```

This generates a `dh.pem` file, which will be used by OpenVPN.

---

### Step 4: Generate Client Certificates and Keys

Each client (e.g., users on different devices) needs its own certificate and key to connect to the VPN securely.

#### 1. Create a Certificate Request for the Client
Repeat this for each client, replacing `client1` with the client's name:
```bash
./easyrsa gen-req client1 nopass
```

#### 2. Sign the Client Certificate
Sign the client’s CSR with the CA to generate the certificate:
```bash
./easyrsa sign-req client client1
```

#### 3. (Optional) Revoke Client Certificates
If a client certificate is compromised, you can revoke it with:
```bash
./easyrsa revoke client1
```

Then, regenerate the **Certificate Revocation List (CRL)**:
```bash
./easyrsa gen-crl
```

---

### Step 5: Configure the OpenVPN Server

Now that you have the server and client certificates and keys, it's time to configure OpenVPN.

#### 1. Copy the Generated Files
Move the server and CA certificates to the OpenVPN directory:

```bash
cp ~/easy-rsa/pki/ca.crt /etc/openvpn/
cp ~/easy-rsa/pki/issued/server.crt /etc/openvpn/
cp ~/easy-rsa/pki/private/server.key /etc/openvpn/
cp ~/easy-rsa/pki/dh.pem /etc/openvpn/
```

#### 2. Create the OpenVPN Server Configuration
OpenVPN requires a configuration file to start. Create a new config file in `/etc/openvpn/server.conf`:
```bash
sudo nano /etc/openvpn/server.conf
```

Example server configuration:
```bash
port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh.pem
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
keepalive 10 120
cipher AES-256-CBC
user nobody
group nogroup
persist-key
persist-tun
status openvpn-status.log
verb 3
```

This configuration sets up a VPN using **UDP on port 1194** and assigns clients an IP address in the range `10.8.0.0/24`.

#### 3. Enable IP Forwarding
Enable IP forwarding to allow traffic to pass through the VPN server:
```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

To make this change permanent, edit `/etc/sysctl.conf` and uncomment:
```bash
net.ipv4.ip_forward=1
```

#### 4. Configure Firewall Rules (Optional)
Use **iptables** to allow VPN traffic and NAT:
```bash
sudo iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
```

Ensure the firewall allows traffic on port 1194:
```bash
sudo ufw allow 1194/udp
```

---

### Step 6: Start and Enable OpenVPN

Now start the OpenVPN service and enable it to run on boot.

#### 1. Start OpenVPN
```bash
sudo systemctl start openvpn@server
```

#### 2. Enable OpenVPN at Boot
```bash
sudo systemctl enable openvpn@server
```

---

### Step 7: Configure VPN Clients

To connect clients, distribute the necessary configuration files along with the **client certificate and key**.

#### 1. Create Client Config File (client1.ovpn)
On the client machine (or distribute this file):
```bash
client
dev tun
proto udp
remote your_server_ip 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert client1.crt
key client1.key
remote-cert-tls server
cipher AES-256-CBC
verb 3
```

#### 2. Transfer Certificate and Key Files to the Client
The client needs the following files to connect:
- `client1.crt` (client certificate)
- `client1.key` (client key)
- `ca.crt` (CA certificate)
- `client1.ovpn` (OpenVPN config file)

---

### Step 8: Connect Clients to the VPN

On the client machine, install the **OpenVPN client** software (available on most platforms) and import the configuration file (`client1.ovpn`). Then, start the VPN connection.

---

### Conclusion:
You've now set up a fully functional **OpenVPN server** using **Easy-RSA** to manage the necessary certificates and keys. Your server is configured to accept secure connections from clients, who can securely access your private network. Easy-RSA simplifies the otherwise complex process of certificate management, making it easier to deploy OpenVPN and manage users.