#
# NOTE: THIS DOCKERFILE IS GENERATED VIA "apply-templates.sh"
#
# PLEASE DO NOT EDIT IT DIRECTLY.
#

FROM buildpack-deps:bullseye-scm AS build
# [build 1/2] FROM docker.io/library/buildpack-deps:bullseye-scm@sha256:e4703b57b3db24d7b2bb43c9512b900bcb631f5af86737dc55eaa7bdf4748be9

ENV PATH /usr/local/go/bin:$PATH

ENV GOLANG_VERSION 1.22.4

RUN set -eux; \
	now="$(date '+%s')"; \
	arch="$(dpkg --print-architecture)"; arch="${arch##*-}"; \
	url=; \
	case "$arch" in \
	'amd64') \
	url='https://dl.google.com/go/go1.22.4.linux-amd64.tar.gz'; \
	sha256='ba79d4526102575196273416239cca418a651e049c2b099f3159db85e7bade7d'; \
	;; \
	'armhf') \
	url='https://dl.google.com/go/go1.22.4.linux-armv6l.tar.gz'; \
	sha256='e2b143fbacbc9cbd448e9ef41ac3981f0488ce849af1cf37e2341d09670661de'; \
	;; \
	'arm64') \
	url='https://dl.google.com/go/go1.22.4.linux-arm64.tar.gz'; \
	sha256='a8e177c354d2e4a1b61020aca3562e27ea3e8f8247eca3170e3fa1e0c2f9e771'; \
	;; \
	'i386') \
	url='https://dl.google.com/go/go1.22.4.linux-386.tar.gz'; \
	sha256='47a2a8d249a91eb8605c33bceec63aedda0441a43eac47b4721e3975ff916cec'; \
	;; \
	'mips64el') \
	url='https://dl.google.com/go/go1.22.4.linux-mips64le.tar.gz'; \
	sha256='7486e2d7dd8c98eb44df815ace35a7fe7f30b7c02326e3741bd934077508139b'; \
	;; \
	'ppc64el') \
	url='https://dl.google.com/go/go1.22.4.linux-ppc64le.tar.gz'; \
	sha256='a3e5834657ef92523f570f798fed42f1f87bc18222a16815ec76b84169649ec4'; \
	;; \
	'riscv64') \
	url='https://dl.google.com/go/go1.22.4.linux-riscv64.tar.gz'; \
	sha256='56a827ff7dc6245bcd7a1e9288dffaa1d8b0fd7468562264c1523daf3b4f1b4a'; \
	;; \
	's390x') \
	url='https://dl.google.com/go/go1.22.4.linux-s390x.tar.gz'; \
	sha256='7590c3e278e2dc6040aae0a39da3ca1eb2e3921673a7304cc34d588c45889eec'; \
	;; \
	*) echo >&2 "error: unsupported architecture '$arch' (likely packaging update needed)"; exit 1 ;; \
	esac; \
	\
	wget -O go.tgz.asc "$url.asc"; \
	wget -O go.tgz "$url" --progress=dot:giga; \
	echo "$sha256 *go.tgz" | sha256sum -c -; \
	\
	# https://github.com/golang/go/issues/14739#issuecomment-324767697
	GNUPGHOME="$(mktemp -d)"; export GNUPGHOME; \
	# https://www.google.com/linuxrepositories/
	gpg --batch --keyserver keyserver.ubuntu.com --recv-keys 'EB4C 1BFD 4F04 2F6D DDCC  EC91 7721 F63B D38B 4796'; \
	# let's also fetch the specific subkey of that key explicitly that we expect "go.tgz.asc" to be signed by, just to make sure we definitely have it
	gpg --batch --keyserver keyserver.ubuntu.com --recv-keys '2F52 8D36 D67B 69ED F998  D857 78BD 6547 3CB3 BD13'; \
	gpg --batch --verify go.tgz.asc go.tgz; \
	gpgconf --kill all; \
	rm -rf "$GNUPGHOME" go.tgz.asc; \
	\
	tar -C /usr/local -xzf go.tgz; \
	rm go.tgz; \
	\
	# save the timestamp from the tarball so we can restore it for reproducibility, if necessary (see below)
	SOURCE_DATE_EPOCH="$(stat -c '%Y' /usr/local/go)"; \
	export SOURCE_DATE_EPOCH; \
	touchy="$(date -d "@$SOURCE_DATE_EPOCH" '+%Y%m%d%H%M.%S')"; \
	# for logging validation/edification
	date --date "@$SOURCE_DATE_EPOCH" --rfc-2822; \
	# sanity check (detected value should be older than our wall clock)
	[ "$SOURCE_DATE_EPOCH" -lt "$now" ]; \
	\
	if [ "$arch" = 'armhf' ]; then \
	[ -s /usr/local/go/go.env ]; \
	before="$(go env GOARM)"; [ "$before" != '7' ]; \
	{ \
	echo; \
	echo '# https://github.com/docker-library/golang/issues/494'; \
	echo 'GOARM=7'; \
	} >> /usr/local/go/go.env; \
	after="$(go env GOARM)"; [ "$after" = '7' ]; \
	# (re-)clamp timestamp for reproducibility (allows "COPY --link" to be more clever/useful)
	touch -t "$touchy" /usr/local/go/go.env /usr/local/go; \
	fi; \
	\
	# ideally at this point, we would just "COPY --link ... /usr/local/go/ /usr/local/go/" but BuildKit insists on creating the parent directories (perhaps related to https://github.com/opencontainers/image-spec/pull/970), and does so with unreproducible timestamps, so we instead create a whole new "directory tree" that we can "COPY --link" to accomplish what we want
	mkdir /target /target/usr /target/usr/local; \
	mv -vT /usr/local/go /target/usr/local/go; \
	ln -svfT /target/usr/local/go /usr/local/go; \
	touch -t "$touchy" /target/usr/local /target/usr /target; \
	\
	# smoke test
	go version; \
	# make sure our reproducibile timestamp is probably still correct (best-effort inline reproducibility test)
	epoch="$(stat -c '%Y' /target/usr/local/go)"; \
	[ "$SOURCE_DATE_EPOCH" = "$epoch" ]; \
	find /target -newer /target/usr/local/go -exec sh -c 'ls -ld "$@" && exit "$#"' -- '{}' +

# [build 2/2] RUN
# set -eux;
# now="$(date '+%s')";
# arch="$(dpkg --print-architecture)";
# arch="${arch##*-}";
# url=;
# case "$arch" in
# 	'amd64')
# 		url='https://dl.google.com/go/go1.22.4.linux-amd64.tar.gz';
# 		sha256='ba79d4526102575196273416239cca418a651e049c2b099f3159db85e7bade7d';
# 		;;
# 	'armhf')
# 		url='https://dl.google.com/go/go1.22.4.linux-armv6l.tar.gz';
# 		sha256='e2b143fbacbc9cbd448e9ef41ac3981f0488ce849af1cf37e2341d09670661de';
# 		;;
# 	'arm64')
# 		url='https://dl.google.com/go/go1.22.4.linux-arm64.tar.gz';
# 		sha256='a8e177c354d2e4a1b61020aca3562e27ea3e8f8247eca3170e3fa1e0c2f9e771';
# 		;;
# 	'i386')
# 		url='https://dl.google.com/go/go1.22.4.linux-386.tar.gz';
# 		sha256='47a2a8d249a91eb8605c33bceec63aedda0441a43eac47b4721e3975ff916cec';
# 		;;
# 	'mips64el')
# 		url='https://dl.google.com/go/go1.22.4.linux-mips64le.tar.gz';
# 		sha256='7486e2d7dd8c98eb44df815ace35a7fe7f30b7c02326e3741bd934077508139b';
# 		;;
# 	'ppc64el')
# 		url='https://dl.google.com/go/go1.22.4.linux-ppc64le.tar.gz';
# 		sha256='a3e5834657ef92523f570f798fed42f1f87bc18222a16815ec76b84169649ec4';
# 		;;
# 	'riscv64')
# 		url='https://dl.google.com/go/go1.22.4.linux-riscv64.tar.gz';
# 		sha256='56a827ff7dc6245bcd7a1e9288dffaa1d8b0fd7468562264c1523daf3b4f1b4a';
# 		;;
# 	's390x')
# 		url='https://dl.google.com/go/go1.22.4.linux-s390x.tar.gz';
# 		sha256='7590c3e278e2dc6040aae0a39da3ca1eb2e3921673a7304cc34d588c45889eec';
# 		;;
# 	*)
# 		echo >&2 "error: unsupported architecture '$arch' (likely packaging update needed)";
# 		exit 1
# 		;;
# esac;
# wget -O go.tgz.asc "$url.asc";
# wget -O go.tgz "$url" --progress=dot:giga;
# echo "$sha256 *go.tgz" | sha256sum -c -;
# GNUPGHOME="$(mktemp -d)";
# export GNUPGHOME;
# gpg --batch --keyserver keyserver.ubuntu.com --recv-keys 'EB4C 1BFD 4F04 2F6D DDCC  EC91 7721 F63B D38B 4796';
# gpg --batch --keyserver keyserver.ubuntu.com --recv-keys '2F52 8D36 D67B 69ED F998  D857 78BD 6547 3CB3 BD13';
# gpg --batch --verify go.tgz.asc go.tgz;
# gpgconf --kill all;
# rm -rf "$GNUPGHOME" go.tgz.asc;
# tar -C /usr/local -xzf go.tgz;
# rm go.tgz;
# SOURCE_DATE_EPOCH="$(stat -c '%Y' /usr/local/go)";
# export SOURCE_DATE_EPOCH;
# touchy="$(date -d "@$SOURCE_DATE_EPOCH" '+%Y%m%d%H%M.%S')";
# date --date "@$SOURCE_DATE_EPOCH" --rfc-2822;
# [ "$SOURCE_DATE_EPOCH" -lt "$now" ];
# if [ "$arch" = 'armhf' ]; then
# 	[ -s /usr/local/go/go.env ];
# 	before="$(go env GOARM)";
# 	[ "$before" != '7' ];
# 	{
# 		echo;
# 		echo '# https://github.com/docker-library/golang/issues/494';
# 		echo 'GOARM=7';
# 	} >> /usr/local/go/go.env;
# 	after="$(go env GOARM)";
# 	[ "$after" = '7' ];
# 	touch -t "$touchy" /usr/local/go/go.env /usr/local/go;
# fi;
# mkdir /target /target/usr /target/usr/local;
# mv -vT /usr/local/go /target/usr/local/go;
# ln -svfT /target/usr/local/go /usr/local/go;
# touch -t "$touchy" /target/usr/local /target/usr /target;
# go version;
# epoch="$(stat -c '%Y' /target/usr/local/go)";
# [ "$SOURCE_DATE_EPOCH" = "$epoch" ];
# find /target -newer /target/usr/local/go -exec sh -c 'ls -ld "$@" && exit "$#"' -- '{}' +

# + date +%s
# + now=1718789438
# + dpkg --print-architecture
# + arch=amd64
# + arch=amd64
# + url=
# + url=https://dl.google.com/go/go1.22.4.linux-amd64.tar.gz
# + sha256=ba79d4526102575196273416239cca418a651e049c2b099f3159db85e7bade7d
# + wget -O go.tgz.asc https://dl.google.com/go/go1.22.4.linux-amd64.tar.gz.asc
# + wget -O go.tgz https://dl.google.com/go/go1.22.4.linux-amd64.tar.gz --progress=dot:giga
# + echo ba79d4526102575196273416239cca418a651e049c2b099f3159db85e7bade7d *go.tgz | sha256sum -c -
# + mktemp -d
# + GNUPGHOME=/tmp/tmp.hMsS8Hy9IC
# + export GNUPGHOME
# + gpg --batch --keyserver keyserver.ubuntu.com --recv-keys EB4C 1BFD 4F04 2F6D DDCC  EC91 7721 F63B D38B 4796
# + gpg --batch --keyserver keyserver.ubuntu.com --recv-keys 2F52 8D36 D67B 69ED F998  D857 78BD 6547 3CB3 BD13
# + gpg --batch --verify go.tgz.asc go.tgz
# + gpgconf --kill all
# + rm -rf /tmp/tmp.hMsS8Hy9IC go.tgz.asc
# + tar -C /usr/local -xzf go.tgz
# + rm go.tgz
# + stat -c %Y /usr/local/go
# + SOURCE_DATE_EPOCH=1717097167
# + export SOURCE_DATE_EPOCH
# + date -d @1717097167 +%Y%m%d%H%M.%S
# + touchy=202405301926.07
# + date --date @1717097167 --rfc-2822
# + [ 1717097167 -lt 1718789438 ]
# + [ amd64 = armhf ]
# + mkdir /target /target/usr /target/usr/local
# + mv -vT /usr/local/go /target/usr/local/go
# + ln -svfT /target/usr/local/go /usr/local/go
# + touch -t 202405301926.07 /target/usr/local /target/usr /target
# + go version
# + stat -c %Y /target/usr/local/go
# + epoch=1717097167
# + [ 1717097167 = 1717097167 ]
# + find /target -newer /target/usr/local/go -exec sh -c ls -ld "$@" && exit "$#" -- {} +

RUN env

RUN go env

FROM buildpack-deps:bullseye-scm

RUN env

# install cgo-related dependencies
RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
	g++ \
	gcc \
	libc6-dev \
	make \
	pkg-config \
	tree \
	; \
	rm -rf /var/lib/apt/lists/*
# [stage-1 2/5] RUN
# set -eux;
# apt-get update;
# apt-get install -y --no-install-recommends g++ gcc libc6-dev make pkg-config;
# rm -rf /var/lib/apt/lists/*

ENV GOLANG_VERSION 1.22.4

# don't auto-upgrade the gotoolchain
# https://github.com/docker-library/golang/issues/472
ENV GOTOOLCHAIN=local

ENV GOPATH /go
ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH

RUN env

RUN tree -L 1 /

# (see notes above about "COPY --link")
COPY --from=build --link /target/ /
# [stage-1 3/5] COPY --from=build --link /target/ /

RUN tree -L 1 /

RUN tree -L 1 /target

RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 1777 "$GOPATH"
# [stage-1 4/5] RUN mkdir -p "/go/src" "/go/bin" && chmod -R 1777 "/go"

RUN tree -L 1 /go

WORKDIR $GOPATH
# [stage-1 5/5] WORKDIR /go
