# Instructions for creating BOSH(dev/final) Release by using BOSH CLI

## Download the Development Tools

### 1. download tile generator(Use Python and Virtualenv)
```
virtualenv -p python3 tile-generator-env

source tile-generator-env/bin/activate

pip install tile-generator
```

### 2. download bosh cli
```
brew install cloudfoundry/tap/bosh-cli

bosh -v
```

## Clone Git Repos
### Clone `signalfx-cloudfoundry-bridge` Repo and `signalfx-cloudfoundry-bridge-boshrelease` Repo
```
git clone git@github.com:signalfx/signalfx-cloudfoundry-bridge.git

git clone git@github.com:signalfx/signalfx-cloudfoundry-bridge-boshrelease.git

cd signalfx-cloudfoundry-bridge-boshrelease

```

## Create Dev Release

### 1. Create the binary signalfx_bridage
```
make bridge-binary
```

### 2. Add local blob
```
bosh add-blob bridge-linux-amd64 signalfx_bridge/bridge-linux-amd64
```

### 3. Create the dev release tarball at path ./tmp/release.tgz
```
bosh create-release --force --tarball=./tmp/release.tgz
```
You can use option `--version <version_number>` to specify the version number. e.g
```
bosh create-release --force --version 0.11.6+dev.1 --tarball=./tmp/release.tgz
```

### 4. Generate tile for dev release
```
tile build <version_number>
```
e.g
```
tile build 0.11.6-dev1
```

### 5. Find the generated Tile and BOSH release
- The Pivotal tile, e.g. `signalfx-bridge-0.11.6-dev1.pivotal`, is stored in the **`product`** subdirectory
- The dev BOSH release, e.g. `signalfx-bridge-11+dev.1.tgz`, is stored in **`release/signalfx_bridge`** subdirectory.


## Test Dev Release
### Test the BOSH release on open-source CF via BOSH CLI
Assuming you are in the release directory,
#### 1. Upload the new release
```
bosh -e bosh-lite upload-release --fix
```
where, 
- `bosh-lite`: Director environment name or URL
- `--fix`: Replaces corrupt and missing jobs and packages
- For more info: https://bosh.io/docs/cli-v2/#upload-blobs


#### 2. Deploy the new release
```
bosh -e bosh-lite -d bridge deploy manifest/bridge.yml
```
where, 
- `bosh-lite`: Director environment name or URL
- `bridge`: Deployment name
- `manifest/bridge.yml`: <path-to-manifest.yml>
- For more info: https://bosh.io/docs/cli-v2/#deploy

If your release fails tests, follow this pattern.

- Fix the code.
- Do a new dev release.
- Run bosh deploy to see whether the new release deploys successfully.

Using `bosh deploy --recreate` can provide a clearer picture because with that option, BOSH deploys all the VMs from scratch.

### Test the Tile on the Ops manager
- Test the generated tile by imported it on Ops Manager

## Create Final Release
**Note: Only proceed to this step if your latest dev release passes all tests.**

### 1. Create the binary signalfx_bridage
```
make bridge-binary
```

### 2. Add local blob
```
bosh add-blob bridge-linux-amd64 signalfx_bridge/bridge-linux-amd64
```
**Note**: You can skip step 1 and step 2 if you already did it for dev release

### 3. Upload blobs
**Note** 
- Make sure you update your s3 information in `config/final.yml`. Or you can specify your own blobstore providers here. 
- Create `config/private.yml` and put keys for accessing the blobstore here.
- Please check [here](https://bosh.io/docs/release-blobstore/) to find out how to configure blobstore providers in both `final.yml` and `private.yml`

```
bosh upload-blobs
```

### 4. Create final release
```
bosh create-release --final --tarball tmp/release.tgz --name signalfx-bridge --force
```

You can use option `--version <version_number>` to specify the version number. e.g
```
bosh create-release --final --tarball tmp/release.tgz --name signalfx-bridge --version 0.11.6 --force
```

**Note**: You will see a new release yml file is generated under `release/signalfx-bridge`

### 5. Generate tile for final release
```
tile build <version_number>
```
e.g
```
tile build 0.11.6
```

### 6. Find the generated Tile and BOSH release
- The Pivotal tile, e.g. `signalfx-bridge-0.11.6.pivotal`, is stored in the **`product`** subdirectory
- The final BOSH release, e.g. `signalfx-bridge-0.11.6.tgz`, is stored in **`release/signalfx_bridge`** subdirectory.


# Reference
- Creating a Release: https://bosh.io/docs/create-release/





