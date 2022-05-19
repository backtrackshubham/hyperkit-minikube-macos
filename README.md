# Docker minikube hyperkit

Steps to run docker on MacOS through hyperkit virtualization in minikube, using vpnkit for seamless networking if you are using a VPN client.  
Content adapted from https://medium.com/rahasak/replace-docker-desktop-with-minikube-and-hyperkit-on-macos-783ce4fb39e3 

## 1. Uninstall Docker Desktop

```sh
sudo rm -Rf /Applications/Docker.app
sudo rm -f /usr/local/bin/docker
sudo rm -f /usr/local/bin/docker-machine
sudo rm -f /usr/local/bin/docker-compose
sudo rm -f /usr/local/bin/docker-credential-desktop
sudo rm -f /usr/local/bin/docker-credential-ecr-login
sudo rm -f /usr/local/bin/docker-credential-osxkeychain
sudo rm -Rf ~/.docker
sudo rm -Rf ~/Library/Containers/com.docker.docker
sudo rm -Rf ~/Library/Application\ Support/Docker\ Desktop
sudo rm -Rf ~/Library/Group\ Containers/group.com.docker
sudo rm -f ~/Library/HTTPStorages/com.docker.docker.binarycookies
sudo rm -f /Library/PrivilegedHelperTools/com.docker.vmnetd
sudo rm -f /Library/LaunchDaemons/com.docker.vmnetd.plist
sudo rm -Rf ~/Library/Logs/Docker\ Desktop
sudo rm -Rf /usr/local/lib/docker
sudo rm -f ~/Library/Preferences/com.docker.docker.plist
sudo rm -Rf ~/Library/Saved\ Application\ State/com.electron.docker-frontend.savedState
sudo rm -f ~/Library/Preferences/com.electron.docker-frontend.plist
```

### Delete previously configured minikube VMs (if applies)

```sh
minikube stop
minikube delete --all --purge
```

## 2. Install docker-cli and minikube

```sh
brew install docker docker-compose minikube
# brew upgrade minikube (if already installed, make sure to have latest version of minikube) 
mkdir -p ~/.docker/cli-plugins
ln -sfn /usr/local/opt/docker-compose/bin/docker-compose ~/.docker/cli-plugins/docker-compose
```

## 3. Install vpnkit and hyperkit

### Binaries (MacOS)

```sh
# extract to somewhere in your $PATH
tar -xvf vpnkit-hyperkit.tgz
cp bin/* /usr/local/bin

# validate install
hyperkit -v
vpnkit --version
```

### Building from source

#### vpnkit

```sh
# install build dependencies
brew install opam gpatch pkg-config dune dylibbundler libtool automake

# build vpnkit
git clone git@github.com:ar2pi/vpnkit.git
cd vpnkit
make -f Makefile.darwin ocaml
make -f Makefile.darwin depends
make -f Makefile.darwin build
mv ~/.opam/4.12.0/bin/vpnkit /usr/local/bin/vpnkit
```

#### hyperkit

```sh
# build hyperkit
git clone git@github.com:moby/hyperkit.git
cd hyperkit
make
mv build/hyperkit /usr/local/bin/hyperkit
```

## 4. Run minikube

#### Start

```sh
vpnkit --ethernet /tmp/vpnkit.eth.sock > /dev/null 2>&1 &
minikube start --driver hyperkit --hyperkit-vpnkit-sock /tmp/vpnkit.eth.sock --memory 8192 --cpus 4 --kubernetes-version "$(kubectl --context dev-eu-west version --short | sed -nE 's/Server Version: (.*)/\1/p')"
eval $(minikube -p minikube docker-env)

# [...] your docker commands
```

#### Stop

```sh
for pid in $(ps -ax | grep "[/]vpnkit" | cut -d " " -f 1); do kill -9 $pid; done
minikube stop
```

#### Configure docker alias (optional)

```sh
cp docker_vm/* /usr/local/bin

echo """
# ensure docker vm is running and env variables are up to date when running docker
alias docker=\"docker_vm\"""" >> ~/.zshrc

source ~/.zshrc
```
