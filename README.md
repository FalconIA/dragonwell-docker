# Dragonwell and Docker
Dockerfiles and build scripts for generating Docker Images based on Alibaba Dragonwell binaries.

# Supported Architectures
* Dragonwell is supported on ```Linux/x86_64``` only.

# Unofficial Images
Dragonwell Unofficial Images (Maintained by FalconIA).
  - Linux
    - Alpine (3.12): Release, Nightly and Slim
    - Debian (Buster): Release, Nightly and Slim
    - UBI (8): Release, Nightly and Slim
    - UBI-Minimal (8): Release, Nightly and Slim
    - Ubuntu (20.04): Nightly and Slim


## Unofficial Images: Docker Image Build Types and Associated Tags

### Legend

   * ${os} = alpine|debian|ubi|ubi-minimal|ubuntu|windows
   * ${slim-os} = alpine|debian|ubi|ubuntu
   * ${jdk-version} Eg. jdk-11.0.3_7, jdk-12.33_openj9-0.13.0
   * ${jre-version} Eg. jre-11.0.3_7, jre-12.33_openj9-0.13.0

1. There are two kinds of build images
   * Release build images
     - These are release tested versions of the JDKs.
     - Associated tags:
     ```
       - latest, ${os},           ${jdk-version},      ${jdk-version}-${os}
       - jre,    ${os}-jre,       ${jre-version},      ${jre-version}-${os}
       - slim,   ${slim-os}-slim, ${jdk-version}-slim, ${jdk-version}-${slim-os}-slim
     ```
   * Nightly build images
     - These are nightly builds with minimal testing.
     - Associated tags:
     ```
       - nightly,      ${os}-nightly,           ${jdk-version}-${os}-nightly
       - jre-nightly,  ${os}-jre-nightly,       ${jre-version}-${os}-nightly
       - nightly-slim, ${slim-os}-nightly-slim, ${jdk-version}-${slim-os}-nightly-slim
     ```  
2. There are two build types
   * Full build images
     - This consists of the full JDK.
     - Associated tags:
     ```
       - latest,      ${os},             ${jdk-version},         ${jdk-version}-${os}
       - jre,         ${os}-jre,         ${jre-version},         ${jre-version}-${os}
       - nightly,     ${os}-nightly,     ${jdk-version}-nightly, ${jdk-version}-${os}-nightly
       - jre-nightly, ${os}-jre-nightly, ${jre-version}-nightly, ${jre-version}-${os}-nightly
     ```  
   * Slim build images
     - These are stripped down JDK builds that remove functionality not typically needed while running in a cloud. See the [./slim-java.sh](./slim-java.sh) script to see what is stripped out.
     - Associated tags:
     ```
       - slim,         ${slim-os}-slim,         ${jdk-version}-slim,         ${jdk-version}-${slim-os}-slim
       - nightly-slim, ${slim-os}-nightly-slim, ${jdk-version}-nightly-slim, ${jdk-version}-${slim-os}-nightly-slim
     ```  
3. There are also JDK and JRE only variants
   * JDK build images
     - This consists of the full JDK.
     - Associated tags:
     ```
       - latest,       ${os},                   ${jdk-version},              ${jdk-version}-${os}
       - slim,         ${slim-os}-slim,         ${jdk-version}-slim,         ${jdk-version}-${slim-os}-slim
       - nightly,      ${os}-nightly,           ${jdk-version}-nightly,      ${jdk-version}-${os}-nightly
       - nightly-slim, ${slim-os}-nightly-slim, ${jdk-version}-nightly-slim, ${jdk-version}-${slim-os}-nightly-slim
     ```  
   * JRE build images
     - This consists of only JRE.
     - Associated tags:
     ```
       - jre,         ${os}-jre,         ${jre-version},         ${jre-version}-${os}
       - jre-nightly, ${os}-jre-nightly, ${jre-version}-nightly, ${jre-version}-${os}-nightly
     ```

**Here is a listing of the image sizes for the various build images and types for JDK Version 8**

| VMs        | latest | slim | alpine | alpine-slim |
|:----------:|:------:|:----:|:------:|:-----------:|
|Dragonwell8 | 267MB  | 226MB| 171MB  | 116MB       |
|Dragonwell11| 432MB  | 367MB| 336MB  | 256MB       |

**Notes:**
1. The alpine-slim images are about 60% smaller than the latest images.
2. The Alpine Linux and the slim images are not yet TCK certified.

# Build and push the Images with multi-arch support

```
# Steps 1-2 needs to be run on all supported arches.
# i.e aarch64, ppc64le, s390x and x86_64.

# 1. Clone this github repo
     $ git clone https://github.com/FalconIA/dragonwell-docker

# 2. Build images and tag them appropriately
     $ cd dragonwell-docker
     $ ./build_all.sh

# Steps 3 needs to be run only on x86_64

# 3. build_all.sh should be run on all supported architectures to build and push images to the
#    docker registry. The images should now be available on hub.docker.com but without multi-arch
#    support. To add multi-arch support, we need to generate the right manifest lists and push them
#    to hub.docker.com. The script generate_manifest_script.sh can be used to
#    generate the right manifest commands. This needs to be run only on x86_64 after docker images
#    for all architecures have been built and made available on hub.docker.com
     $ ./update_manifest_all.sh

# We should now have the proper manifest lists pushed to hub.docker.com to support multi-arch pulls.
```

# Info on other scripts
 - [update_all.sh](/update_all.sh): Script to generate all Dockerfiles.
   - [update_multiarch.sh](/update_multiarch.sh): Helper script that generates Dockerfiles for a specific Java version.
   ```
     $ ./update_multiarch.sh $version
   ```
   - [dockerfile_functions.sh](/dockerfile_functions.sh): Dockerfile content is generated from this. Update this script if you want any changes to the generated Dockerfiles.
 - [build_all.sh](/build_all.sh): Script to build all supported unofficial docker images on a particular architecture.
   - [build_latest.sh](/build_latest.sh): Helper script that builds a docker image for a specific Java version, VM and package combination.
 
 - [update_manifest_all.sh](/update_manifest_all.sh): Script that generates the multi-arch manifest for all unofficial docker images for supported/released architectures at any given time.
   - [generate_manifest_script.sh](/generate_manifest_script.sh): Helper script that generates the manifest for a given Java version, VM and Package combination for all supported architectures. If a build is unavailable for a supported architecture (build failed, not yet released etc), a manifest entry for that architecture will not be added.

 - [linter.sh](/linter.sh): Linting dockerfiles (via [hadolint](https://github.com/hadolint/hadolint)). 
   ```
    To lint generated dockerfiles run 
    $ ./linter.sh
   ```
#### Helper Scripts

 - [slim-java.sh](/slim-java.sh): Script that is used to generate the slim docker images. This script strips out various aspects of the JDK that are typically not needed in a server side containerized application. This includes debug info, symbols, classes related to audio, desktop etc
 - [dockerhub_doc_config_update.sh](/dockerhub_doc_config_update.sh): Script that generates the tag documentation for each of the unofficial docker image pages on hub.docker.com.

#### Config Files

The [config](/config/) dir consists of configuration files used by the scripts to determine the supported combinations of Version / OS / VM / Package / Build types and Architectures for both Official/Unofficial images as well as the corresponding tags.
- [dragonwell.config](/config/dragonwell.config): Configuration for unofficial images for Dragonwell.
- [tags.config](/config/tags.config): Configuration for creating tags.

## Mac OS X
Please note you'll need to [upgrade bash shell on Mac OS X](https://itnext.io/upgrading-bash-on-macos-7138bd1066ba) if you're to use our Docker images on there.

# License
The Dockerfiles and associated scripts found in this project are licensed under the [Apache License 2.0.](https://www.apache.org/licenses/LICENSE-2.0.html).
