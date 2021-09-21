---
title: Working with the Docker registry
intro: '{% ifversion fpt %}The Docker registry has now been replaced by the {% data variables.product.prodname_container_registry %}.{% else %}You can push and pull your Docker images using the {% data variables.product.prodname_registry %} Docker registry, which uses the package namespace `https://docker.pkg.github.com`.{% endif %}'
product: '{% data reusables.gated-features.packages %}'
redirect_from:
  - /articles/configuring-docker-for-use-with-github-package-registry
  - /github/managing-packages-with-github-package-registry/configuring-docker-for-use-with-github-package-registry
  - /github/managing-packages-with-github-packages/configuring-docker-for-use-with-github-packages
  - /packages/using-github-packages-with-your-projects-ecosystem/configuring-docker-for-use-with-github-packages
  - /packages/guides/container-guides-for-github-packages/configuring-docker-for-use-with-github-packages
  - /packages/guides/configuring-docker-for-use-with-github-packages
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
shortTitle: Docker registry
---

<!-- Main versioning block. Short page for dotcom -->
{% ifversion fpt %}

{% data variables.product.prodname_dotcom %}'s Docker registry (which used the namespace `docker.pkg.github.com`) has been replaced by the {% data variables.product.prodname_container_registry %} (which uses the namespace `https://ghcr.io`). The {% data variables.product.prodname_container_registry %} offers benefits such as granular permissions and storage optimization for Docker images.

Docker images previously stored in the Docker registry are being automatically migrated into the {% data variables.product.prodname_container_registry %}. For more information, see "[Migrating to the {% data variables.product.prodname_container_registry %} from the Docker registry](/packages/working-with-a-github-packages-registry/migrating-to-the-container-registry-from-the-docker-registry)" and "[Working with the {% data variables.product.prodname_container_registry %}](/packages/working-with-a-github-packages-registry/working-with-the-container-registry)."

{% else %}
<!-- The remainder of this article is displayed for releases that don't support the Container registry -->

{% data reusables.package_registry.packages-ghes-release-stage %}
{% data reusables.package_registry.packages-ghae-release-stage %}

{% data reusables.package_registry.admins-can-configure-package-types %}

## About Docker support

When installing or publishing a Docker image, the Docker registry does not currently support foreign layers, such as Windows images.

{% ifversion ghes = 2.22 %}

Before you can use the Docker registry on {% data variables.product.prodname_registry %}, the site administrator for {% data variables.product.product_location %} must enable Docker support and subdomain isolation for your instance. For more information, see "[Managing GitHub Packages for your enterprise](/enterprise/admin/packages)."

{% endif %}

## Authenticating to {% data variables.product.prodname_registry %}

{% data reusables.package_registry.authenticate-packages %}

{% data reusables.package_registry.authenticate-packages-github-token %}

### Authenticating with a personal access token

{% data reusables.package_registry.required-scopes %}

You can authenticate to {% data variables.product.prodname_registry %} with Docker using the `docker` login command.

To keep your credentials secure, we recommend you save your personal access token in a local file on your computer and use Docker's `--password-stdin` flag, which reads your token from a local file.

{% ifversion fpt %}
{% raw %}
  ```shell
  $ cat <em>~/TOKEN.txt</em> | docker login https://docker.pkg.github.com -u <em>USERNAME</em> --password-stdin
  ```
{% endraw %}
{% endif %}

{% ifversion ghes or ghae %}
{% ifversion ghes > 2.22 %}
If your instance has subdomain isolation enabled:
{% endif %}
{% raw %}
 ```shell
 $ cat <em>~/TOKEN.txt</em> | docker login docker.HOSTNAME -u <em>USERNAME</em> --password-stdin
```
{% endraw %}
{% ifversion ghes > 2.22 %}
If your instance has subdomain isolation disabled:

{% raw %}
 ```shell
 $ cat <em>~/TOKEN.txt</em> | docker login <em>HOSTNAME</em> -u <em>USERNAME</em> --password-stdin
```
{% endraw %}
{% endif %}

{% endif %}

To use this example login command, replace `USERNAME` with your {% data variables.product.product_name %} username{% ifversion ghes or ghae %}, `HOSTNAME` with the URL for {% data variables.product.product_location %},{% endif %} and `~/TOKEN.txt` with the file path to your personal access token for {% data variables.product.product_name %}.

For more information, see "[Docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin)."

## Publishing an image

{% data reusables.package_registry.docker_registry_deprecation_status %}

{% note %}

**Note:** Image names must only use lowercase letters.

{% endnote %}

{% data variables.product.prodname_registry %} supports multiple top-level Docker images per repository. A repository can have any number of image tags. You may experience degraded service publishing or installing Docker images larger than 10GB, layers are capped at 5GB each. For more information, see "[Docker tag](https://docs.docker.com/engine/reference/commandline/tag/)" in the Docker documentation.

{% data reusables.package_registry.viewing-packages %}

1. Determine the image name and ID for your docker image using `docker images`.
  ```shell
  $ docker images
  > <&nbsp>
  > REPOSITORY        TAG        IMAGE ID       CREATED      SIZE
  > <em>IMAGE_NAME</em>        <em>VERSION</em>    <em>IMAGE_ID</em>       4 weeks ago  1.11MB
  ```
2. Using the Docker image ID, tag the docker image, replacing *OWNER* with the name of the user or organization account that owns the repository, *REPOSITORY* with the name of the repository containing your project, *IMAGE_NAME* with name of the package or image,{% ifversion ghes or ghae %} *HOSTNAME* with the hostname of {% data variables.product.product_location %},{% endif %} and *VERSION* with package version at build time.
  {% ifversion fpt %}
  ```shell
  $ docker tag <em>IMAGE_ID</em> docker.pkg.github.com/<em>OWNER/REPOSITORY/IMAGE_NAME:VERSION</em>
  ```
  {% else %}
  {% ifversion ghes > 2.22 %}
  If your instance has subdomain isolation enabled:
  {% endif %}
  ```shell
  $ docker tag <em>IMAGE_ID</em> docker.<em>HOSTNAME/OWNER/REPOSITORY/IMAGE_NAME:VERSION</em>
  ```
  {% ifversion ghes > 2.22 %}
  If your instance has subdomain isolation disabled:
  ```shell
  $ docker tag <em>IMAGE_ID</em> <em>HOSTNAME/OWNER/REPOSITORY/IMAGE_NAME:VERSION</em>
  ```
  {% endif %}
  {% endif %}
3. If you haven't already built a docker image for the package, build the image, replacing *OWNER* with the name of the user or organization account that owns the repository, *REPOSITORY* with the name of the repository containing your project, *IMAGE_NAME* with name of the package or image, *VERSION* with package version at build time,{% ifversion ghes or ghae %} *HOSTNAME* with the hostname of {% data variables.product.product_location %},{% endif %} and *PATH* to the image if it isn't in the current working directory.
  {% ifversion fpt %}
  ```shell
  $ docker build -t docker.pkg.github.com/<em>OWNER/REPOSITORY/IMAGE_NAME:VERSION</em> <em>PATH</em>
  ```
  {% else %}
  {% ifversion ghes > 2.22 %}
  If your instance has subdomain isolation enabled:
  {% endif %}
  ```shell
  $ docker build -t docker.<em>HOSTNAME/OWNER/REPOSITORY/IMAGE_NAME:VERSION</em> <em>PATH</em>
  ```
  {% ifversion ghes > 2.22 %}
  If your instance has subdomain isolation disabled:
  ```shell
  $ docker build -t <em>HOSTNAME/OWNER/REPOSITORY/IMAGE_NAME:VERSION</em> <em>PATH</em>
  ```
  {% endif %}
  {% endif %}
4. Publish the image to {% data variables.product.prodname_registry %}.
  {% ifversion fpt %}
  ```shell
  $ docker push docker.pkg.github.com/<em>OWNER/REPOSITORY/IMAGE_NAME:VERSION</em>
  ```
  {% else %}
  {% ifversion ghes > 2.22 %}
  If your instance has subdomain isolation enabled:
  {% endif %}
  ```shell
  $ docker push docker.<em>HOSTNAME/OWNER/REPOSITORY/IMAGE_NAME:VERSION</em>
  ```
  {% ifversion ghes > 2.22 %}
  If your instance has subdomain isolation disabled:
  ```shell
  $ docker push <em>HOSTNAME/OWNER/REPOSITORY/IMAGE_NAME:VERSION</em>
  ```
  {% endif %}
  {% endif %}
  {% note %}

  **Note:** You must push your image using `IMAGE_NAME:VERSION` and not using `IMAGE_NAME:SHA`.

  {% endnote %}

### Example publishing a Docker image

{% ifversion ghes > 2.22 %}
These examples assume your instance has subdomain isolation enabled.
{% endif %}

You can publish version 1.0 of the `monalisa` image to the `octocat/octo-app` repository using an image ID.

{% ifversion fpt %}
```shell
$ docker images

> REPOSITORY           TAG      IMAGE ID      CREATED      SIZE
> monalisa             1.0      c75bebcdd211  4 weeks ago  1.11MB

# Tag the image with <em>OWNER/REPO/IMAGE_NAME</em>
$ docker tag c75bebcdd211 docker.pkg.github.com/octocat/octo-app/monalisa:1.0

# Push the image to {% data variables.product.prodname_registry %}
$ docker push docker.pkg.github.com/octocat/octo-app/monalisa:1.0
```

{% else %}

```shell
$ docker images

> REPOSITORY           TAG      IMAGE ID      CREATED      SIZE
> monalisa             1.0      c75bebcdd211  4 weeks ago  1.11MB

# Tag the image with <em>OWNER/REPO/IMAGE_NAME</em>
$ docker tag c75bebcdd211 docker.<em>HOSTNAME</em>/octocat/octo-app/monalisa:1.0

# Push the image to {% data variables.product.prodname_registry %}
$ docker push docker.<em>HOSTNAME</em>/octocat/octo-app/monalisa:1.0
```

{% endif %}

You can publish a new Docker image for the first time and name it `monalisa`.

{% ifversion fpt %}
```shell
# Build the image with docker.pkg.github.com/<em>OWNER/REPOSITORY/IMAGE_NAME:VERSION</em>
# Assumes Dockerfile resides in the current working directory (.)
$ docker build -t docker.pkg.github.com/octocat/octo-app/monalisa:1.0 .

# Push the image to {% data variables.product.prodname_registry %}
$ docker push docker.pkg.github.com/octocat/octo-app/monalisa:1.0
```

{% else %}
```shell
# Build the image with docker.<em>HOSTNAME/OWNER/REPOSITORY/IMAGE_NAME:VERSION</em>
# Assumes Dockerfile resides in the current working directory (.)
$ docker build -t docker.<em>HOSTNAME</em>/octocat/octo-app/monalisa:1.0 .

# Push the image to {% data variables.product.prodname_registry %}
$ docker push docker.<em>HOSTNAME</em>/octocat/octo-app/monalisa:1.0
```
{% endif %}

## Downloading an image

{% data reusables.package_registry.docker_registry_deprecation_status %}

You can use the `docker pull` command to install a docker image from {% data variables.product.prodname_registry %}, replacing *OWNER* with the name of the user or organization account that owns the repository, *REPOSITORY* with the name of the repository containing your project, *IMAGE_NAME* with name of the package or image,{% ifversion ghes or ghae %} *HOSTNAME* with the host name of {% data variables.product.product_location %}, {% endif %} and *TAG_NAME* with tag for the image you want to install.

{% ifversion fpt %}
```shell
$ docker pull docker.pkg.github.com/<em>OWNER/REPOSITORY/IMAGE_NAME:TAG_NAME</em>
```
{% else %}
<!--Versioning out this "subdomain isolation enabled" line because it's the only option for GHES 2.22 so it can be misleading.-->
{% ifversion ghes > 2.22 %}
If your instance has subdomain isolation enabled:
{% endif %}
```shell
$ docker pull docker.<em>HOSTNAME/OWNER/REPOSITORY/IMAGE_NAME:TAG_NAME</em>
```
{% ifversion ghes > 2.22 %}
If your instance has subdomain isolation disabled:
```shell
$ docker pull <em>HOSTNAME/OWNER/REPOSITORY/IMAGE_NAME:TAG_NAME</em>
```
{% endif %}
{% endif %}

{% note %}

**Note:** You must pull the image using `IMAGE_NAME:VERSION` and not using `IMAGE_NAME:SHA`.

{% endnote %}

## Further reading

- "{% ifversion fpt or ghes > 3.0 %}[Deleting and restoring a package](/packages/learn-github-packages/deleting-and-restoring-a-package){% elsif ghes < 3.1 or ghae %}[Deleting a package](/packages/learn-github-packages/deleting-a-package){% endif %}"

{% endif %}  <!-- End of main versioning block -->

Registry compatibility

Synopsis
If a manifest is pulled by digest from a registry 2.3 with Docker Engine 1.9 and older, and the manifest was pushed with Docker Engine 1.10, a security check causes the Engine to receive a manifest it cannot use and the pull fails.

Registry manifest support
Historically, the registry has supported a single manifest type known as Schema 1.

With the move toward multiple architecture images, the distribution project introduced two new manifest types: Schema 2 manifests and manifest lists. Registry 2.3 supports all three manifest types and sometimes performs an on-the-fly transformation of a manifest before serving the JSON in the response, to preserve compatibility with older versions of Docker Engine.

This conversion has some implications for pulling manifests by digest and this document enumerates these implications.

Content Addressable Storage (CAS)
Manifests are stored and retrieved in the registry by keying off a digest representing a hash of the contents. One of the advantages provided by CAS is security: if the contents are changed, then the digest no longer matches. This prevents any modification of the manifest by a MITM attack or an untrusted third party.

When a manifest is stored by the registry, this digest is returned in the HTTP response headers and, if events are configured, delivered within the event. The manifest can either be retrieved by the tag, or this digest.

For registry versions 2.2.1 and below, the registry always stores and serves Schema 1 manifests. Engine 1.10 first attempts to send a Schema 2 manifest, falling back to sending a Schema 1 type manifest when it detects that the registry does not support the new version.

Registry v2.3
Manifest push with Docker 1.10
The Engine constructs a Schema 2 manifest which the registry persists to disk.

When the manifest is pulled by digest or tag with Docker Engine 1.10, a Schema 2 manifest is returned. Docker Engine 1.10 understands the new manifest format.

When the manifest is pulled by tag with Docker Engine 1.9 and older, the manifest is converted on-the-fly to Schema 1 and sent in the response. The Docker Engine 1.9 is compatible with this older format.

When the manifest is pulled by digest with Docker Engine 1.9 and older, the same rewriting process does not happen in the registry. If it did, the digest would no longer match the hash of the manifest and would violate the constraints of CAS.

For this reason if a manifest is pulled by digest from a registry 2.3 with Docker Engine 1.9 and older, and the manifest was pushed with Docker Engine 1.10, a security check causes the Engine to receive a manifest it cannot use and the pull fails.

Manifest push with Docker 1.9 and older
The Docker Engine constructs a Schema 1 manifest which the registry persists to disk.

When the manifest is pulled by digest or tag with any Docker version, a Schema 1 manifest is returned.

Docker release notes
Find out what’s new in Docker! Release notes contain information about new features, improvements, known issues, and bug fixes in each release. You can find release notes for each component in the Manuals section. We suggest that you regularly visit the release notes to learn about updates.

Docker Engine
Docker Desktop for Mac
Docker Desktop for Windows
Docker Hub
Docker Compose

Docker Machine overview

You can use Docker Machine to:

Install and run Docker on Mac or Windows
Provision and manage multiple remote Docker hosts
Provision Swarm clusters
What is Docker Machine?
Docker Machine is a tool that lets you install Docker Engine on virtual hosts, and manage the hosts with docker-machine commands. You can use Machine to create Docker hosts on your local Mac or Windows box, on your company network, in your data center, or on cloud providers like Azure, AWS, or DigitalOcean.

Using docker-machine commands, you can start, inspect, stop, and restart a managed host, upgrade the Docker client and daemon, and configure a Docker client to talk to your host.

Point the Machine CLI at a running, managed host, and you can run docker commands directly on that host. For example, run docker-machine env default to point to a host called default, follow on-screen instructions to complete env setup, and run docker ps, docker run hello-world, and so forth.

Machine was the only way to run Docker on Mac or Windows previous to Docker v1.12. Starting with the beta program and Docker v1.12, Docker Desktop for Mac and Docker Desktop for Windows are available as native apps and the better choice for this use case on newer desktops and laptops. We encourage you to try out these new apps.

If you aren’t sure where to begin, see Get Started with Docker, which guides you through a brief end-to-end tutorial on Docker.

Why should I use it?
Docker Machine enables you to provision multiple remote Docker hosts on various flavors of Linux.

Additionally, Machine allows you to run Docker on older Mac or Windows systems, as described in the previous topic.

Docker Machine has these two broad use cases.

I have an older desktop system and want to run Docker on Mac or Windows

Docker Machine on Mac and Windows

If you work primarily on an older Mac or Windows laptop or desktop that doesn’t meet the requirements for the new Docker Desktop for Mac and Docker Desktop for Windows apps, then you need Docker Machine to run Docker Engine locally.

I want to provision Docker hosts on remote systems

Docker Machine for provisioning multiple systems

Docker Engine runs natively on Linux systems. If you have a Linux box as your primary system, and want to run docker commands, all you need to do is download and install Docker Engine. However, if you want an efficient way to provision multiple Docker hosts on a network, in the cloud or even locally, you need Docker Machine.

Whether your primary system is Mac, Windows, or Linux, you can install Docker Machine on it and use docker-machine commands to provision and manage large numbers of Docker hosts. It automatically creates hosts, installs Docker Engine on them, then configures the docker clients. Each managed host (“machine”) is the combination of a Docker host and a configured client.

What’s the difference between Docker Engine and Docker Machine?
When people say “Docker” they typically mean Docker Engine, the client-server application made up of the Docker daemon, a REST API that specifies interfaces for interacting with the daemon, and a command line interface (CLI) client that talks to the daemon (through the REST API wrapper). Docker Engine accepts docker commands from the CLI, such as docker run <image>, docker ps to list running containers, docker image ls to list images, and so on.

Docker Engine

Docker Machine is a tool for provisioning and managing your Dockerized hosts (hosts with Docker Engine on them). Typically, you install Docker Machine on your local system. Docker Machine has its own command line client docker-machine and the Docker Engine client, docker. You can use Machine to install Docker Engine on one or more virtual systems. These virtual systems can be local (as when you use Machine to install and run Docker Engine in VirtualBox on Mac or Windows) or remote (as when you use Machine to provision Dockerized hosts on cloud providers). The Dockerized hosts themselves can be thought of, and are sometimes referred to as, managed “machines”.

Docker Machine

Where to go next
Install Docker Machine
Create and run a Docker host on your local system using VirtualBox
Provision multiple Docker hosts on your cloud provider
Getting started with swarm mode
Understand Machine concepts
Docker Machine driver reference
Docker Machine subcommand reference
Migrate from Boot2Docker to Docker Machine
  
  Install Docker Machine
  
  Install Docker Machine
Install Docker.

Download the Docker Machine binary and extract it to your PATH.

If you are running macOS:

 base=https://github.com/docker/machine/releases/download/v0.16.0 \
  && curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/usr/local/bin/docker-machine \
  && chmod +x /usr/local/bin/docker-machine
If you are running Linux:

 base=https://github.com/docker/machine/releases/download/v0.16.0 \
  && curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine \
  && sudo mv /tmp/docker-machine /usr/local/bin/docker-machine \
  && chmod +x /usr/local/bin/docker-machine
If you are running Windows with Git BASH:

 base=https://github.com/docker/machine/releases/download/v0.16.0 \
  && mkdir -p "$HOME/bin" \
  && curl -L $base/docker-machine-Windows-x86_64.exe > "$HOME/bin/docker-machine.exe" \
  && chmod +x "$HOME/bin/docker-machine.exe"
The above command works on Windows only if you use a terminal emulator such as Git BASH, which supports Linux commands like chmod.

Otherwise, download one of the releases from the docker/machine release page directly.

Check the installation by displaying the Machine version:

 docker-machine version
Install bash completion scripts
The Machine repository supplies several bash scripts that add features such as:

command completion
a function that displays the active machine in your shell prompt
a function wrapper that adds a docker-machine use subcommand to switch the active machine
Confirm the version and save scripts to /etc/bash_completion.d or /usr/local/etc/bash_completion.d:

base=https://raw.githubusercontent.com/docker/machine/v0.16.0
for i in docker-machine-prompt.bash docker-machine-wrapper.bash docker-machine.bash
do
  sudo wget "$base/contrib/completion/bash/${i}" -P /etc/bash_completion.d
done
Then you need to run source /etc/bash_completion.d/docker-machine-prompt.bash in your bash terminal to tell your setup where it can find the file docker-machine-prompt.bash that you previously downloaded.

To enable the docker-machine shell prompt, add $(__docker_machine_ps1) to your PS1 setting in ~/.bashrc.

PS1='[\u@\h \W$(__docker_machine_ps1)]\$ '
You can find additional documentation in the comments at the top of each script.

How to uninstall Docker Machine
To uninstall Docker Machine:

Optionally, remove the machines you created.

To remove each machine individually: docker-machine rm <machine-name>

To remove all machines: docker-machine rm -f $(docker-machine ls -q) (you might need to use -force on Windows).

Removing machines is an optional step because there are cases where you might want to save and migrate existing machines to a Docker for Mac or Docker Desktop for Windows environment, for example.

Remove the executable: rm $(which docker-machine)

Note

As a point of information, the config.json, certificates, and other data related to each virtual machine created by docker-machine is stored in ~/.docker/machine/machines/ on Mac and Linux and in ~\.docker\machine\machines\ on Windows. We recommend that you do not edit or remove those files directly as this only affects information for the Docker CLI, not the actual VMs, regardless of whether they are local or on remote servers.

Where to go next
Docker Machine overview
Create and run a Docker host on your local system using virtualization
Provision multiple Docker hosts on your cloud provider
Docker Machine driver reference
Docker Machine subcommand reference
  
  Docker Machine release notes
  
  0.16.2 (2019-09-02)
General
Compile with Golang 1.12.9.
Fix VTX detection.
Drivers
amazonec2

Add ssh-port flag
Add eu-north-1 zone
vmwarefusion

Add nonempty flag to fix share-folder bug
hyperv

Allow localized names for Virtual Switch
google

Add support for non-default service account
openstack

Add flag for metadata
softlayer

Don’t set the request method again
0.16.0 (2018-11-08)
General
Updated the default storage driver to overlay2 for several systems.
Improved error reporting for the ssh subcommand when using the --native-ssh flag.
Drivers
amazonec2

Improved handling of VPC errors.
openstack

Machine removal no longer fails upon attempting to delete a nonexistent keypair.
0.15.0 (2018-06-12)
General
docker-machine can now be installed using go install.
Docker Machine is now built with Go 1.10.
SSH connections now include a keep alive option. (docker/machine #4450)
Drivers
amazonec2

Updated default AMIs to mitigate Meltdown and Spectre.
Added --amazonec2-security-group-readonly flag to prevent mutating security groups.
exoscale

Updated driver to v0.9.23.
hyperv

Fixed Hyper-V precreate issues. (docker/machine #4426)
Added the ability to disable Hyper-V dynamic memory management during VM creation with --hyperv-disable-dynamic-memory.
vmwarefusion

Improved shell checks (docker/machine #4491).
0.14.0 (2018-03-06)
General
Added --client-certs flag to the docker-machine regenerate-certs command.
Improved OpenBSD support.
Fixed a bug with scp commands issued from a Windows host.
Enabled progress output by default for scp commands using rsync.
Added --quiet flag to scp to suppress progress output.
Machine now uses the ss command to detect connectivity when netstat is unavailable.
Added bash completion for docker-machine mount.
Improved provisioning resilience on Debian-based hosts.
Drivers
amazonec2

Added support for eu-west-3 region
Upon failure, the create command now ensures dangling resources are cleaned up before exiting
Machine creation no longer fails when waiting on spot instance readiness
digitalocean

Added --digitalocean-monitoring flag
Increased the default droplet size
exoscale

Updated driver library
Several improvements and fixes to the default machine template
Added support for user-provided SSH key (--exoscale-ssh-key)
Added support for arbitrary disk size
google

Enabled disk auto-deletion on newly created machines.
Fixed a bug preventing the removal of a machine if it had already been removed remotely.
Added support for fully qualified network and subnetwork names.
hyperv

Fixed potential cmdlet collision with VMWare powercli.
Fixed a bug with virtual switch selection.
Machine now correctly detects if the user is a Hyper-V administrator when using a localized version of Windows.
openstack

Added --openstack-config-drive flag
Fixed an issue causing some user-uploaded keypairs to be removed when removing the associated machine.
Fixed a bug preventing the removal of a machine if it had already been removed remotely.
virtualbox

Added OpenBSD support
vmwarefusion

Improved error detection and reporting when creating a new instance.
vmwarevsphere

Added --vmwarevsphere-folder flag.
0.13.0 (2017-10-12)
General
Added new docker-machine mount command for mounting machine directories over SSHFS.
Improved some logging messages.
Fixed a bug with the scp command when using an identity file.
Fixed a parsing error that caused the boot2docker ISO cache to malfunction, forcing a new download everytime.
Drivers
azure
docker-machine rm now also cleans up the associated storage account if it has no remaining storage containers.
The creation process will no longer recreate the associated subnet if it already it exists.
exoscale
Updated driver.
Removed default docker-machine affinity group if no other affinity group was specified.
virtualbox
Fixed a bug where the machine would sometimes be assigned an invalid IP address at creation time.
vmwaresphere
Added support for multiple networks.
0.12.2 (2017-7-12)
General
The scp sub-command now allows to provide an optional user@ to the address.
Fixed bash completion on OS X.
Drivers
amazonec2
Updated default AMIs to the latest version of Ubuntu 16.04 LTS.
Fixed a bug preventing proper machine removal.
vmwarevsphere
Creating VMs on a DRS-enabled cluster should now work properly.
Fixed a bug that prevented provisioning.
vmwarefusion
Fixed a bug that prevented provisioning.
exoscale
Updated library.
0.12.1 (2017-6-30)
General
Fixed an issue with the version comparison function that prevented machines created with Engine 17.06.0 from starting properly.
0.12.0 (2017-6-5)
General
Various bash completion improvements.
Bump Go to version 1.8.3.
Drivers
openstack
Enable HTTP_PROXY
digitalocean
Add support for tagging.
virtualbox
Scope DHCP address range based on CIDR
generic
Increase default timeout
google
Add subnetwork support
Provisioners
Remove restriction on --engine-install-url in default-to-boot2docker drivers (virtualbox, vmwarefusion, etc.)
Reduce provisioning time of SUSE/openSUSE systems
0.11.0 (2017-4-25)
General
Various bugfixes and updated library dependencies
new docker-machine scp --delta to invoke rsync behind the scenes for more efficient transfer
Drivers
digitalocean
Add support for tagging DigitalOcean instances.
google
Add support for subnetworks
0.10.0 (2017-2-27)
General
Various improvements to shell tab completion
Add support for compiling on ARM64 architecture
Drivers
Make virtualbox default driver
amazonec2
Update AMIs to latest version of Ubuntu 16.04 LTS
virtualbox
Fix parsing of --virtualbox-share-folder on Windows
google
Add --google-open-port flag to specify additional ports to open
Provisioners
Machine now uses systemd drop-in files instead of over-writing the system units
Add support for upgrade to new Docker versioning scheme
Use dockerd only in Docker versions where it is available
Support multiple architectures in SUSE provisioner
0.9.0 (2017-1-17)
General
On Windows, the COMPOSE_CONVERT_WINDOWS_PATHS environment variable is now set by docker-machine env to improve Compose usability.
Docker Machine can now be built on FreeBSD
docker-machine scp non-22 port support
scp supports SSH agent
Bump Go version to 1.7.4
Drivers
amazonec2
Credentials can now be loaded from IAM instance profiles
Add --amazonec2-userdata flag
Add --amazonec2-block-duration-minutes flag
Add support for us-east-2 (Ohio)
Update base images to Ubuntu 16.04
azure
Add --azure-dns flag for specifying DNS names
Add --azure-storage-type flag
Allow using vnets from another resource group
Add AzureGermanCloud support
Add support for custom data
Support Service Principal authentication
Update base images to Ubuntu 16.04
digitalocean
Add ability to specify the private SSH key path
gce
Update base images to Ubuntu 16.04
virtualbox
Shared folder location can be specified instead of “hardcoded” to C:\Users or /Users
openstack
Add support for OS_CACERT
Provisioners
OpenSUSE provisioner refactored to use properly supported 3rd party code
# 0.8.2 (2016-8-26)
Update Go version to 1.7.1
0.8.1 (2016-8-20)
Provisioners
Fix issue with generated systemd service file on RedHat family distros
Drivers
azure
Bump Ubuntu image to 16.04
Update docs with updated default parameters
Change logging slightly
0.8.0 (2016-6-14)
General
Fix issue with plugin heartbeat log repeating on disconnect
Add tcsh support to env --shell
Add zsh completion scripts
Bump Go version to 1.6.2
Drivers
amazonec2
Workaround to prevent orphaned SSH keys
virtualbox
Add option for VM UI type (--virtualbox-ui-type)
vmwarefusion
Fix CPU option inconsistency
openstack
Expose user data parameter (--openstack-user-data-file)
generic
Copy public key to created Machine directory
Provisioners
Add Oracle Enterprise Linux support
Fix port binding of Swarm master
Add ability to create a manager instance which does not get scheduled on
Introduce --swarm-join-opt to pass options to agent nodes
Various SSH-related fixes
Fix state for upgrade path
0.7.0 (2016-4-13)
General
DRIVER environment variable now supported to supply value for create --driver flag
Update to Go 1.6.1
SSH client has been refactored
RC versions of Machine will now create and upgrade to boot2docker RCs instead of stable versions if available
Drivers
azure
Driver has been completely re-written to use resource templates and a significantly easier-to-use authentication model
digitalocean
New --digitalocean-ssh-key-fingerprint for using existing SSH keys instead of creating new ones
virtualbox
Fix issue with bootlocal.sh
New --virtualbox-nictype flag to set driver for NAT network
More robust host-only interface collision detection
Add support for running VirtualBox on a Windows 32 bit host
Change default DNS passthrough handling
amazonec2
Specifying multiple security groups to use is now supported
exoscale
Add support for user-data
hyperv
Machines can now be created by a non-administrator
rackspace
New --rackspace-active-timeout parameter
vmwarefusion
Bind mount shared folder directory by default
google
New --google-use-internal-ip-only parameter
Provisioners
General
Support for specifying Docker engine port in some cases
CentOS
Now defaults to using upstream get.docker.com script instead of custom RPMs.
boot2docker
More robust eth* interface detection
Swarm
Add --swarm-experimental parameter to enable experimental Swarm features
0.6.0 (2016-02-04)
Fix SSH wait before provisioning issue
0.6.0-rc4 (2016-02-03)
General
env
Fix shell auto detection
Drivers
exoscale
Fix configuration of exoscale endpoint
0.6.0-rc3 (2016-02-01)
Exit with code 3 if error is during pre-create check
0.6.0-rc2 (2016-01-28)
Fix issue creating Swarms
Fix ls header issue
Add code to wait for Docker daemon before returning from start / restart
Start porting integration tests to Go from BATS
Add Appveyor for Windows tests
Update CoreOS provisioner to use docker daemon
Various documentation and error message fixes
Add ability to create GCE machine using existing VM
0.6.0-rc1 (2016-01-18)
General
Update to Go 1.5.3
Short form of command invocations is now supported
docker-machine start, docker-machine stop and others will now use default as the machine name argument if one is not specified
Fix issue with panics in drivers
Machine now returns exit code 3 if the pre-create check fails.
This is potentially useful for scripting docker-machine.
docker-machine provision command added to allow re-running of provisioning on instances.
This allows users to re-run provisioning if it fails during create instead of needing to completely start over.
Provisioning
Most provisioners now use docker daemon instead of docker -d
Swarm masters now run with replication enabled
If /var/lib is a BTRFS partition, btrfs will now be used as the storage driver for the instance
Drivers
Amazon EC2
Default VPC will be used automatically if none is specified
Credentials are now be read from the conventional ~/.aws/credentials file automatically
Fix a few issues such as nil pointer dereferences
VMware Fusion
Try to get IP from multiple DHCP lease files
OpenStack
Only derive tenant ID if tenant name is supplied
0.5.6 (2016-01-11)
General
create
Set swarm master to advertise on port 3376
Fix swarm restart policy
Stop asking for ssh key passwords interactively
env
Improve documentation
Fix bash on windows
Automatic shell detection on Windows
help
Don’t show the full path to docker-machine.exe on windows
ls
Allow custom format
Improve documentation
restart
Improve documentation
rm
Improve documentation
Better user experience when removing multiple hosts
version
Don’t show the full path to docker-machine.exe on windows
start, stop, restart, kill
Better logs and homogeneous behaviour across all drivers
Build
Introduce CI tests for external binary compatibility
Add amazon EC2 integration test
Misc
Improve BugSnags reports: better shell detection, better windows version detection
Update DockerClient dependency
Improve bash-completion script
Improve documentation for bash-completion
Drivers
Amazon EC2
Improve documentation
Support optional tags
Option to create EbsOptimized instances
Google
Fix remove when instance is stopped
Openstack
Flags to import and reuse existing nova keypairs
VirtualBox
Fix multiple bugs related to host-only adapters
Retry commands when VBoxManage is not ready
Reject VirtualBox versions older that 4.3
Fail with a clear message when Hyper-v installation prevents VirtualBox from working
Print a warning for Boot2Docker v1.9.1, which is known to have an issue with AUFS
Vmware Fusion
Support soft links in VM paths
Libmachine
Fix code sample that uses libmachine
libmachine can be used in external applications
0.5.5 (2015-12-28)
General
env
Better error message if swarm is down
Add quotes to command if there are spaces in the path
Fix Powershell env hints
Default to cmd shell on windows
Detect fish shell
scp
Ignore empty ssh key
stop, start, kill
Add feedback to the user
rm
Now works when config.json is not found
ssh
Disable ControlPath
Log which SSH client is used
ls
Listing is now faster by reducing calls to the driver
Shows if the active machine is a swarm cluster
Build
Automate 90% of the release process
Upgrade to Go 1.5.2
Don’t build 32bits binaries for Linux and OSX
Prevent makefile from defaulting to using containers
Misc
Update docker-machine version
Updated the bash completion with new options added
Bugsnag: Retrieve windows version on non-English OS
Drivers
Amazon EC2
Convert API calls to official SDK
Make DeviceName configurable
DigitalOcean
Custom SSH port support
Generic
Don’t support kill since stop is not supported
Google
Coreos provisionning
Hyper-V
Lot’s of code simplifications
Pre-Check that the user is an Administrator
Pre-Check that the virtual switch exists
Add Environment variables for each flag
Fix how Powershell is detected
VSwitch name should be saved to config.json
Add a flag to set the CPU count
Close handle after copying boot2docker.iso into vm folder - will otherwise keep hyper-v from starting vm
Update Boot2Docker cache in PreCreateCheck phase
OpenStack
Filter floating IPs by tenant ID
VirtualBox
Reject duplicate hostonlyifs Name/IP with clear message
Detect when hostonlyif can’t be created. Point to known working version of VirtualBox
Don’t create the VM if no hardware virtualization is available and add a flag to force create
Add VBox.log to bugsnag crashreport
Update Boot2Docker cache in PreCreateCheck phase
Detect Incompatibility with Hyper-v
VSphere
Rewrite driver to work with govmomi instead of wrapping govc
All
Change host restart to use the driver implementation
Fix truncated logs
Increase heartbeat interval and timeout
Provisioners
Download latest Boot2Docker if it is out-of-date
Add swarm config to coreos
All provisioners now honor engine-install-url
0.5.4 (2015-12-28)
This is a patch release to fix a regression with STDOUT/STDERR behavior (#2587).

0.5.3 (2015-12-14)
With this release Machine reverts to distribution in a single binary, which is more efficient on bandwidth and hard disk space. All the core driver plugins are now included in the main binary. Delete the old driver binaries that you might have in your path.

 rm /usr/local/bin/docker-machine-driver-{amazonec2,azure,digitalocean,exoscale,generic,google,hyperv,none,openstack,rackspace,softlayer,virtualbox,vmwarefusion,vmwarevcloudair,vmwarevsphere}
Non-core driver plugins should still work as intended (in externally distributed binaries of the form docker-machine-driver-name. Report any issues you encounter them with externally loaded plugins.

General
Optionally report crashes to Bugsnag to help us improve docker-machine
Fix multiple nil dereferences in docker-machine ls command
Improve the build and CI
docker-machine env now supports emacs
Run Swarm containers in provisioning step using Docker API instead of SSH/shell
Show docker daemon version in docker-machine ls
docker-machine ls can filter by engine label
docker-machine ls filters are case insensitive
--timeout flag for docker-machine ls
Logs use logrus library
Swarm container network is now host
Added advertise flag to Swarm manager template
Fix help flag for docker-machine ssh
Add confirmation -y flag to docker-machine rm
Fix docker-machine config for fish
Embed all core drivers in docker-machine binary to reduce the bundle from 120M to 15M
Drivers
Generic
Support password protected ssh keys though ssh-agent
Support DNS names
VirtualBox
Show a warning if virtualbox is too old
Recognize yet another Hardware Virtualization issue pattern
Fix Hardware Virtualization on Linux/AMD
Add the --virtualbox-host-dns-resolver flag
Allow virtualbox DNSProxy override
Google
Open firewall port for Swarm when needed
VMware Fusion
Explicitly set umask before invoking vmrun in vmwarefusion
Activate the plugin only on OSX
Add id/gid option to mount when using vmhgfs
Fix for vSphere driver boot2docker ISO issues
DigitalOcean
Support for creating Droplets with Cloud-init User Data
Openstack
Sanitize keynames by replacing dots with underscores
All
Most base images are now set to Ubuntu 15.10
Fix compatibility with drivers developed with docker-machine 0.5.0
Better error report for broken/incompatible drivers
Don’t break config.json configuration when the disk is full
Provisioners
Increase timeout for installing boot2docker
Support Ubuntu 15.10
Misc
Improve the documentation
Update known drivers list
0.5.2 (2015-11-30)
General
Bash autocompletion and helpers fixed
Remove RawDriver from config.json - Driver parameters can now be edited directly again in this file.
Change fish env variable setting to be global
Add docker-machine version command
Move back to normal codegangsta/cli upstream
--tls-san flag for extra SANs
Drivers
Fix GetURL IPv6 compatibility
Add documentation page for available 3rd party drivers
VirtualBox
Support for shared folders and virtualization detection on Linux hosts
Improved detection of invalid host-only interface settings
Google
Update default images
VMware Fusion
Add option to disable shared folder
Generic
New environment variables for flags
Provisioners
Support for Ubuntu >=15.04. This means Ubuntu machines can be created which work with overlay driver of lib network.
Fix issue with current netstat / daemon availability checking
0.5.1 (2015-11-16)
Fixed boot2docker VM import regression
Fix regression breaking docker-machine env -u to unset environment variables
Enhanced virtualization capability detection and VBoxManage path detection
Properly lock VirtualBox access when running several commands concurrently
Allow plugins to write to STDOUT without --debug enabled
Fix Rackspace driver regression
Support colons in docker-machine scp filepaths
Pass environment variables for provisioned Engines to Swarm as well
Various enhancements around boot2docker ISO upgrade (progress bar, increased timeout)
0.5.0 (2015-11-1)
General
-   Add pluggable driver model
-   Clean up code to be more modular and reusable in `libmachine`
-   Add `--github-api-token` for situations where users are getting rate limited
    by GitHub attempting to get the current `boot2docker.iso` version
-   Various enhancements around the Makefile and build toolchain (still an active WIP)
-   Disable SSH multiplex explicitly in commands run with the "External" client
-   Show "-" for "inactive" machines instead of nothing
-   Make daemon status detection more robust ### Provisioners
-   New CoreOS, SUSE, and Arch Linux provisioners
-   Fixes around package installation / upgrade code on Debian and Ubuntu ### CLI
-   Support for regular expression pattern matching and matching by names in `ls --filter`
-   `--no-proxy` flag for `env` (sets `NO_PROXY` in addition to other environment variables) ### Drivers
-   `openstack`
    -   `--openstack-ip-version` parameter
    -   `--openstack-active-timeout` parameter
-   `google`
    -   fix destructive behavior of `start` / `stop`
-   `hyperv`
    -   fix issues with PowerShell
-   `vmwarefusion`
    -   some issues with shared folders fixed
    -   `--vmwarefusion-configdrive-url` option for configuration via `cloud-init`
-   `amazonec2`
    -   `--amazonec2-use-private-address` option to use private networking
-   `virtualbox`
    -   Enhancements around robustness of the created host-only network
    -   Fix IPv6 network mask prefix parsing
    -   `--virtualbox-no-share` option to disable the automatic home directory mount
    -   `--virtualbox-hostonly-nictype` and `--virtualbox-hostonly-nicpromisc` for controlling settings around the created hostonly NIC
0.4.1 (2015-08)
Fixes upgrade functionality on Debian based systems
Fixes upgrade functionality on Ubuntu based systems
0.4.0 (2015-08-11)
Updates
HTTP Proxy support for Docker Engine
RedHat distros now use Docker Yum repositories
Ability to set environment variables in the Docker Engine
Internal libmachine updates for stability
Drivers
Google:
Preemptible instances
Static IP support
Fixes
Swarm Discovery Flag is verified
Timeout added to ls command to prevent hangups
SSH command failure now reports information about error
Configuration migration updates
0.3.0 (2015-06-18)
Features
Engine option configuration (ability to configure all engine options)
Swarm option configuration (ability to configure all swarm options)
New Provisioning system to allow for greater flexibility and stability for installing and configuring Docker
New Provisioners
Rancher OS
RedHat Enterprise Linux 7.0+ (experimental)
Fedora 21+ (experimental)
Debian 8+ (experimental)
PowerShell support (configure Windows Docker CLI)
Command Prompt (cmd.exe) support (configure Windows Docker CLI)
Filter command help by driver
Ability to import Boot2Docker instances
Boot2Docker CLI migration guide (experimental)
Format option for inspect command
New logging output format to improve readability and display across platforms
Updated “active” machine concept - now is implicit according to DOCKER_HOST environment variable. Note: this removes the implicit “active” machine and can no longer be specified with the active command. You change the “active” host by using the env command instead.
Specify Swarm version (--swarm-image flag)
Drivers
New: Exoscale Driver
New: Generic Driver (provision any host with supported base OS and SSH)
Amazon EC2
SSH user is configurable
Support for Spot instances
Add option to use private address only
Base AMI updated to 20150417
Google
Support custom disk types
Updated base image to v20150316
Openstack
Support for Keystone v3 domains
Rackspace
Misc fixes including environment variable for Flavor Id and stability
Softlayer
Enable local disk as provisioning option
Fixes for SSH access errors
Fixed bug where public IP would always be returned when requesting private
Add support for specifying public and private VLAN IDs
VirtualBox
Use Intel network interface driver (adds great stability)
Stability fixes for NAT access
Use DNS pass through
Default CPU to single core for improved performance
Enable shared folder support for Windows hosts
VMware Fusion
Boot2Docker ISO updated
Shared folder support
Fixes
Provisioning improvements to ensure Docker is available
SSH improvements for provisioning stability
Fixed SSH key generation bug on Windows
Help formatting for improved readability
Breaking Changes
“Short-Form” name reference no longer supported Instead of “docker-machine “ implying the active host you must now use docker-machine
VMware shared folders require Boot2Docker 1.7
Special Thanks
We would like to thank all contributors. Machine would not be where it is without you. We would also like to give special thanks to the following contributors for outstanding contributions to the project:

@frapposelli for VMware updates and fixes
@hairyhenderson for several improvements to Softlayer driver, inspect formatting and lots of fixes
@ibuildthecloud for rancher os provisioning
@sthulb for portable SSH library
@vincentbernat for exoscale
@zchee for Amazon updates and great doc updates
0.2.0 (2015-04-16)
Core Stability and Driver Updates

Core
Support for system proxy environment
New command to regenerate TLS certificates
Note: this will restart the Docker engine to apply
Updates to driver operations (create, start, stop, etc) for better reliability
New internal libmachine package for internal api (not ready for public usage)
Updated Driver Interface
Driver Spec
Removed host provisioning from Drivers to enable a more consistent install
Removed SSH commands from each Driver for more consistent operations
Swarm: machine now uses Swarm default binpacking strategy
Driver Updates
All drivers updated to new Driver interface
Amazon EC2
Better checking for subnets on creation
Support for using Private IPs in VPC
Fixed bug with duplicate security group authorization with Swarm
Support for IAM instance profile
Fixed bug where IP was not properly detected upon stop
DigitalOcean
IPv6 support
Backup option
Private Networking
Openstack / Rackspace
Gophercloud updated to latest version
New insecure flag to disable TLS (use with caution)
Google
Google source image updated
Ability to specify auth token via file
VMware Fusion
Paravirtualized driver for disk (pvscsi)
Enhanced paravirtualized NIC (vmxnet3)
Power option updates
SSH keys persistent across reboots
Stop now gracefully stops VM
vCPUs now match host CPUs
SoftLayer
Fixed provision bug where curl was not present
VirtualBox
Correct power operations with Saved VM state
Fixed bug where image option was ignored
CLI
Auto-regeneration of TLS certificates when TLS error is detected
Note: this will restart the Docker engine to apply
Minor UI updates including improved sorting and updated command docs
Bug with config and env with spaces fixed
Note: you now must use eval $(docker-machine env machine) to load environment settings
Updates to better support fish shell
Use --tlsverify for both config and env commands
Commands now use eval for better interoperability with shell
Testing
New integration test framework (bats)
0.1.0 (2015-02-26)
Initial beta release.

Provision Docker Engines using multiple drivers
Provide light management for the machines
Create, Start, Stop, Restart, Kill, Remove, SSH
Configure the Docker Engine for secure communication (TLS)
Easily switch target machine for fast configuration of Docker Engine client
Provision Swarm clusters (experimental)
Included drivers
Amazon EC2
DigitalOcean
Google
Microsoft Azure
Microsoft Hyper-V
Openstack
Rackspace
VirtualBox
VMware Fusion
VMware vCloud Air
VMware vSpher
  
  Get started with Docker Machine and a local VM
  Let’s take a look at using docker-machine to create, use, and manage a Docker host inside of a local virtual machine.

Note

Docker Machine is no longer included in the Docker Desktop installer. You can download it separately from the docker/machine release page on GitHub.

Prerequisite information
With the advent of Docker Desktop for Mac and Docker Desktop for Windows, we recommend that you use these for your primary Docker workflows. You can use these applications to run Docker natively on your local system without using Docker Machine at all.

For now, however, if you want to create multiple local machines, you still need Docker Machine to create and manage machines for multi-node experimentation.

The new solutions come with their own native virtualization solutions rather than Oracle VirtualBox, so keep the following considerations in mind when using Machine to create local VMs.

Docker Desktop for Mac - You can use docker-machine create with the virtualbox driver to create additional local machines.

Docker Desktop for Windows - You can use docker-machine create with the hyperv driver to create additional local machines.

If you are using Docker Desktop for Windows
Docker Desktop for Windows uses Microsoft Hyper-V for virtualization, and Hyper-V is not compatible with Oracle VirtualBox. Therefore, you cannot run the two solutions simultaneously. But you can still use docker-machine to create more local VMs by using the Microsoft Hyper-V driver.

The prerequisites are:

Have Docker Desktop for Windows installed, and running (which requires that virtualization and Hyper-V are enabled, as described in What to know before you install Docker Desktop for Windows).

Set up the Hyper-V driver to use an external virtual network switch See the Docker Machine driver for Microsoft Hyper-V topic, which includes an example of how to do this.

If you are using Docker Desktop for Mac
Docker Desktop for Mac uses HyperKit, a lightweight macOS virtualization solution built on top of the Hypervisor.framework.

Currently, there is no docker-machine create driver for HyperKit, so use the virtualbox driver to create local machines. (See the Docker Machine driver for Oracle VirtualBox.) You can run both HyperKit and Oracle VirtualBox on the same system.

Make sure you have the latest VirtualBox correctly installed on your system.
Use Machine to run Docker containers
To run a Docker container, you:

create a new (or start an existing) Docker virtual machine
switch your environment to your new VM
use the docker client to create, load, and manage containers
Once you create a machine, you can reuse it as often as you like. Like any VirtualBox VM, it maintains its configuration between uses.

The examples here show how to create and start a machine, run Docker commands, and work with containers.

Create a machine
Open a command shell or terminal window.

These command examples shows a Bash shell. For a different shell, such as C Shell, the same commands are the same except where noted.

Use docker-machine ls to list available machines.

In this example, no machines have been created yet.

 $ docker-machine ls
 NAME   ACTIVE   DRIVER   STATE   URL   SWARM   DOCKER   ERRORS
Create a machine.

Run the docker-machine create command, pass the appropriate driver to the --driver flag and provide a machine name. If this is your first machine, name it default as shown in the example. If you already have a “default” machine, choose another name for this new machine.

On Docker Desktop for Windows systems that support Hyper-V, use the hyperv driver as shown in the Docker Machine Microsoft Hyper-V driver reference and follow the example, which shows how to use an external network switch and provides the flags for the full command. (See prerequisites above to learn more.)

     $ docker-machine create --driver virtualbox default
     Running pre-create checks...
     Creating machine...
     (staging) Copying /Users/ripley/.docker/machine/cache/boot2docker.iso to /Users/ripley/.docker/machine/machines/default/boot2docker.iso...
     (staging) Creating VirtualBox VM...
     (staging) Creating SSH key...
     (staging) Starting the VM...
     (staging) Waiting for an IP...
     Waiting for machine to be running, this may take a few minutes...
     Machine is running, waiting for SSH to be available...
     Detecting operating system of created instance...
     Detecting the provisioner...
     Provisioning with boot2docker...
     Copying certs to the local machine directory...
     Copying certs to the remote machine...
     Setting Docker configuration on the remote daemon...
     Checking connection to Docker...
     Docker is up and running!
     To see how to connect Docker to this machine, run: docker-machine env default
This command downloads a lightweight Linux distribution (boot2docker) with the Docker daemon installed, and creates and starts a VirtualBox VM with Docker running.

List available machines again to see your newly minted machine.

 $ docker-machine ls
 NAME      ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER   ERRORS
 default   *        virtualbox   Running   tcp://192.168.99.187:2376           v1.9.1
Get the environment commands for your new VM.

As noted in the output of the docker-machine create command, you need to tell Docker to talk to the new machine. You can do this with the docker-machine env command.

 $ docker-machine env default
 export DOCKER_TLS_VERIFY="1"
 export DOCKER_HOST="tcp://172.16.62.130:2376"
 export DOCKER_CERT_PATH="/Users/<yourusername>/.docker/machine/machines/default"
 export DOCKER_MACHINE_NAME="default"
 # Run this command to configure your shell:
 # eval "$(docker-machine env default)"
Connect your shell to the new machine.

 $ eval "$(docker-machine env default)"
Note: If you are using fish, or a Windows shell such as Powershell/cmd.exe, the above method does not work as described. Instead, see the env command’s documentation to learn how to set the environment variables for your shell.

This sets environment variables for the current shell that the Docker client reads which specify the TLS settings. You need to do this each time you open a new shell or restart your machine. (See also, how to unset environment variables in the current shell.)

You can now run Docker commands on this host.

Run containers and experiment with Machine commands
Run a container with docker run to verify your set up.

Use docker run to download and run busybox with a simple ‘echo’ command.

 $ docker run busybox echo hello world
 Unable to find image 'busybox' locally
 Pulling repository busybox
 e72ac664f4f0: Download complete
 511136ea3c5a: Download complete
 df7546f9f060: Download complete
 e433a6c5b276: Download complete
 hello world
Get the host IP address.

Any exposed ports are available on the Docker host’s IP address, which you can get using the docker-machine ip command:

 $ docker-machine ip default
 192.168.99.100
Run a Nginx webserver in a container with the following command:

 $ docker run -d -p 8000:80 nginx
When the image is finished pulling, you can hit the server at port 8000 on the IP address given to you by docker-machine ip. For instance:

     $ curl $(docker-machine ip default):8000
     <!DOCTYPE html>
     <html>
     <head>
     <title>Welcome to nginx!</title>
     <style>
         body {
             width: 35em;
             margin: 0 auto;
             font-family: Tahoma, Verdana, Arial, sans-serif;
         }
     </style>
     </head>
     <body>
     <h1>Welcome to nginx!</h1>
     <p>If you see this page, the nginx web server is successfully installed and
     working. Further configuration is required.</p>

     <p>For online documentation and support, refer to
     <a href="https://nginx.org">nginx.org</a>.<br/>
     Commercial support is available at
     <a href="https://www.nginx.com">www.nginx.com</a>.</p>

     <p><em>Thank you for using nginx.</em></p>
     </body>
     </html>
You can create and manage as many local VMs running Docker as your local resources permit; just run docker-machine create again. All created machines appear in the output of docker-machine ls.

Start and stop machines
If you are finished using a host for the time being, you can stop it with docker-machine stop and later start it again with docker-machine start.

    $ docker-machine stop default
    $ docker-machine start default
Operate on machines without specifying the name
Some docker-machine commands assume that the given operation should be run on a machine named default (if it exists) if no machine name is specified. Because using a local VM named default is such a common pattern, this allows you to save some typing on the most frequently used Machine commands.

For example:

      $ docker-machine stop
      Stopping "default"....
      Machine "default" was stopped.

      $ docker-machine start
      Starting "default"...
      (default) Waiting for an IP...
      Machine "default" was started.
      Started machines may have new IP addresses.  You may need to re-run the `docker-machine env` command.

      $ eval $(docker-machine env)

      $ docker-machine ip
        192.168.99.100
Commands that follow this style are:

    - `docker-machine config`
    - `docker-machine env`
    - `docker-machine inspect`
    - `docker-machine ip`
    - `docker-machine kill`
    - `docker-machine provision`
    - `docker-machine regenerate-certs`
    - `docker-machine restart`
    - `docker-machine ssh`
    - `docker-machine start`
    - `docker-machine status`
    - `docker-machine stop`
    - `docker-machine upgrade`
    - `docker-machine url`
For machines other than default, and commands other than those listed above, you must always specify the name explicitly as an argument.

Unset environment variables in the current shell
You might want to use the current shell to connect to a different Docker Engine. In such scenarios, you have the option to switch the environment for the current shell to talk to different Docker engines.

Run env|grep DOCKER to check whether DOCKER environment variables are set.

$ env | grep DOCKER
DOCKER_HOST=tcp://192.168.99.100:2376
DOCKER_MACHINE_NAME=default
DOCKER_TLS_VERIFY=1
DOCKER_CERT_PATH=/Users/<your_username>/.docker/machine/machines/default
If it returns output (as shown in the example), you can unset the DOCKER environment variables.

Use one of two methods to unset DOCKER environment variables in the current shell.

Run the unset command on the following DOCKER environment variables.

unset DOCKER_TLS_VERIFY
unset DOCKER_CERT_PATH
unset DOCKER_MACHINE_NAME
unset DOCKER_HOST
Alternatively, run a shortcut command docker-machine env -u to show the command you need to run to unset all DOCKER variables:

$ docker-machine env -u
unset DOCKER_TLS_VERIFY
unset DOCKER_HOST
unset DOCKER_CERT_PATH
unset DOCKER_MACHINE_NAME
# Run this command to configure your shell:
# eval $(docker-machine env -u)
Run eval $(docker-machine env -u) to unset all DOCKER variables in the current shell.

Now, after running either of the above commands, this command should return no output.

 $ env | grep DOCKER
If you are running Docker Desktop for Mac, you can run Docker commands to talk to the Docker Engine installed with that app.

Start local machines on startup
To ensure that the Docker client is automatically configured at the start of each shell session, you can embed eval $(docker-machine env default) in your shell profiles, by adding it to the ~/.bash_profile file or the equivalent configuration file for your shell. However, this fails if a machine called default is not running. You can configure your system to start the default machine automatically. The following example shows how to do this in macOS.

Create a file called com.docker.machine.default.plist in the ~/Library/LaunchAgents/ directory, with the following content:

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>EnvironmentVariables</key>
        <dict>
            <key>PATH</key>
            <string>/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin</string>
        </dict>
        <key>Label</key>
        <string>com.docker.machine.default</string>
        <key>ProgramArguments</key>
        <array>
            <string>/usr/local/bin/docker-machine</string>
            <string>start</string>
            <string>default</string>
        </array>
        <key>RunAtLoad</key>
        <true/>
    </dict>
</plist>
You can change the default string above to make this LaunchAgent start a different machine.

Where to go next
Provision multiple Docker hosts on your cloud provider
Understand Machine concepts
Docker Machine list of reference pages for all supported drivers
Docker Machine driver for Oracle VirtualBox
Docker Machine driver for Microsoft Hyper-V
docker-machine command line reference
  
