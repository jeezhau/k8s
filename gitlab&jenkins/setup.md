# gitlab
[ubuntu](https://about.gitlab.com/install/#ubuntu)
## 1. Install and configure the necessary dependencies
```sh
sudo apt-get update
sudo apt-get install -y curl openssh-server ca-certificates
```
## 2. Add the GitLab package repository and install the package
Add the GitLab package repository.
```sh
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
```