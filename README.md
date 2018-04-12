# Infrastructure repository for Webstore

## Setup to provision GoCD on local or cloud env.

<p>Basic Setup required to create vagrant machine</p>


1. install VirtualBox 
2. install Vagrant
3. install managed server plugin
    ```shell 
    vagrant plugin install vagrant-managed-servers
    ```
4. Import gerrit-access key from secrets
   ```shell
    ./add-secrets.sh
   ```

## Provisioning GoCD machine on local machine

#### Starting local cluster impostor
<p>VM pretending to be the MCloud machine. Execute following commands to start local cluster imposter</p>

1.  ```shell 
    vagrant up local-mcloud-impostor
    ```
#### Provisioning local GoCD

1. ```shell
    vagrant up gocd-local
    ```
2. ```shell 
    vagrant provision gocd-local
    ```

#### GoCD User Interface

<p> Hola! you have successfully provisioned GoCD server on your local machine. GoCD server can be accessed from host machine using <a href=http://192.168.33.66:80>url</a></p>
     ```