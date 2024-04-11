
cd rv64gc-emu-software/output
../../rv64gc-emu/build/rv64gc_emu   --bios fw_jump.bin --kernel ../linux/buildroot/output/build/linux-5.15.43/arch/riscv/boot/Image --dtb dtb.dtb --font font.ttf

cd rv64gc-emu-software/linux/buildroot/output/build/linux-5.15.43
sudo make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- all

# Unpack original buildroot rootfs
cd buildroot
cpio -i -d -H newc -F ../rv64gc-emu-software/linux/buildroot/output/images/rootfs.cpio --no-absolute-filenames

# Create debian rootfs
sudo debootstrap --arch=riscv64 unstable debianroot
cd debianroot
cp ../buildroot/init .
find . | cpio -o -H newc > ../debianroot.cpio


riscv_em/build/riscv_em -f linux_for_riscv_em/output/linux-5.10.6/loader_64.bin -d riscv_em/dts/riscv_em.dtb -i buildroot.riscv_em.squashfs 


cd debianroot; mksquashfs . ../debianroot.10.squashfs -Xcompression-level 9


cd linux_for_riscv_em/output/linux-5.10.6
make ARCH=riscv CROSS_COMPILE=riscv64-unknown-elf- -j16 loader
riscv64-unknown-elf-objcopy -O binary arch/riscv/boot/loader loader_64.bin


Config options to set in kernel to get initrd to work
https://unix.stackexchange.com/a/482016


What SATP should be: 8000000003000000
What SATP is:        8000000000080a6a

riscv64-unknown-elf-objdump -t -d vmlinux.o > vmlinux.asm.txt



0000000000000044 <relocate>:
  44:   800005b7                lui     a1,0x80000
  48:   00000617                auipc   a2,0x0
  4c:   00060613                mv      a2,a2
  50:   40c585b3                sub     a1,a1,a2
  54:   00b080b3                add     ra,ra,a1
  58:   00000617                auipc   a2,0x0
  5c:   00060613                mv      a2,a2
  60:   00b60633                add     a2,a2,a1
  64:   10561073                csrw    stvec,a2
  68:   00c55613                srl     a2,a0,0xc
  6c:   fff0059b                addw    a1,zero,-1
  70:   03f59593                sll     a1,a1,0x3f
  74:   00b66633                or      a2,a2,a1
  78:   00000517                auipc   a0,0x0
  7c:   00050513                mv      a0,a0
  80:   00c55513                srl     a0,a0,0xc
  84:   00b56533                or      a0,a0,a1
  88:   12000073                sfence.vma
  8c:   18051073                csrw    satp,a0


a0 = x10
a1 = x11



"%x" % (0x0000000080000074 + 2666 * 2**12 - 116)
'80a6a000'


CONFIG_PAGE_OFFSET=0xffffffff80000000
idx = (((CONFIG_PAGE_OFFSET) >> PGDIR_SHIFT) & (PTRS_PER_PGD - 1))


pgd_index(a)  (((a) >> PGDIR_SHIFT) & (PTRS_PER_PGD - 1))
