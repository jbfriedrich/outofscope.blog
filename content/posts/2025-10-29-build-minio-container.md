---
title: Build your own MinIO container
date: 2025-10-29T02:10:00
tags: [selfhosting, container, minio, s3]
languages: [en]
---

With very shitty timing, shortly after `CVE RELEASE 2025-10-15T17-29-55Z`, MinIO [decided](https://github.com/minio/minio/issues/21647) that the MinIO community version is "source only" from now on, you have to build the docker container yourself.

![MinIO comment](https://media.jason.re/2025/10/29/minio_comment_communityrel.png)

Fork incoming, I guess...? [Garage](https://garagehq.deuxfleurs.fr) might be a good alternative. But I need some time to check it out. So I guess I will have to build a couple of containers for myself before I can switch. I already have a remote Docker server setup with the proper context, so let's go and get the MinIO build requirements installed on my M2 Mac:

```bash
$ brew install minisign golang
$ minisign -G
$ echo "mySuperSecreptPassphrase" > ~/.minisign/minisign-passphrase
```

This installed `go`, `minisign`, created a key and put the password into the `minisign-passphrase` file, so that the `docker-buildx.sh` build script can later use it to sign the binaries after compilation. `sha256sum` was already available on my Mac, it will be required by the script as well. So let's prepare the source code we need:

```bash
$ git clone https://github.com/minio/minio minio
$ cd minio
$ git remote add upstream git@github.com:minio/minio.git
```

Source repository cloned and "upstream" remote added to the local repo so the build script works properly. Now I just have to slightly edit the `docker-buildx.sh` script and tailor it to my needs. I decided to make the following changes:

- Only build `linux/amd64`, I do not need or want anything else.
- The credential directory for minisign will be `~/.minisign`.
- I cross-compile for `linux/amd64` on my Apple Silicon Mac Mini, so the LDFLAGS need to be generated using my native `darwin/arm64` architecture.
- Remove the `sysctl` commands to disable ipv6.

```bash
#!/bin/bash
#---------------------------------
# file: docker-buildx.sh
#---------------------------------

set -ex

function _init() {
	## All binaries are static make sure to disable CGO.
	export CGO_ENABLED=0
	export CRED_DIR="/Users/jason/.minisign"

	## List of architectures and OS to test coss compilation.
	SUPPORTED_OSARCH="linux/amd64"

	remote=$(git remote get-url upstream)
	if test "$remote" != "git@github.com:minio/minio.git"; then
		echo "Script requires that the 'upstream' remote is set to git@github.com:minio/minio.git"
		exit 1
	fi

	git remote update upstream && git checkout master && git rebase upstream/master

	release=$(git describe --abbrev=0 --tags)
	export release
}

function _build() {
	local osarch=$1
	IFS=/ read -r -a arr <<<"$osarch"
	os="${arr[0]}"
	arch="${arr[1]}"
	package=$(go list -f '{{.ImportPath}}')
	printf -- "--> %15s:%s\n" "${osarch}" "${package}"

	# go build -trimpath to build the binary.
	export GOOS=$os
	export GOARCH=$arch
	export MINIO_RELEASE=RELEASE
	# Generate the LDFLAGS with the native "darwin/arm64" architecure, otherwise the build
    # will fail as you cannot natively run a "linux/amd64" binary on Mac
	LDFLAGS=$(env GOOS= GOARCH= go run buildscripts/gen-ldflags.go)
	go build -tags kqueue -trimpath --ldflags "${LDFLAGS}" -o ./minio-${arch}.${release}
	minisign -qQSm ./minio-${arch}.${release} -s "$CRED_DIR/minisign.key" <"$CRED_DIR/minisign-passphrase"

	sha256sum_str=$(sha256sum <./minio-${arch}.${release})
	rc=$?
	if [ "$rc" -ne 0 ]; then
		abort "unable to generate sha256sum for ${1}"
	fi
	echo "${sha256sum_str// -/minio.${release}}" >./minio-${arch}.${release}.sha256sum
}

function main() {
	echo "Testing builds for OS/Arch: ${SUPPORTED_OSARCH}"
	for each_osarch in ${SUPPORTED_OSARCH}; do
		_build "${each_osarch}"
	done

	docker buildx build --push --no-cache \
		--build-arg RELEASE="${release}" \
		-t "my.very.own.container.hub/minio:latest" \
		-t "my.very.own.container.hub:${release}" \
		--platform=linux/amd64 \
		-f Dockerfile .

	docker buildx prune -f
}

_init && main "$@"
```

Now just build the container:

```bash
$ ./docker-buildx.sh
[...]
$ docker push -a my.very.own.container.hub/minio
```

Now I have my very own up-to-date, latest and greatest MinIO container, with [blackjack and hookers](https://www.youtube.com/watch?v=ubPWaDWcOLU)!
