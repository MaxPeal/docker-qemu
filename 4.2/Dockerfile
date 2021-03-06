FROM debian:buster-slim

RUN apt-get update && apt-get install -y --no-install-recommends \
		ovmf \
	&& rm -rf /var/lib/apt/lists/*

COPY *.patch /qemu-patches/

# https://wiki.qemu.org/SecurityProcess
ENV QEMU_KEYS \
# Michael Roth
		CEACC9E15534EBABB82D3FA03353C9CEF108B584
# https://wiki.qemu.org/Planning/ReleaseProcess#Sign_the_resulting_tarball_with_GPG: (they get signed by whoever is making the release)

# https://www.qemu.org/download/#source
# https://download.qemu.org/?C=M;O=D
ENV QEMU_VERSION 4.2.0
ENV QEMU_URL https://download.qemu.org/qemu-4.2.0.tar.xz

RUN set -eux; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	\
	apt-get update; \
	apt-get install -y --no-install-recommends \
		ca-certificates \
		gnupg dirmngr \
		wget \
		xz-utils \
		\
		patch \
		\
		bzip2 \
		gcc \
		gnutls-dev \
		libaio-dev \
		libbz2-dev \
		libc-dev \
		libcap-dev \
		libcap-ng-dev \
		libcurl4-gnutls-dev \
		libglib2.0-dev \
		libiscsi-dev \
		libjpeg-dev \
		libncursesw5-dev \
		libnfs-dev \
		libnuma-dev \
		libpixman-1-dev \
		libpng-dev \
		libseccomp-dev \
		libssh-dev \
		libusb-1.0-0-dev \
		libusbredirparser-dev \
		libxen-dev \
		make \
		pkg-config \
		python \
		xfslibs-dev \
		zlib1g-dev \
	; \
	rm -rf /var/lib/apt/lists/*; \
	\
# patch kernel headers ala https://github.com/torvalds/linux/commit/467d12f5c7842896d2de3ced74e4147ee29e97c8
# filed in debian at https://bugs.debian.org/960271
	wget -O bits-per-long.patch 'https://github.com/torvalds/linux/commit/467d12f5c7842896d2de3ced74e4147ee29e97c8.patch'; \
	#DISABEL###patch --input="$PWD/bits-per-long.patch" --strip=4 --directory=/usr/include/linux; \
	rm bits-per-long.patch; \
	\
	tarball="$(basename "$QEMU_URL")"; \
	wget -O "$tarball.sig" "$QEMU_URL.sig"; \
	wget -O "$tarball" "$QEMU_URL" --progress=dot:giga; \
	\
	export GNUPGHOME="$(mktemp -d)"; \
	for key in $QEMU_KEYS; do \
		gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
	done; \
	gpg --batch --verify "$tarball.sig" "$tarball"; \
	gpgconf --kill all; \
	rm -rf "$GNUPGHOME"; \
	\
	mkdir /usr/src/qemu; \
	tar -xf "$tarball" -C /usr/src/qemu --strip-components=1; \
	rm "$tarball" "$tarball.sig"; \
	\
	cd /usr/src/qemu; \
	\
	for p in /qemu-patches/*.patch; do \
		patch --strip 1 --input "$p"; \
	done; \
	rm -rf /qemu-patches; \
	\
	./configure --help; \
	./configure \
		--disable-docs \
		--disable-gtk --disable-vte \
		--disable-sdl \
		--enable-numa \
		--enable-attr \
		--enable-bzip2 \
		--enable-cap-ng \
		--enable-curl \
		--enable-curses \
		--enable-fdt \
		--enable-gnutls \
		--enable-kvm \
		--enable-libiscsi \
		--enable-libnfs \
		--enable-libssh \
		--enable-libusb \
		--enable-linux-aio \
		--enable-linux-user \
		--enable-modules \
		--enable-seccomp \
		--enable-system \
		--enable-tools \
		--enable-usb-redir \
		--enable-vhost-net \
		--enable-vhost-user \
		--enable-vhost-vsock \
		--enable-virtfs \
		--enable-vnc \
		--enable-vnc-jpeg \
		--enable-vnc-png \
		--enable-xen \
		--enable-xfsctl \
#		--enable-rbd \
#		--enable-vde \
	; \
	make -j "$(nproc)"; \
	make install; \
	\
	cd /; \
	rm -rf /usr/src/qemu; \
	\
	apt-mark auto '.*' > /dev/null; \
	[ -z "$savedAptMark" ] || apt-mark manual $savedAptMark > /dev/null; \
	find /usr/local -type f -executable -exec ldd '{}' ';' \
		| awk '/=>/ { print $(NF-1) }' \
		| sort -u \
		| xargs -r dpkg-query --search \
		| cut -d: -f1 \
		| sort -u \
		| xargs -r apt-mark manual \
	; \
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	\
# basic smoke test
	qemu-img --version

RUN set -eux; \
        savedAptMark="$(apt-mark showmanual)"; \
        apt-get update; apt-get install -y --no-install-recommends \
                curl wget tar xz-utils zstd ca-certificates dnsutils iproute2 net-tools; \
        rm -rf /var/lib/apt/lists/*; \
        HFILE=/usr/local/share/qemu/q40pofw-serial.rom HASHcmd=sha256sum HASHSUM=4a98bca4ab2149e2fe9e7cffb2215c04e2451f30d2e1cff173086908cb94fd17 HURL=https://github.com/artyom-tarasenko/openfirmware/releases/download/40p-20190413/q40pofw-serial.rom ;\
        printf "HFILE=$HFILE HASHcmd=$HASHcmd HASHSUM=$HASHSUM HURL=$HURL" ;\
        curl -o $HFILE -LR -C- -f -S --connect-timeout 15 --max-time 600 --retry 3 $HURL ; (printf %b CHECKSUM\\072\\040expect\\040this\\040$HASHcmd\\072\\040$HASHSUM\\040\\052$HFILE\\012 ; printf %b $HASHSUM\\040\\052$HFILE\\012 | $HASHcmd -c - ;) || (printf %b ERROR\\072\\040CHECKSUMFAILD\\072\\040the\\040file\\040has\\040this\\040$HASHcmd\\072\\040 ; $HASHcmd -b $HFILE ; exit 1) ;\
\
        HFILE=/usr/local/share/qemu/q40pofw-vga.rom HASHcmd=sha256sum HASHSUM=84b55473bb4445b201b545dc5938fce5be96080182a92ea16c7ce131705bc204 HURL=https://github.com/artyom-tarasenko/openfirmware/releases/download/40p-20190413/q40pofw-vga.rom ;\
        printf "HFILE=$HFILE HASHcmd=$HASHcmd HASHSUM=$HASHSUM HURL=$HURL" ;\
        curl -o $HFILE -LR -C- -f -S --connect-timeout 15 --max-time 600 --retry 3 $HURL ; (printf %b CHECKSUM\\072\\040expect\\040this\\040$HASHcmd\\072\\040$HASHSUM\\040\\052$HFILE\\012 ; printf %b $HASHSUM\\040\\052$HFILE\\012 | $HASHcmd -c - ;) || (printf %b ERROR\\072\\040CHECKSUMFAILD\\072\\040the\\040file\\040has\\040this\\040$HASHcmd\\072\\040 ; $HASHcmd -b $HFILE ; exit 1)

RUN wget https://github.com/MaxPeal/docker-qemu/raw/master/openbios-ppc-rtas --directory-prefix=/usr/local/share/qemu/ 
RUN wget https://github.com/MaxPeal/docker-qemu/releases/download/40p-test2/qemu-system-ppc-qemu-40p-20190406-aix-boots.tar.xz ; \
	tar xvJf qemu-system-ppc-qemu-40p-20190406-aix-boots.tar.xz ; \
	chmod 755 *qemu-40p-20190406-aix-boots ; mv *qemu-40p-20190406-aix-boots /usr/local/bin/ ; rm q40*.rom

STOPSIGNAL SIGHUP

EXPOSE 22
EXPOSE 5900

COPY start-qemu /usr/local/bin/
CMD ["start-qemu"]
