# tianon/qemu

	touch $HOME/hda.qcow2
	cd downloads && wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-10.4.0-amd64-xfce-CD-1.iso 
	ln -s debian-10.4.0-amd64-xfce-CD-1.iso debian.iso 
	docker run -it --rm \
		--device /dev/kvm \
		--name qemu-container \
		-v $HOME/hda.qcow2:/tmp/hda.qcow2 \
		-e QEMU_HDA=/tmp/hda.qcow2 \
		-e QEMU_HDA_SIZE=100G \
		-e QEMU_CPU=4 \
		-e QEMU_RAM=4096 \
		-v $HOME/downloads/debian.iso:/tmp/debian.iso:ro \
		-e QEMU_CDROM=/tmp/debian.iso \
		-e QEMU_BOOT='order=d' \
		-e QEMU_PORTS='2375 2376' \
		tianon/qemu

Note: port 22 will always be mapped (regardless of the contents of `QEMU_PORTS`).

For supplying additional arguments, use a command of `start-qemu <args>`. For example, to use `-curses`, one would `docker run ... tianon/qemu start-qemu -curses`.

For UEFI support, [the `ovmf` package](https://packages.debian.org/sid/ovmf) is installed, which can be utilized most easily by supplying `--bios /usr/share/ovmf/OVMF.fd`.
