+++
title = 'Hugo Setup'
date = 2024-04-18T07:07:07+01:00
draft = false
+++
# Install Go

Download Go.

```shell
wget https://go.dev/dl/go1.22.3.linux-amd64.tar.gz
```

Decompress the file.

```shell
sudo tar -C /usr/local -xzf go1.22.3.linux-amd64.tar.gz
```

Add the following content to **~/.bashrc**

```shell
export PATH=$PATH:/usr/local/go/bin
```

Restart the shell, and verify if the installation is successful.

```shell
go version
```

# Install Dart Sass

Download the Dart Sass.

```shell
wget https://github.com/sass/dart-sass/releases/download/1.77.1/dart-sass-1.77.1-linux-x64.tar.gz
```

Decompress the file.

```shell
tar -xzf dart-sass-1.77.1-linux-x64.tar.gz
```

Add excutable file to PATH.

```shell
export PATH=$PATH:/usr/local/dart-sass
```

Verify the installation.

```shell
sass
```

# Install Hugo

Download Hugo.

```shell
wget https://github.com/gohugoio/hugo/releases/download/v0.126.0/hugo_0.126.0_linux-amd64.tar.gz
```

Add excutable file to PATH.

```shell
export PATH=$PATH:/usr/local/hugo
```

Verify the installation.

```shell
hugo
```

Run Hugo local server.

```shell
hugo server --bind 0.0.0.0 --buildDrafts
```

publish the site

```shell
hugo
```