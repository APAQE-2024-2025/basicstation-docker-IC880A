# LoRa Basics™ Station for Docker

This project deploys a LoRaWAN gateway with Basics™ Station Packet Forward protocol using Docker or Balena.io. It runs on a PC, a Raspberry Pi 3/4, Compute Module 3/4 or balenaFin with SX1301, SX1302 or SX1303 LoRa concentrators (e.g. RAK831, RAK833, RAK2245, RAK2247, RAK2287, RAK5146, Seeed WM1302 and IMST iC880a among others).


## Introduction

Deploy a LoRaWAN gateway running the Basics™ Station Semtech Packet Forward protocol in a docker container. Also, you can use balena.io and RAK to reduce friction for the LoRa gateway fleet owners. 

The Basics™ Station protocol enables the LoRa gateways with a reliable and secure communication between the gateways and the cloud and it is becoming the standard Packet Forward protocol used by most of the LoRaWAN operators.

This project is available in Docker Hub (https://hub.docker.com/r/xoseperez/basicstation) and GitHub (https://github.com/xoseperez/basicstation).

This project is an evolution of the BasicStation implementation for Docker @mpous and I have been working on. You can still find it here: https://github.com/mpous/basicstation.

This project has been tested with The Things Stack Community Edition (TTSCE or TTNv3).


## Requirements

### Hardware

As long as the host can run docker containers, the Basics™ Station service can run on:

* AMD64: most PCs out there
* ARMv8: Raspberry Pi 3/4, 400, Compute Module 3/4, Zero 2 W,...
* ARMv7: Raspberry Pi 2

> **NOTE**: you will need an OS in the host machine, for some SBC like a Raspberry Pi that means and SD card with an OS (like Rasperry Pi OS) flashed on it.

#### LoRa Concentrators

Supported LoRa concentrators:

* SX1301 (only SPI)
  * [IMST iC880a](https://shop.imst.de/wireless-modules/lora-products/8/ic880a-spi-lorawan-concentrator-868-mhz)
  * [RAK 831 Concentrator](https://store.rakwireless.com/products/rak831-gateway-module)
  * [RAK 833 Concentrator](https://store.rakwireless.com/products/rak833-gateway-module)
  * [RAK 2245 Pi Hat](https://store.rakwireless.com/products/rak2245-pi-hat)
  * [RAK 2247 Concentrator](https://store.rakwireless.com/products/rak2247-lpwan-gateway-concentrator-module)
* SX1302 (SPI or USB)
  * [RAK 2287 Concentrator](https://store.rakwireless.com/products/rak2287-lpwan-gateway-concentrator-module)
  * [Seeed WM1302](https://www.seeedstudio.com/WM1302-LoRaWAN-Gateway-Module-SPI-EU868-p-4889.html)
* SX1303 (SPI or USB)
  * [RAK 5146 Concentrator](https://store.rakwireless.com/collections/wislink-lpwan/products/wislink-lpwan-concentrator-rak5146)

> **NOTE**: The basicstation project is not compatible with SX1301 USB LoRa concentrators. This means that you won't be able to use it with a RAK2247-USB.

> **NOTE**: SPI concentrators in MiniPCIe form factor will require a special Hat or adapter to connect them to the SPI interface in the SBC. USB concentrators in MiniPCIe form factor will require a USB adapter to connect them to a USB2/3 socket on the PC or SBC. Other form factors might also require an adaptor for the target host.

> **NOTE**: Other concentrators might also work. If you manage to make this work with a different setup, report back :)


### Software

If you are going to use docker to deploy the project, you will need:

* An OS running your host (Linux or MacOS for AMD64 systems, Raspberry Pi OS, Ubuntu OS for ARM,...)
* Docker (and optionally docker-compose) on the machine (see below for installation instructions)

If you are going to use this image with Balena, you will need:

* A balenaCloud account ([sign up here](https://dashboard.balena-cloud.com/))

On both cases you will also need:

* A The Things Stack V3 account [here](https://ttc.eu1.cloud.thethings.industries/console/)
* [balenaEtcher](https://balena.io/etcher) to burn the OS image on the SD card of eMMC for SBC if you have not already done so

Once all of this is ready, you are able to deploy this repository following instructions below.


## Installing docker & docker-compose on the OS

If you are going to run this project directly using docker (not using Balena) then you will need to install docker on the OS first. Installing docker-compose is strongly recommended. This is pretty straight forward, just follow these steps:

```
sudo apt-get update && sudo apt-get upgrade -y
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker ${USER}
newgrp docker
sudo apt install -y python3 python3-dev python3-pip libffi-dev libssl-dev
sudo pip3 install docker-compose
sudo systemctl enable docker
```

Once done, you should be able to check the instalation is alright by testing:

```
docker --version
docker-compose --version
```


## Deploy the code

### Via docker-compose

You can use the `docker-compose.yml` file below to configure and run your instance of Basics™ Station connected to TTNv3:

```
version: '2.0'

services:

  basicstation:
    image: xoseperez/basicstation:latest
    container_name: basicstation
    restart: unless-stopped
    privileged: true
    network_mode: host      # required to read main interface MAC instead of virtual one
    environment:
      MODEL: "SX1303"
      TC_KEY: "..."         # Copy here your API key from the LNS
```

Modify the environment variables to match your setup. You will need a gateway key (`TC_KEY` variable above) to connect it to your LoRaWAN Network Server (LNS). If you want to do it beforehand you will need the Gateway EUI. Check the `Get the EUI of the Gateway` section below to know how. Otherwise, check the logs messages when the service starts to know the Gateway EUI to use.

### Build the image (not required)

In case you can not pull the already built image from Docker Hub or if you want to customize the cose, you can easily build the image by using the [buildx extension](https://docs.docker.com/buildx/working-with-buildx/) of docker and push it to your local repository by doing:

```
docker buildx bake --load aarch64
```

Once built (it will take some minutes) you can bring it up by using `xoseperez/basicstation:aarch64` as the image name in your `docker-compose.yml` file. If you are not in an ARMv8 64 bits machine (like a Raspberry Pi 4) you can change the `aarch64` with `armv7hf` (ARMv7) or `amd64` (AMD64).

The default built is the `std` variant. In case you want to build the `stdn` variant that supports multiple radios you can do it like this:

```
VARIANT=stdn docker buildx bake --load aarch64
```

The included build script in the root folder can be user to build all architectures and (optionally) push the to a repository. The default repository is `https://hub.docker.com/r/xoseperez/basicstation` which you don't have permissions to push to (obviously), but you can easily push the images to your own repo by doing:

```
REGISTRY="registry.example.com/basicstation" ./build.sh --push
```



### Via [Balena Deploy](https://www.balena.io/docs/learn/deploy/deploy-with-balena-button/)

Running this project is as simple as deploying it to a balenaCloud application. You can do it in just one click by using the button below:

[![](https://www.balena.io/deploy.png)](https://dashboard.balena-cloud.com/deploy?repoUrl=https://github.com/balenalabs/basicstation)

Follow instructions, click Add a Device and flash an SD card with that OS image dowloaded from balenaCloud. Enjoy the magic 🌟Over-The-Air🌟!


### Via [Balena-CLI](https://www.balena.io/docs/reference/balena-cli/)

If you are a balena CLI expert, feel free to use balena CLI.

- Sign up on [balena.io](https://dashboard.balena.io/signup)
- Create a new application on balenaCloud.
- Clone this repository to your local workspace.
- Using [Balena CLI](https://www.balena.io/docs/reference/cli/), push the code with `balena push <application-name>`
- See the magic happening, your device is getting updated 🌟Over-The-Air🌟!


## Configure the Gateway


### Basics Station Service Variables

These variables you can set them under the `environment` tag in the `docker-compose.yml` file or using an environment file (with the `env_file` tag). If you are using Balena you can also set them in the `Device Variables` tab for the device (or globally for the whole application). Only `MODEL` and `TC_KEY` are mandatory.

Variable Name | Value | Description | Default
------------ | ------------- | ------------- | -------------
**`MODEL`** | `STRING` | Concentrator model (see `Define your MODEL` section below) | ```SX1301```
**`DEVICE`** | `STRING` | Where the concentrator is connected to | `/dev/spidev0.0`
**`INTERFACE`** | `SPI` or `USB` | Concentrator interface | Guessed from DEVICE
**`SPI_SPEED`** | `INT` | Speed of the SPI interface | 2000000 (2MHz) for SX1301 concentrators, 8000000 (8Mhz) for the rest
**`GW_RESET_GPIO`** | `INT` | GPIO number that resets (Broadcom pin number, if not defined it's calculated based on the GW_RESET_PIN) | 17
**`GW_POWER_EN_GPIO`** | `INT` | GPIO number that enables power (by pulling HIGH) to the concentrator (Broadcom pin number). 0 means no required. | 0
**`GW_POWER_EN_LOGIC`** | `INT` | If `GW_POWER_EN_GPIO` is not 0, the corresponding GPIO will be set to this value | 1
**`GATEWAY_EUI_NIC`** | `STRING` | Interface to use when generating the EUI | `eth0`
**`GATEWAY_EUI`** | `STRING` | Gateway EUI to use | Autogenerated from `GATEWAY_EUI_NIC` if defined, otherwise in order from: `eth0`, `wlan0`, `usb0`
**`TTN_REGION`** | `STRING` | Region of the TTNv3 server to use | ```eu1```
**`TC_URI`** | `STRING` | LoRaWAN Network Server to connect to | Automatically created based on TTN_REGION for TTN
**`TC_TRUST`** | `STRING` | Certificate for the server | Precached certificate
**`TC_KEY`** | `STRING` | Unique gateway key used to connect to the LNS | Paste API key from your LNS
**`LORAGW_SPI`** | `STRING` | Where the concentrator is connected to **depecated, use DEVICE instead** | `/dev/spidev0.0`
**`GW_RESET_PIN`** | `INT` | PIN number that resets de concentrator (Header pin number) **deprecated, use GW_RESET_GPIO instead** | 11

> At least `MODEL` and `TC_KEY` must be defined.

> If you have more than one concentrator on the same device, you can set the BasicStation service to use both at the same time. Check `Advanced configuration` section below to know more. You can also bring up two instances of BasicStation on the same device to control two different concentrators. In this case you will want to assign different `DEVICE`, `GATEWAY_EUI` and `TC_KEY` values to each instance.


### Define your MODEL

The model is defined depending on the version of the LoRa concentrator chip: `SX1301`, `SX1302` or `SX1303`. You can also use the concentrator module name or even the gateway model (for RAKwireless gateways). Actual list of valid values:

* Semtech chip model: SX1301, SX1302, SX1303
* Concentrator modules: IC880A, RAK2245, RAK2247, RAK2287, RAK5146, RAK831, RAK833, WM1302
* RAK WisGate Development gateways: RAK7243, RAK7243C, RAK7244, RAK7244C, RAK7248, RAK7248C, RAK7271, RAK7371

If the module is not on the list check the manufacturer to see which one to use. It's important to change the `MODEL` variable (in you `docker-compose.yml` file or in balenaCloud) to the correct one. The default model is the `SX1301`.


### Get the EUI of the Gateway

LoRaWAN gateways are identified with a unique 64 bits (8 bytes) number, called EUI, which can be used to register the gateway on the LoRaWAN Network Server. You can check the gateway EUI (and other data) by inspecting the service logs or running the command below while the container is up:

```
docker run -it --network host --rm xoseperez/basicstation:latest ./get_eui.sh

```

You can do so before bringing up the service, so you first get the EUI, register the gateway and get the KEY to populate it on the `docker-compose.yml` file. If you are specifying a different NIC to create the EUI from (see the GATEWAY_EUI_NIC variable above), you can do it like this:

```
docker run -it --network host --rm xoseperez/basicstation:latest bash -c "GATEWAY_EUI_NIC=wlan0 ./get_eui.sh"

```

If using balenaCloud the ```EUI``` will be visible as a TAG on the device dashboard. Be careful when you copy the tag, as other characters will be copied.


### Using TTN? Define your REGION

If you plan to use The Things Network, set the `TTN_REGION` variable. It needs to be changed if your region is not Europe. The default value is `eu1` for the European server. By setting this variable (`TTN_REGION`) the `TC_URI` will be generated automatically. 


### Configure your The Things Stack gateway

1. Sign up at [The Things Stack console](https://ttc.eu1.cloud.thethings.industries/console/).
2. Click "Go to Gateways" icon.
3. Click the "Add gateway" button.
4. Introduce the data for the gateway.
5. Paste the EUI of the gateway (see `Get the EUI of the Gateway` section above)
6. Complete the form and click Register gateway.
7. Once the gateway is created, click "API keys" link.
8. Click "Add API key" button.
9. Select "Grant individual rights" and then "Link as Gateway to a Gateway Server for traffic exchange ..." and then click "Create API key".
10. Copy the API key generated and paste it into your docker-compose.yml file or use it on balenaCloud as ```TC_KEY``` variable.


### Not using TTN?

In case that you want to point to another LNS different from The Things Network you will have to define a specific `TC_URI`. If your server certificate is not issued by a known authority (for instance, if it's self-signed) you will also have to define the `TC_TRUST` variable with the public certificate for your LSN server. It should be something like (in one line): 

```
-----BEGIN CERTIFICATE----- MIIDSjCCAjKgAwIBAgIQRK+wgNajJ7qJMDmGLvhAazANBgkqhkiG9w0BAQUFADA/MSQwIgYDVQQKExtEaWdpdGFsIFNpZ25hdHVyZSBUcnVzdCBDby4xFzAVBgNVBAMTDkRTVCBSb290IENBIFgzMB4XDTAwMDkzMDIxMTIxOVoXDTIxMDkzMDE0MDExNVowPzEkMCIGA1UEChMbRGlnaXRhbCBTaWduYXR1cmUgVHJ1c3QgQ28uMRcwFQYDVQQDEw5EU1QgUm9vdCBDQSBYMzCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAN+v6ZdQCINXtMxiZfaQguzH0yxrMMpb7NnDfcdAwRgUi+DoM3ZJKuM/IUmTrE4Orz5Iy2Xu/NMhD2XSKtkyj4zl93ewEnu1lcCJo6m67XMuegwGMoOifooUMM0RoOEqOLl5CjH9UL2AZd+3UWODyOKIYepLYYHsUmu5ouJLGiifSKOeDNoJjj4XLh7dIN9bxiqKqy69cK3FCxolkHRyxXtqqzTWMIn/5WgTe1QLyNau7Fqckh49ZLOMxt+/yUFw7BZy1SbsOFU5Q9D8/RhcQPGX69Wam40dutolucbY38EVAjqr2m7xPi71XAicPNaDaeQQmxkqtilX4+U9m5/wAl0CAwEAAaNCMEAwDwYDVR0TAQH/BAUwAwEB/zAOBgNVHQ8BAf8EBAMCAQYwHQYDVR0OBBYEFMSnsaR7LHH62+FLkHX/xBVghYkQMA0GCSqGSIb3DQEBBQUAA4IBAQCjGiybFwBcqR7uKGY3Or+Dxz9LwwmglSBd49lZRNI+DT69ikugdB/OEIKcdBodfpga3csTS7MgROSR6cz8faXbauX+5v3gTt23ADq1cEmv8uXrAvHRAosZy5Q6XkjEGB5YGV8eAlrwDPGxrancWYaLbumR9YbK+rlmM6pZW87ipxZzR8srzJmwN0jP41ZL9c8PDHIyh8bwRLtTcm1D9SZImlJnt1ir/md2cXjbDaJWFBM5JDGFoqgCWjBH4d1QB7wCCZAA62RjYJsWvIjJEubSfZGL+T0yjWW06XyxV3bqxbYoOb8VZRzI9neWagqNdwvYkQsEjgfbKbYK7p2CNTUQ -----END CERTIFICATE-----
```


### Advanced configuration

In some special cases you might want to specify the radio configuration in detail (frequencies, power, ...). This will also be the case when you want to use more than one concentrator on the same gateway, using the same BasicStation service. You can do that by mounting the `config` folder on your host machine and providing custom files, like a specific `station.conf` file.

You can start by modifying the `docker-compose.yml` file to mount that folder locally:

```
version: '2.0'

services:

  basicstation:
    image: xoseperez/basicstation:latest
    container_name: basicstation
    restart: unless-stopped
    privileged: true
    network_mode: host
    volumes:
      - ./config:/app/config
    environment:
      MODEL: "SX1303"
      TC_KEY: "..."
```

Then bring up the service and it will populate several config files in this folder. Now you can shut the service down and proceed to edit these files to match your needs. Now, when you bring the service up again it will find the pre-created files and it will use them instead of creating new ones. The files you might want to change are:

* `station.conf`: Main configuration file. It does not include frequencies and power since these are retrieved form the LNS. Check https://doc.sm.tc/station/conf.html.
* `slave-N.conf`: Specific configuration file for each radio. They must exist, even if all settings are inherited from the `station.conf` file. In that case, the slave file will only contain an empty object: `{}`.
* `tc.uri`: File containing the URL of the LNS.
* `tc.trust`: File containing the certificate of the LNS server.
* `tc.key`: File containing the key of the gateway to connect to the LNS server.

**NOTE**: files in the config folder take precedence over variables, so if you mount a `config` folder with a `station.conf` file, the `DEVICE` or `GATEWAY_EUI` variables will not be used. If you want to change any of them, you will have to modify the file manually or delete it so it will be recreated form the variables again.


## Troubleshoothing

* It's possible that on the TTN Console the gateway appears as Not connected if it's not receiving any LoRa message. Sometimes the websockets connection among the LoRa Gateway and the server can get broken. However a new LoRa package will re-open the websocket between the Gateway and TTN or TTI.

* The RAK833-SPI/USB concentrator has both interfaces and they are selected using a SPDT switch in the module. Since this is an SX1301-based concentrator only the SPI interface is supported so you will have to assert pin 17 in the MiniPCIe gold-finger of the concentrator LOW. This can be done using the `GW_POWER_EN_GPIO` and `GW_POWER_EN_LOGIC` variables. If using a [RAK 2247 Pi Hat](https://store.rakwireless.com/collections/wishat/products/rak2247-pi-hat) or [RAK 2287 Pi Hat](https://store.rakwireless.com/collections/wishat/products/rak2287-pi-hat) this pin is wired to GPIO20 in the Raspberry Pi. So a working configuration for this concentrator would be:

```
version: '2.0'

services:

  basicstation:
    image: xoseperez/basicstation:latest
    container_name: basicstation
    restart: unless-stopped
    privileged: true
    network_mode: host
    environment:
      MODEL: "RAK833"
      GW_POWER_EN_GPIO: 20
      GW_POWER_EN_LOGIC: 0
      TC_KEY: "..."
```


Feel free to introduce issues on this repo and contribute with solutions.

## Attribution

- This is an adaptation of the [Semtech Basics Station repository](https://github.com/lorabasics/basicstation). Documentation [here](https://doc.sm.tc/station).
- This is in part working thanks of the work of Jose Marcelino from RAK Wireless, Xose Pérez from Allwize & RAK Wireless and Marc Pous from balena.io.
- This is in part based on excellent work done by Rahul Thakoor from the Balena.io Hardware Hackers team.


## License

The contents of this repository (not of those repositories linked or used by this one) are under BSD 3-Clause License.

Copyright (c) 2021-2022 Xose Pérez <xose.perez@gmail.com>
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice,
   this list of conditions and the following disclaimer.
2. Redistributions in binary form must reproduce the above copyright
   notice, this list of conditions and the following disclaimer in the
   documentation and/or other materials provided with the distribution.
3. Neither the name of mosquitto nor the names of its
   contributors may be used to endorse or promote products derived from
   this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE
LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
POSSIBILITY OF SUCH DAMAGE.
