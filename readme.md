## Piston
Piston is the underlying engine for running untrusted and possibly malicious
code that originates from EMKC contests and challenges. It's also used in the
Engineer Man Discord server via [felix bot](https://github.com/engineer-man/felix).

#### Installation
```
# clone and enter repo
git clone https://github.com/engineer-man/piston
cd piston/lxc

# install dependencies

# centos:
yum install -y epel-release
yum install -y lxc lxc-templates debootstrap libvirt
systemctl start libvirtd

# ubuntu server 18.04:
apt install lxc lxc-templates debootstrap libvirt0

# everything else:
# not documented, please open pull requests with commands for debian/arch/macos

# create and start container
lxc-create -t download -n piston -- --dist ubuntu --release bionic --arch amd64
./start

# open a shell to the container
./shell

# install all necessary piston dependencies
echo 'source /opt/.profile' >> /opt/.bashrc
echo 'export HOME=/opt' >> /opt/.profile
echo 'export TERM=linux' >> /opt/.profile
export HOME=/opt
export TERM=linux
sed -i 's/\/root/\/opt/' /etc/passwd
sed -i \
    's/http:\/\/archive.ubuntu.com\/ubuntu/http:\/\/mirror.math.princeton.edu\/pub\/ubuntu/' \
    /etc/apt/sources.list
apt-get update
apt-get install -y \
    nano wget build-essential pkg-config libxml2-dev \
    libsqlite3-dev mono-complete curl cmake libpython2.7-dev \
    ruby

# install python2
# final binary: /opt/python2/Python-2.7.17/python -V
cd /opt && mkdir python2 && cd python2
wget https://www.python.org/ftp/python/2.7.17/Python-2.7.17.tar.xz
unxz Python-2.7.17.tar.xz
tar -xf Python-2.7.17.tar
cd Python-2.7.17
./configure
# open Modules/Setup and uncomment zlib line
make
echo 'export PATH=$PATH:/opt/python2/Python-2.7.17' >> /opt/.profile
source /opt/.profile

# install python3
# final binary: /opt/python3/Python-3.8.2/python -V
cd /opt && mkdir python3 && cd python3
wget https://www.python.org/ftp/python/3.8.2/Python-3.8.2.tar.xz
unxz Python-3.8.2.tar.xz
tar -xf Python-3.8.2.tar
cd Python-3.8.2
./configure
make
ln -s python python3.8
echo 'export PATH=$PATH:/opt/python3/Python-3.8.2' >> /opt/.profile
source /opt/.profile

# install node.js
# final binary: /opt/nodejs/node-v12.16.1-linux-x64/bin/node -v
cd /opt && mkdir nodejs && cd nodejs
wget https://nodejs.org/dist/v12.16.1/node-v12.16.1-linux-x64.tar.xz
unxz node-v12.16.1-linux-x64.tar.xz
tar -xf node-v12.16.1-linux-x64.tar
echo 'export PATH=$PATH:/opt/nodejs/node-v12.16.1-linux-x64/bin' >> /opt/.profile
source /opt/.profile

# install typescript
# final binary: /opt/nodejs/node-v12.16.1-linux-x64/bin/tsc -v
/opt/nodejs/node-v12.16.1-linux-x64/bin/npm i -g typescript

# install golang
# final binary: /opt/go/go/bin/go version
cd /opt && mkdir go && cd go
wget https://dl.google.com/go/go1.14.1.linux-amd64.tar.gz
tar -xzf go1.14.1.linux-amd64.tar.gz
echo 'export PATH=$PATH:/opt/go/go/bin' >> /opt/.profile
echo 'export GOROOT=/opt/go/go' >> /opt/.profile
echo 'export GOCACHE=/tmp' >> /opt/.profile
source /opt/.profile

# install php
# final binary: /usr/local/bin/php -v
cd /opt && mkdir php && cd php
wget https://www.php.net/distributions/php-7.4.4.tar.gz
tar -xzf php-7.4.4.tar.gz
cd php-7.4.4
./configure
make
make install

# install rust
# final binary: /opt/.cargo/bin/rustc --version
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
echo 'export PATH=$PATH:/opt/.cargo/bin' >> /opt/.profile
source /opt/.profile

# install swift
# final binary: /opt/swift/swift-5.1.5-RELEASE-ubuntu18.04/usr/bin/swift --version
cd /opt && mkdir swift && cd swift
wget https://swift.org/builds/swift-5.1.5-release/ubuntu1804/swift-5.1.5-RELEASE/swift-5.1.5-RELEASE-ubuntu18.04.tar.gz
tar -xzf swift-5.1.5-RELEASE-ubuntu18.04.tar.gz
echo 'export PATH=$PATH:/opt/swift/swift-5.1.5-RELEASE-ubuntu18.04/usr/bin' >> /opt/.profile
source /opt/.profile

# install nasm
# final binary: /opt/nasm/nasm-2.14.02/nasm -v
cd /opt && mkdir nasm && cd nasm
wget https://www.nasm.us/pub/nasm/releasebuilds/2.14.02/nasm-2.14.02.tar.gz
tar -xzf nasm-2.14.02.tar.gz
cd nasm-2.14.02
./configure
make
echo 'export PATH=$PATH:/opt/nasm/nasm-2.14.02' >> /opt/.profile
source /opt/.profile

# install java
# final binary: /opt/java/jdk-14/bin/java -version
cd /opt && mkdir java && cd java
wget https://download.java.net/java/GA/jdk14/076bab302c7b4508975440c56f6cc26a/36/GPL/openjdk-14_linux-x64_bin.tar.gz
tar -xzf openjdk-14_linux-x64_bin.tar.gz
echo 'export PATH=$PATH:/opt/java/jdk-14/bin' >> /opt/.profile
source /opt/.profile

# install julia
#final binary: /opt/julia/julia-1.4.1/bin/julia --version
cd /opt && mkdir julia && cd julia
wget https://julialang-s3.julialang.org/bin/linux/x64/1.4/julia-1.4.1-linux-x86_64.tar.gz
tar -xzf julia-1.4.1-linux-x86_64.tar.gz
echo 'export PATH=$PATH:/opt/julia/julia-1.4.1/bin' >> /opt/.profile
source /opt/.profile


# create runnable users and apply limits
for i in {1..150}; do
    useradd -M runner$i
    usermod -d /tmp runner$i
    echo "runner$i soft nproc 64" >> /etc/security/limits.conf
    echo "runner$i hard nproc 64" >> /etc/security/limits.conf
    echo "runner$i soft nofile 2048" >> /etc/security/limits.conf
    echo "runner$i hard nofile 2048" >> /etc/security/limits.conf
done

# cleanup
rm -rf /home/ubuntu
chmod 777 /tmp

# leave container
exit

# optionally run tests
cd ../tests
./test_all_lxc
```

#### CLI Usage
- `lxc/execute [language] [file path] [arg]...`

#### API Usage
To use the API, it must first be started. To start the API, run the following:
```
cd api
./start
```
The Piston API exposes one endpoint at `http://127.0.0.1:2000/execute`.
This endpoint takes the following JSON payload and expects at least the language and source. If
source is not provided, a blank file is passed as the source.
```json
{
    "language": "js",
    "source": "console.log(process.argv)",
    "args": [
        "1",
        "2",
        "3"
    ]
}
```
A typical response when everything succeeds will be similar to the following:
```json
{
    "ran": true,
    "language": "js",
    "version": "12.13.0",
    "output": "[ '/usr/bin/node',\n  '/tmp/code.code',\n  '1',\n  '2',\n  '3' ]"
}
```
If an invalid language is supplied, a typical response will look like the following:
```json
{
    "code": "unsupported_language",
    "message": "whatever is not supported by Piston"
}
```

#### Supported Languages
Currently python2, python3, c, c++, go, node, ruby, r, c#, nasm, php, java,
swift, brainfuck, rust, bash, awk, and typescript is supported.

#### Principle of Operation
Piston utilizes LXC as the primary mechanism for sandboxing. There is a small API written in Go which takes
in execution requests and executes them in the container. High level, the API writes
a temporary source and args file to `/tmp` and that gets mounted read-only along with the execution scripts into the container.
The source file is either ran or compiled and ran (in the case of languages like c, c++, c#, go, etc.).

#### Security
LXC provides a great deal of security out of the box in that it's separate from the system.
Piston takes additional steps to make it resistant to
various privilege escalation, denial-of-service, and resource saturation threats. These steps include:
- Disabling outgoing network interaction
- Capping max processes at 64 (resists `:(){ :|: &}:;`, `while True: os.fork()`, etc.)
- Capping max files at 2048 (resists various file based attacks)
- Mounting all resources read-only (resists `sudo rm -rf --no-preserve-root /`)
- Running as a variety of unprivileged users
- Capping runtime execution at 3 seconds
- Capping stdout to 65536 characters (resists yes/no bombs and runaway output)
- SIGKILLing misbehaving code

#### License
Piston is licensed under the MIT license.
