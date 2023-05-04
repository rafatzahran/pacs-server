## NEO PACS SERVER

>**PS!** This project is cloned from [zahiva.ai](https://github.com/zhiva-ai/pacs-server.git) and the config files are updated to suit NeoMedSys test environment.

### Requirements

Before we start please make sure your server has access to:

- Unix-base shell
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [OpenSSL](https://www.openssl.org/)

### Get the server code

You can either clone this repo
```shell
git clone https://github.com/rafatzahran/pacs-server.git
```

### Generate local TSL certificate (only once every 365 days)

> Notice
> 
> If you already have the `.crt` and `.key` files do not generate them again.

Before we start storing data on the server we have to secure the connection between the server and the viewer. We're going to use [Transport Layer Security (TSL)](https://en.wikipedia.org/wiki/Transport_Layer_Security). First, we have to generate certificate (`.crt`) and key (`.key`) files. Because we're doing it from the local server we're going to use [OpenSSL](https://www.openssl.org/) (we don't need external party to sign certificate, if you want to use it on the external server you should sign with something like [Let's Encrypt](https://letsencrypt.org/)).

To generate local certificate please execute following command in the terminal window (make sure `openssl` is installed).

```shell
openssl req -x509 \
    -nodes -days 365 \
    -newkey rsa:2048 \
    -keyout neo-pacs.key \
    -out neo-pacs.crt \
    -subj "/C=NO/L=Oslo/O=NEO, Inc./CN=localhost"
```

Change `-subj` to describe your organization:
- `C` - country code (2 letter code)
- `L` - location (city name)
- `O` - organization name
- `CN` - full domain name (use `localhost` for local servers)

Then copy `.crt` file as trusted certificate:

```shell
cat neo-pacs.crt > trusted.crt
```
At this point you should have 2 `.crt` files and one `.key` file
#### Build the server

```shell
docker compose up
```

After starting the server you should be able to access [localhost/neoserv/app/explorer.html#upload](https://localhost/neoserv/app/explorer.html#upload) where you can upload your files.

> Important!!!
> 
> You might be prompted with the message about invalid SSL certificate. This is caused by using OpenSSL to generate certificate for `localhost` and that certificate has no 3rd party that confirms its authenticity. It's fine for local network but remember to use proper certificate if the server is accessible from outside your network.

### Access from within internal network

If you have more than one computer inside your network (or VPN connection), then you can share the server settings with them. To check the server address please run the following command:

Linux or Mac:
```shell
ifconfig
```

Windows
```shell
ipconfig
```

and look for the setting with the `inet` value that starts with `192.168.`. That should by your address in the local network. You should be able to access the upload page from `192.168.x.x/neoserv/app/explorer.html`. 
### Upload your DICOMs

Go to [localhost/neoserv/app/explorer.html#upload](https://localhost/neoserv/app/explorer.html#upload) and click on `Select files to upload`.

![Upload dicoms screenshot](./upload-dicoms.png)

After selecting all the DICOMs click `Start the upload` to store them on the server.

All DICOMs are stored in [Docker's persistent volume](https://docs.docker.com/storage/volumes/) so even after restarting the server all your files are still accessible.

### FAQs

- _"Why my server doesn't accept very large DICOM files?"_

There is an maximum file size setting inside server config. If you want to change that please go to `./nginx.conf` and modify:
```nginx configuration
client_max_body_size 2000m;
```

### References
>**PS!** This project is cloned from [zahiva.ai](https://github.com/zhiva-ai/pacs-server.git) and the config files are updated to suit NeoMedSys test environment.

- [zhiva.ai](https://docs.zhiva.ai/latest)