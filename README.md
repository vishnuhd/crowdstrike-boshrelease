# Crowdstrike Bosh release

* Open ssh connection to any VM having access to BOSH env.

* Add the required Bosh parameters to the env like BOSH_ALL_PROXY, BOSH_ENVIRONMENT, etc.

* Clone this repository, cd into its directory and execute:

  `bosh init-release`


* The config directory is created with the following contents:
  ```
  ├ config
  │   ├ blobs.yml
  │   └ final.yml
  ├ jobs
  ├ packages
  └ src
  ```

* Copy Falcon sensor agent (Crowdstrike agent) to `src` directory, example:
  ```
  ├ src
  │   ├  falcon-sensor_6.37.0-13402_amd64.deb
  ```

* Register blobs or a new version using the following command:
  ```
  bosh add-blob src/falcon-sensor_6.37.0-13402_amd64.deb falcon-sensor_6.37.0-13402_amd64.deb
  ```
This populates the `blobs.yml` with blob filename, filename size, and SHA

* Update `packages > crowdstrike-agent > spec`, to include the blob
  ```
  files:
    - falcon-sensor_6.37.0-13402_amd64.deb
  ```

* Update the file under `config > final.yml` to include the `blobstore_path` variable:

  ```
  blobstore:
    provider: local
    options:
      blobstore_path: /Users/bob/blobs

  name: crowdstrike-boshrelease
  ```

* Create the Release
  ```
  bosh create-release --version='6.37.0' --final --force --tarball=crowdstrike-boshrelease.tgz
  ```

* Update the file under `runtime-config > crowstrike.yml` file and update the version number, from the output of the previous command
  ```
  version: 6.37.0
  ```

* Upload the bosh Release
  ```
  bosh upload-release crowdstrike-boshrelease.tgz
  ```

* Generate runtime config
  ```
  bosh int runtime-config/crowdstrike.yml --var=release_version='6.37.0' --var=crowdstrike_customer_id='xxxxxxxxxx' --var=crowdstrike_proxy_hostname='10.0.0.x' > generated_runtime_config.yml
  ```

* Update the runtime config
  ```
  bosh update-runtime-config --name=crowdstrike generated_runtime_config.yml
  ```

* Trigger `Apply Changes` from Ops Manager
