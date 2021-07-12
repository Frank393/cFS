To set up nektos/act and start testing Git-action locally you first need to set up Docker.

# **Docker Setup:**

Install Docker:
`sudo apt install docker.io`

Enable Docker:
`sudo systemctl enable --now docker`

**Create Docker Image with essencials**
Create file named **Dockerfile** with no extension

```
FROM ubuntu:18.04


ENV DEBIAN_FRONTEND=noninteractive


RUN apt-get update && apt-get install build-essential -y


# Basic dependencies

RUN apt-get install -y git make sudo


# Dependencies to install git from source

RUN apt-get install -y libz-dev libssl-dev libcurl4-gnutls-dev libexpat1-dev gettext cmake gcc

RUN curl -o git.tar.gz https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.26.2.tar.gz

RUN tar -zxf git.tar.gz

RUN cd git-* && make prefix=/usr/local && make prefix=/usr/local install

RUN apt-get purge -y libz-dev libssl-dev libcurl4-gnutls-dev libexpat1-dev gettext gcc


RUN git --version

# Install dependencies for build-cfs

RUN apt-get install -y lcov


# Install dependencies for build-documentation

RUN apt-get install -y doxygen graphviz texlive-latex-base \

      texlive-fonts-recommended texlive-fonts-extra texlive-latex-extra

# Install dependencies for static-analysis

RUN apt-get install -y cppcheck
```

Authenticating to GitHub Package Registry
`docker login docker.pkg.github.com -u <username> -p <access_token>`

**Publishing a package (docker image)**

Example of building :`docker build -t docker.pkg.github.com/frank393/images/cfs_act .`

Tagging of the image:`docker.pkg.github.com/owner/repository/image_name:version`

Pushing to Github: `docker push docker.pkg.github.com/frank393/images/cfs_act`


# **Nektos/act Setup**

Install act
`curl https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash`



**Using Nektos/act with cFS**

First make sure you clone the cFS repository in your local computer.
Change directory to the cFS and now you can star using it.

Basic command for act for cFS
`act -P ubuntu-18.04=docker.pkg.github.com/frank393/images/cfs_act`
This command will run the .yml files in the .workflow directory

If you wish to only test one .yml file will use `-W` flag

example:`act -P ubuntu-18.04=docker.pkg.github.com/frank393/images/cfs_act -W ./.github/workflows/build-cfs.yml`

# **Various flags**

```
  -a, --actor string                     user that triggered the event (default "nektos/act")
  -b, --bind                             bind working directory to container, rather than copy
      --container-architecture string    Architecture which should be used to run containers, e.g.: linux/amd64. If not specified, will use host default architecture. Requires Docker server API Version 1.41+. Ignored on earlier Docker server platforms.
      --container-daemon-socket string   Path to Docker daemon socket which will be mounted to containers (default "/var/run/docker.sock")
      --defaultbranch string             the name of the main branch
      --detect-event                     Use first event type from workflow as event that triggered the workflow
  -C, --directory string                 working directory (default ".")
  -n, --dryrun                           dryrun mode
      --env stringArray                  env to make available to actions with optional value (e.g. --env myenv=foo or --env myenv)
      --env-file string                  environment file to read and use as env in the containers (default ".env")
  -e, --eventpath string                 path to event JSON file
      --github-instance string           GitHub instance to use. Don't use this if you are not using GitHub Enterprise Server. (default "github.com")
  -g, --graph                            draw workflows
  -h, --help                             help for act
      --insecure-secrets                 NOT RECOMMENDED! Doesn't hide secrets while printing logs.
  -j, --job string                       run job
  -l, --list                             list workflows
      --no-recurse                       Flag to disable running workflows from subdirectories of specified path in '--workflows'/'-W' flag
  -P, --platform stringArray             custom image to use per platform (e.g. -P ubuntu-18.04=nektos/act-environments-ubuntu:18.04)
      --privileged                       use privileged mode
  -p, --pull                             pull docker image(s) even if already present
  -q, --quiet                            disable logging of output from steps
  -r, --reuse                            reuse action containers to maintain state
  -s, --secret stringArray               secret to make available to actions with optional value (e.g. -s mysecret=foo or -s mysecret)
      --secret-file string               file with list of secrets to read from (e.g. --secret-file .secrets) (default ".secrets")
      --use-gitignore                    Controls whether paths specified in .gitignore should be copied into container (default true)
      --userns string                    user namespace to use
  -v, --verbose                          verbose output
  -w, --watch                            watch the contents of the local repo and run when files change
  -W, --workflows string                 path to workflow file(s) (default "./.github/workflows/")
```

# **Save output in logs**

When you run act all .yml will run in a single terminal making it hard to analyze.
You can create a bash script that can run all the .yml files and save them into a log file.

```
unbuffer act -P ubuntu-18.04=docker.pkg.github.com/frank393/images/cfs_act -W ./.github/workflows/build-cfs.yml |& tee build-cfs.log &

gnome-terminal -- bash -c "unbuffer act -P ubuntu-18.04=docker.pkg.github.com/frank393/images/cfs_act -W ./.github/workflows/build-cfs-deprecated.yml |& tee build-cfs-deprecated.log" &

gnome-terminal -- bash -c "unbuffer act -P ubuntu-18.04=docker.pkg.github.com/frank393/images/cfs_act -W ./.github/workflows/changelog.yml |& tee changelog.log"  &

gnome-terminal -- bash -c "unbuffer act -P ubuntu-18.04=docker.pkg.github.com/frank393/images/cfs_act -W ./.github/workflows/changelog.yml |& tee changelog.log"  &

gnome-terminal -- bash -c "unbuffer act -P ubuntu-18.04=docker.pkg.github.com/frank393/images/cfs_act -W ./.github/workflows/codeql-build.yml |& tee codeql-build.log" &

gnome-terminal -- bash -c "unbuffer act -P ubuntu-18.04=docker.pkg.github.com/frank393/images/cfs_act -W ./.github/workflows/static-analysis.yml |& tee static-analysis.log" &

gnome-terminal -- bash -c "unbuffer act -P ubuntu-18.04=docker.pkg.github.com/frank393/images/cfs_act -W ./.github/workflows/build-documentation.yml |& tee build-documentation.log" 
```
# **Skipping steps**

In the cFS action there are some steps that can't be run locally and in this situation Act adds a special environment variable `ACT` that can be used to skip.
```yml
- name: Some step
  if: ${{ !env.ACT }}
  run: |
    ...
```
