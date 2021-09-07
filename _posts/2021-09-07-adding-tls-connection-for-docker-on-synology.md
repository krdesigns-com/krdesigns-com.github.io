---
title: Adding TLS connection for Docker on Synology
tags: [home-assistant, HA, Synology, Supervised, Monitor Docker, Plugin, HACS, TLS, Secure, WUD]
style: border
color: primary
description: Allowing secure remote connection (TLS) to your Synology Docker.
---
Source: [KRDesigns.com blogs](https://www.krdesigns.com), [Portainer](https://hub.docker.com/r/portainer/portainer-ce), [Whats up docker](https://github.com/fmartinou/whats-up-docker), [Original post on making the TLS by mwolter](https://community.home-assistant.io/t/whats-up-docker-how-to-keep-your-containers-up-to-date/168315/4)


## SOFTWARE
- TLS certificate
- root access to your Synology via ssh

## Why this guide?
I need to monitored my docker using portainer and gather some docker data monitoring to my [Home Assistant](https://www.home-assistant.io) I install multiple docker on multiple VM/Machine to devided my system allowing faster data recovery and avoid multiple application turn off at the same time. So, after many months trying to find the right way to do it, I finally found it and I would share to you guys.

For Linux machine is very easy since it can run using the script, so I will share a scripts made by mwolter. However for those who are running synology will not being able to run the scripts since synology have their own custom docker and so I will give you detail on how to do it.

## How do I do it:
- Make sure you build your certificate by running this scripts made by mwolter. To create the script simply do `mkdir certs && cd certs` then you can used any linux editor such as `nano certs_create.sh`
and then paste:
```bash
#!/usr/bin/env bash

export CA_KEY=${CA_KEY-"ca-key.pem"}
export CA_CERT=${CA_CERT-"ca.pem"}
export CA_SUBJECT=${CA_SUBJECT:-"test-ca"}
export CA_EXPIRE=${CA_EXPIRE:-"3650"}

export SSL_CONFIG=${SSL_CONFIG:-"openssl.cnf"}
export SSL_KEY=${SSL_KEY:-"key.pem"}
export SSL_CSR=${SSL_CSR:-"key.csr"}
export SSL_CERT=${SSL_CERT:-"cert.pem"}
export SSL_SIZE=${SSL_SIZE:-"4096"}
export SSL_EXPIRE=${SSL_EXPIRE:-"3650"}

export SSL_SUBJECT=${SSL_SUBJECT:-"example.com"}
export SSL_DNS=${SSL_DNS}
export SSL_IP=${SSL_IP}

# Print IP v4 addresses, select only the interface and address, exclude certain interfaces, remove everything except the IP address, replace new line characters \n with comma (,), remove comma from end of string if exists

# name for docker daemon file
export DAEMON=${DAEMON:-"daemon.json"}

export K8S_NAME=${K8S_NAME:-"omgwtfssl"}
export K8S_NAMESPACE=${K8S_NAMESPACE:-"default"}
export K8S_SAVE_CA_KEY=${K8S_SAVE_CA_KEY}
export K8S_SAVE_CA_CRT=${K8S_SAVE_CA_CRT}
export K8S_SHOW_SECRET=${K8S_SHOW_SECRET}

export OUTPUT=${OUTPUT:-"yaml"}

echo "Generating new config file ${SSL_CONFIG}"

# needed for k8s cert-manager compatibility
echo "
[ v3_ca ]
basicConstraints = critical,CA:TRUE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer:always
" >> /etc/ssl/openssl.cnf

[[ -z $SILENT ]] && echo "--> Certificate Authority"

if [[ -e ./${CA_KEY} ]]; then
    [[ -z $SILENT ]] && echo "====> Using existing CA Key ${CA_KEY}"
else
    [[ -z $SILENT ]] && echo "====> Generating new CA key ${CA_KEY}"
    openssl genrsa -out ${CA_KEY} ${SSL_SIZE} > /dev/null
fi

if [[ -e ./${CA_CERT} ]]; then
    [[ -z $SILENT ]] && echo "====> Using existing CA Certificate ${CA_CERT}"
else
    [[ -z $SILENT ]] && echo "====> Generating new CA Certificate ${CA_CERT}"
    openssl req -x509 -new -nodes -key ${CA_KEY} -days ${CA_EXPIRE} -out ${CA_CERT} -extensions v3_ca -subj "/CN=${CA_SUBJECT}" > /dev/null  || exit 1
fi

cat > ${SSL_CONFIG} <<EOM
[ v3_ca ]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer:always
basicConstraints = CA:true
[req]
x509_extensions = v3_ca
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_ca ]
basicConstraints = critical,CA:TRUE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer:always
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, serverAuth
EOM


ips=0

if [[ -n ${SSL_DNS} || -n ${SSL_IP} ]]; then
    cat >> ${SSL_CONFIG} <<EOM
subjectAltName = @alt_names
[alt_names]
EOM

	IFS=","
	dns=(${SSL_DNS})
	dns+=(${SSL_SUBJECT})
	for i in "${!dns[@]}"; do
	  echo DNS.$((i+1)) = ${dns[$i]} >> ${SSL_CONFIG}
	done

	if [[ -n ${SSL_IP} ]]; then
	    ip=(${SSL_IP})
	    for i in "${!ip[@]}"; do
	      echo IP.$((i+1)) = ${ip[$i]} >> ${SSL_CONFIG}
	    done
	fi
fi

[[ -z $SILENT ]] && echo "--> Certificate Authority"

if [[ -e ./${CA_KEY} ]]; then
    [[ -z $SILENT ]] && echo "====> Using existing CA Key ${CA_KEY}"
else
    [[ -z $SILENT ]] && echo "====> Generating new CA key ${CA_KEY}"
    openssl genrsa -out ${CA_KEY} ${SSL_SIZE} > /dev/null
fi

if [[ -e ./${CA_CERT} ]]; then
    [[ -z $SILENT ]] && echo "====> Using existing CA Certificate ${CA_CERT}"
else
    [[ -z $SILENT ]] && echo "====> Generating new CA Certificate ${CA_CERT}"
    openssl req -x509 -new -nodes -key ${CA_KEY} -days ${CA_EXPIRE} -out ${CA_CERT} -subj "/CN=${CA_SUBJECT}" -config ${SSL_CONFIG} > /dev/null  || exit 1
fi

[[ -z $SILENT ]] && echo "====> Generating new SSL KEY ${SSL_KEY}"
openssl genrsa -out ${SSL_KEY} ${SSL_SIZE} > /dev/null || exit 1

[[ -z $SILENT ]] && echo "====> Generating new SSL CSR ${SSL_CSR}"
openssl req -new -key ${SSL_KEY} -out ${SSL_CSR} -subj "/CN=${SSL_SUBJECT}" -config ${SSL_CONFIG} > /dev/null || exit 1

[[ -z $SILENT ]] && echo "====> Generating new SSL CERT ${SSL_CERT}"
openssl x509 -req -in ${SSL_CSR} -CA ${CA_CERT} -CAkey ${CA_KEY} -CAcreateserial -out ${SSL_CERT} \
    -days ${SSL_EXPIRE} -extensions v3_req -extfile ${SSL_CONFIG} > /dev/null || exit 1


# cleanup unnecessary files
# rm -v key.csr openssl.cnf

# permissions to protect the keys
chmod -v 0400 ca-key.pem key.pem

# create daemon file
echo "
{
    \"hosts\": [\"unix:///var/run/docker.sock\", \"tcp://0.0.0.0:2375\"],
    \"tls\": true,
    \"tlscacert\": \"$(pwd)/${CA_CERT}\",
    \"tlscert\": \"$(pwd)/${SSL_CERT}\",
    \"tlskey\": \"$(pwd)/${SSL_KEY}\",
    \"tlsverify\": true
}    
" > ${DAEMON}


if [[ -z $SILENT ]]; then
echo "====> Complete"
echo "keys can be found in volume mapped to $(pwd)"
echo
fi
```
save and you are ready to run the scripts to create your certificate and be sure to `chmod +x *`. to make the script executable.

2. Run the command and adding SUBJECT and IP to your certificate by typing `SSL_SUBJECT=dockerhost1.local SSL_IP=192.168.1.12,192.168.1.11 ./certs_create.sh`

3. For Linux machine (ONLY) you can also run this scripts (made my mwolter) to have it setup for docker-proxy-socket to be set. You simply add `nano certs_apply.sh` inside the certs directory and then paste
```bash
#!/bin/bash
set -ex

mkdir -p /etc/systemd/system/docker.service.d
cat > /etc/systemd/system/docker.service.d/docker.conf <<EOM
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd
EOM

cp daemon.json /etc/docker/

systemctl daemon-reload
systemctl restart docker
```
Save and you should again `chmod +x certs_apply.sh` to make the script executable 

4. Run the certs_apply.sh to setup your linux machine and restart docker.

5. For Synology we will have to have a different approach all together since the certs_apply.sh did not work (well at least at the time being, since I have not have time to modified the script) 

6. First of all you need to stop your docker from you synology control panel. 

7. SSH to your synology and be sure you have a root access

8. go and edit `/var/packages/Docker/etc/dockerd.json'

9. You need to add the five line to make it work
```bash
   "hosts": ["unix:///var/run/docker.sock", "tcp://0.0.0.0:2375"],
   "tls" : true,
   "tlscacert": "/volume1/docker/certs/ca.pem",
   "tlscert": "/volume1/docker/certs/cert.pem",
   "tlskey": "/volume1/docker/certs/key.pem",
   "tlsverify": true,
```
in my case I store my certs on `/volume1/docker/certs` directory so I will get a easy access via synology control panel.

Be sure to save it and you are almost done.

10. Restart your docker once again and TLS is now up and running. Thats it.

## How do I access my docker remotely?
1. Install [portainer](https://www.portainer.io/) in one of your machine.
2. Go to "Settings >> Endpoints >> Add endpoint"
3. Fill the necessary info like IP and upload your created TLS
![pic 1](https://raw.githubusercontent.com/krdesigns-com/krdesigns-com.github.io/master/img/portaine1.png "Pic 1")
4. Add endpoint and thats it.
5. Go to Home and you should see the remote docker at your disposal

## Closing
I hope this guide will help you with secure remote docker access and maybe I'll update more detail on how to add your docker info into [Home-Assistant](https://www.home-assistant.io). 
