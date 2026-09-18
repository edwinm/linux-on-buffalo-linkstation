# Installing Debian Linux on a Buffalo LinkStation Live (HS-DHGL)

**A complete, beginner-friendly guide to giving a 2007 NAS a second life as a modern backup server.**

You do not need to be a Linux expert to follow this. Every command is explained, every
piece of jargon is defined the first time it appears, and every chapter ends with a check so
you know whether it worked before moving on.

---

## Table of contents

| Chapter | What you achieve |
|---|---|
| [0. Is this project for you?](#chapter-0--is-this-project-for-you) | Decide honestly whether to start |
| [1. Understanding the machine](#chapter-1--understanding-the-machine) | Know what you are working with |
| [2. Shopping list and preparation](#chapter-2--shopping-list-and-preparation) | Have everything on the desk |
| [3. Rescue the data on your old disk](#chapter-3--rescue-the-data-on-your-old-disk-optional) | Nothing is lost *(optional)* |
| [4. Build a Linux workstation](#chapter-4--build-a-linux-workstation) | Your Mac or PC can run Linux commands |
| [5. Connect a serial console](#chapter-5--connect-a-serial-console-strongly-recommended) | You can see what the NAS is doing |
| [6. Fit the new disk](#chapter-6--fit-the-new-disk) | Hardware is ready |
| [7. Partition the disk](#chapter-7--partition-the-disk) | The disk is divided correctly |
| [8. Create the filesystems](#chapter-8--create-the-filesystems) | The disk can store files |
| [9. Install the Debian base system](#chapter-9--install-the-debian-base-system) | Debian is on the disk |
| [10. Configure the system](#chapter-10--configure-the-system) | It knows its name, network and disks |
| [11. Install the kernel](#chapter-11--install-the-kernel) | The NAS can actually boot it |
| [12. First boot](#chapter-12--first-boot) | Debian is running on the NAS |
| [13. Turn it into a NAS](#chapter-13--turn-it-into-a-nas) | You can store files on it |
| [14. Keeping it healthy](#chapter-14--keeping-it-healthy) | It stays working and safe |
| [Appendix A: Glossary](#appendix-a--glossary) | Every term in one place |
| [Appendix B: Troubleshooting](#appendix-b--troubleshooting) | When something goes wrong |
| [Appendix C: Sources](#appendix-c--sources-and-further-reading) | Where this knowledge comes from |

---

## Chapter 0 — Is this project for you?

### What this guide does

It replaces the original Buffalo software on a **LinkStation Live HS-DHGL** with **Debian
Linux**, and fits a larger hard disk (up to 2 TB). The result is a small, quiet, always-on
file server that receives backups from your Mac or PC.

> **Jargon**
> - **NAS** — *Network Attached Storage*. A small computer with a hard disk in it, whose only
>   job is to share files over your home network.
> - **Debian** — one of the oldest and most stable Linux distributions. A *distribution* is a
>   complete, ready-to-use collection of the Linux kernel plus all the programs around it.
> - **Firmware** — the software built into a device by its manufacturer. Replacing the
>   firmware means replacing the manufacturer's software with your own.

### Be honest with yourself about the result

This machine was built in 2007. After all your work you get:

| | Reality |
|---|---|
| Speed | Roughly **10–20 MB/s**. Copying 100 GB takes about 2 hours. |
| Memory | **128 MB RAM.** A modern phone has 50–100× more. |
| Power use | About **15–20 W**, around €40–50 per year of electricity. |
| Support lifetime | Security updates for this processor type end around **2028**. |
| Time needed | **3–5 hours** of work, plus waiting. |

A second-hand mini-PC for €50 would be roughly five times faster and supported for many more
years. **Do this project because you enjoy it, or because you want to keep working hardware
out of the bin — not because it is the cheapest way to get storage.**

### What you need to be comfortable with

- Typing commands into a black window and reading the output carefully.
- Opening a computer case and handling a hard disk.
- Accepting that you might have to start a chapter again.

You do **not** need to know Linux beforehand. You do need to read each command before you
press Enter.

### The one rule that matters

> ⚠️ **Several commands in this guide erase an entire disk.** They do it instantly, without
> asking, and there is no undo and no recycle bin. The difference between erasing the right
> disk and erasing your laptop's disk is a single letter. Chapter 7 shows you how to check.
> Never skip that check, not even the second time.

---

## Chapter 1 — Understanding the machine

Read this once. It explains *why* the later steps are so fussy, which makes them much easier
to follow.

### What is inside the box

| Part | What it is |
|---|---|
| **Marvell Orion 88F5182**, 400 MHz | The processor. It uses the **ARM** design, not the Intel/AMD design your laptop uses. |
| **128 MB RAM** | Working memory. This is the tightest limit in the whole project. |
| **1 SATA bay** | Room for one ordinary 3.5-inch desktop hard disk. |
| **Gigabit Ethernet** | Network port. |
| **A small flash chip** | Holds the bootloader, and nothing else. |

> **Jargon**
> - **ARM** — a processor family used in phones, Raspberry Pis and many small devices.
>   Programs compiled for Intel/AMD chips **cannot run** on ARM, and vice versa. This is why
>   you cannot simply download a normal Linux installer and run it.
> - **armel** — Debian's name for the specific ARM variant this chip needs. When you see
>   `--arch=armel` later, that is what it means. Your laptop is almost certainly `amd64`.
> - **Bootloader** — the very first program that runs when you switch a computer on. Its job
>   is to find the operating system and start it. On a PC this is usually GRUB; here it is
>   called **U-Boot**.

### The crucial detail: the operating system lives on the hard disk

On this NAS, only the bootloader is stored inside the machine. **The kernel and the entire
operating system live on the hard disk itself.**

> **Jargon**
> - **Kernel** — the core of Linux. It talks directly to the hardware. Everything else runs on
>   top of it.

Two consequences shape this whole guide:

1. **Put in a blank disk and the machine has no operating system at all.** It will switch on,
   find nothing, and sit there. This is normal and expected — you have not broken anything.
2. **You cannot "brick" this device by following this guide.** Nothing you do touches the
   bootloader chip. If the install fails, take the disk out, plug it into your computer, and
   start again. The machine itself is safe.

That is unusually forgiving, and it is why this is a good first hardware project.

### Why U-Boot is so picky

The U-Boot on this board is from 2007. It knows how to read a hard disk, but only in the
limited way that was normal back then. It cannot understand modern disk formatting
techniques. Three old-fashioned choices in this guide exist purely to keep U-Boot happy:

- An **MBR** partition table instead of the modern **GPT**.
- The `/boot` area formatted as **ext3** instead of the modern **ext4**.
- An **inode size of 128 bytes** instead of the modern default of 256.

You do not need to understand these deeply. You need to type them exactly as written. They
are each explained where they appear.

---

## Chapter 2 — Shopping list and preparation

### Hardware to buy or find

| Item | Notes | Rough price |
|---|---|---|
| **3.5" SATA hard disk, max 2 TB** | See the warnings below. Must be **CMR**, not SMR. | €60–80 |
| **USB-to-SATA adapter** | A cable or dock that lets you connect a bare 3.5" hard disk to a computer's USB port. Must include its **own power supply** — 3.5" disks need 12 V, which USB cannot provide. | €20–30 |
| **USB stick, 8 GB or larger** | Will be completely erased. | €8 |
| **USB-to-TTL serial adapter (3.3 V)** | Optional but strongly recommended. See Chapter 5. | €5–10 |
| **Small Phillips screwdriver** | | |

#### Why exactly 2 TB, and not more

The old bootloader can only reliably read a disk using the **MBR** partitioning scheme, which
cannot address more than 2 TiB (about 2199 GB). A disk sold as "2 TB" is actually 1.82 TiB, so
it fits comfortably. A 4 TB disk would require **GPT**, which this 2007 bootloader may not
understand.

#### Why CMR and not SMR

These are two ways of writing data to a spinning disk. **SMR** disks overlap the data tracks
to fit more in, which makes them cheap but extremely slow when rewriting data — sometimes
dropping to a few MB/s. On a machine this slow already, that is unusable. Manufacturers often
hide this in the spec sheet; search for `<model number> CMR SMR` before buying.

#### Why "512e" and not "4Kn"

This describes how the disk presents itself to the computer. Old software expects 512-byte
blocks. A **512e** disk pretends to use 512-byte blocks even though it internally uses 4096.
A **4Kn** disk does not pretend, and this 2007 hardware cannot cope. Almost every consumer
disk is 512e; 4Kn is rare and usually sold for servers. Check before buying.

### What you need before you start

- **A wired internet connection** for the NAS, and for the computer you work on.
- **Access to your internet router's admin page**, so you can see which IP address the NAS
  gets. Usually `http://192.168.1.1` or `http://192.168.178.1`.
- **A free evening.** Do not start this an hour before you need the network.

---

## Chapter 3 — Rescue the data on your old disk *(optional)*

**Skip this chapter if the old 500 GB disk contains nothing you want.**

Everything from here on assumes the old disk's contents are expendable.

1. Switch the NAS off and unplug it. Remove the old disk (Chapter 6 shows how to open it).
2. Connect it to your computer with the USB-to-SATA adapter.
3. Windows and macOS **cannot read** this disk — Buffalo used the Linux filesystems `XFS` or
   `ext3`. You have two choices:
   - Copy the files off later, from inside the Linux environment you build in Chapter 4.
   - Or, easier: put the old disk back in the NAS, switch it on, and copy your files over the
     network the normal way *before* you start this project.

The second option is far simpler. Do that first, then come back here.

> 💡 **Keep the old disk.** Do not erase it and do not throw it away until your new setup has
> been running happily for a few weeks. It is your only way back.

---

## Chapter 4 — Build a Linux workstation

### Why this chapter exists

To install Debian onto the NAS's disk, you need a computer that can run Linux commands and
write Linux filesystems. Windows and macOS cannot do this. So you will temporarily turn your
own computer into a Linux machine — **without touching or changing anything on it**.

The trick is a **live USB**: a USB stick that a computer can start from, which runs a complete
Linux system entirely in memory. Switch off, pull the stick out, and your computer is exactly
as it was.

> **Jargon**
> - **ISO file** — a single large file (2–5 GB) containing an exact image of an installation
>   disc. You cannot just copy it onto a USB stick; it has to be *written* with a special tool.
> - **Live USB** — a bootable USB stick that runs an operating system without installing it.
> - **Boot** — to start a computer and load an operating system.

### Choose your route

| Your computer | Route |
|---|---|
| **Windows PC** | Route A — live USB. Works well. |
| **Intel Mac** (2020 or older) | Route A — live USB. Works well. |
| **Apple Silicon Mac** (M1/M2/M3/M4) | Route B — virtual machine. More fiddly. |
| **You own a Raspberry Pi** | Route C — use it directly. Easiest of all. |

---

### Route A — Live USB (Windows and Intel Mac)

#### A1. Download Ubuntu

We use **Ubuntu** rather than Debian for this step. Ubuntu is built on Debian, is beginner
friendly, and its "Try Ubuntu" mode is ideal here. (You are installing *Debian* on the NAS;
Ubuntu is only the temporary toolbox on your own computer.)

- Go to **<https://ubuntu.com/download/desktop>**
- Download the latest **LTS** version. *LTS* means *Long Term Support* — the stable version
  recommended for most people.
- You get a file like `ubuntu-26.04-desktop-amd64.iso`, around 5 GB.

#### A2. Download balenaEtcher

**balenaEtcher** is a free, open-source program whose only job is to write an ISO file
correctly onto a USB stick. It is popular because it is almost impossible to misuse: it
refuses to write to your system disk, and it verifies the result afterwards.

- Go to **<https://etcher.balena.io/>**
- Download the version for Windows or macOS and install it.

*Alternatives, if you prefer:* [Rufus](https://rufus.ie/) (Windows only) or
[Raspberry Pi Imager](https://www.raspberrypi.com/software/) (both platforms). Any of them
works.

#### A3. Write the stick

1. Plug in the USB stick. **Everything on it will be destroyed.**
2. Open balenaEtcher.
3. *Flash from file* → select the `.iso` you downloaded.
4. *Select target* → choose your USB stick. Check the size matches — if it says 1 TB, that is
   your external disk, not your stick.
5. *Flash!* and wait. Roughly 5–10 minutes.
6. Ignore any Windows popup offering to format the drive afterwards. Click **Cancel**.

#### A4. Start your computer from the stick

**On a Windows PC:**

1. Shut down fully (not sleep, and not "restart").
2. Switch on and immediately tap the boot-menu key repeatedly. It is usually **F12**,
   sometimes **F9**, **F11** or **Esc**. The correct key is often shown for a second on the
   manufacturer's logo screen.
3. Choose your USB stick from the list.
4. At the Ubuntu menu, choose **Try Ubuntu** (*not* "Install Ubuntu").

If the stick does not appear, search the web for `<your PC model> boot from USB` — every
manufacturer does it slightly differently. You may need to disable **Secure Boot** in the
BIOS/UEFI settings, although recent Ubuntu versions normally work with it switched on.

**On an Intel Mac:**

1. Shut down.
2. Switch on while holding the **Option (⌥)** key.
3. Choose the orange **EFI Boot** icon.
4. Choose **Try Ubuntu**.

#### A5. Get online

Once the Ubuntu desktop appears, connect to your Wi-Fi or plug in a network cable. You need
internet for the rest of the guide.

---

### Route B — Virtual machine (Apple Silicon Mac)

An M-series Mac cannot start from an Ubuntu USB stick. Instead you run Linux *inside* macOS.

> **Jargon**
> - **Virtual machine (VM)** — a complete computer simulated in software, running as a window
>   on your normal desktop.
> - **USB passthrough** — handing a physical USB device from the host (macOS) to the guest
>   (Linux), so Linux sees the disk directly.

1. Install **UTM**, a free virtual-machine program for macOS: **<https://mac.getutm.app/>**
   (the version on their site is free; the App Store version is a paid convenience).
2. Download the **ARM64** version of Ubuntu Server:
   **<https://ubuntu.com/download/server/arm>**
3. In UTM, create a new **Virtualize → Linux** machine, select that ISO, give it 4 GB RAM,
   and start it. Choose **Try or Install**, then work from the command line.
4. Plug in your USB-to-SATA adapter with the new disk attached. In the UTM window, use the
   USB icon in the toolbar to pass the device through to the VM.
5. Inside the VM, run `lsblk` and confirm your 2 TB disk appears.

> **Honest warning:** USB passthrough is the fiddly part, and failures here are hard to
> diagnose for a beginner. If you have any access to a Windows PC or an old laptop, Route A
> is genuinely much easier. Borrowing a laptop for an evening is a reasonable shortcut.

---

### Route C — Raspberry Pi

If you have a Raspberry Pi 4 or 5 running Raspberry Pi OS, you already have everything. Plug
the USB-to-SATA adapter in and skip to the next section. No live USB needed.

---

### Install the tools you will need

Whichever route you took, you are now looking at a Linux command line. On the Ubuntu desktop,
open **Terminal** (press the Windows/Command key, type `terminal`, press Enter).

Type this and press Enter:

```bash
sudo apt update
sudo apt install -y debootstrap qemu-user-static binfmt-support parted
```

> **Jargon**
> - **Terminal** — the window where you type commands.
> - **sudo** — "do this as the administrator". Linux protects system-level changes behind it.
>   In a live session there is usually no password; on a Pi, use your normal one.
> - **apt** — Debian and Ubuntu's software installer.
> - **debootstrap** — the tool that builds a fresh Debian system inside a folder.
> - **qemu-user-static** — lets your Intel/ARM64 computer run the NAS's `armel` programs by
>   emulating them. Without this, step 9 cannot work.
> - **parted** — a disk partitioning tool.

### ✅ Check before continuing

```bash
debootstrap --version
```

If it prints a version number, you are ready.

---

## Chapter 5 — Connect a serial console *(strongly recommended)*

### Why bother

When the NAS starts up, it prints a running commentary — which disk it found, which file it
loaded, what went wrong. Without a serial console you see none of this. You get a blinking
light and no explanation.

You *can* finish this project without one. But if something fails at Chapter 12, you will be
guessing blindly, and you will probably end up buying the €6 adapter anyway.

> **Jargon**
> - **Serial console / UART** — a very simple, very old way for a device to send text to
>   another computer over three wires. Almost every embedded device has one hidden inside.
> - **TTL 3.3 V** — the voltage level these wires use. **This matters.** A 5 V adapter can
>   permanently damage the NAS's processor.

### What to buy

A **USB-to-TTL serial adapter, 3.3 V**, based on an FTDI FT232RL, CP2102 or CH340 chip. Search
those terms on any electronics shop. Many adapters have a jumper to switch between 5 V and
3.3 V — **set it to 3.3 V.**

### Find the right connector on the board

Open the NAS (Chapter 6) and look at the circuit board. You may see a block of **20 holes in
two rows of 10**. That is *not* the serial port — that is **JTAG**, a chip-debugging
connector. Ignore it.

You are looking for a smaller, separate row of about **4 holes**.

> **Jargon**
> - **JTAG** — a low-level debugging interface used to rescue devices with destroyed
>   bootloaders. Not needed for this project, because the bootloader here is never touched.

### Identify the pins safely

Do not trust a photo from a different board. Measure, using a multimeter:

1. **Unplugged**, set the meter to continuity (the beep setting). Touch one probe to the metal
   chassis. The pin that beeps is **GND** (ground).
2. **Powered on**, meter on DC volts, black probe on that GND pin:
   - A pin at a steady **3.3 V** that never changes = **VCC** (power).
   - A pin that sits near 3.3 V but visibly flickers while the NAS starts = **TX** (the NAS's
     output — this is the one you want).
   - A pin with an unstable, floating reading = **RX** (the NAS's input).

### Wire it up

With the NAS **unplugged from mains**, solder a 4-pin header into the holes and connect:

| Adapter | NAS board |
|---|---|
| GND | GND |
| RX | TX |
| TX | RX |
| **VCC** | ❌ **nothing — leave it disconnected** |

> ⚠️ Connecting the adapter's power wire is the single most common way people destroy these
> boards. The NAS powers its own serial port. You only need three wires.

Note that TX connects to RX and vice versa — one device's "talk" wire is the other's "listen"
wire.

### Open the connection

On your Linux machine:

```bash
sudo apt install -y picocom
sudo picocom -b 115200 --logfile boot.log /dev/ttyUSB0
```

Leave with `Ctrl-A` then `Ctrl-X`.

- `115200` is the speed. `8N1` (8 data bits, no parity, 1 stop bit) is the default and is
  correct.
- `--logfile boot.log` saves everything to a file. Do this — when a boot fails, you will want
  to read back through it.

On Windows you can instead use **PuTTY** (<https://www.putty.org/>): choose *Serial*, enter the
COM port from Device Manager, and set speed 115200.

### ✅ Check before continuing

Power on the NAS with its **original disk still fitted**. Within a few seconds you should see
text appear. If you do, your console works — and you have now confirmed it *before* you need
it in anger.

- **Garbled nonsense?** Wrong speed. Try `-b 57600`, then `-b 9600`.
- **Nothing at all?** Swap the TX and RX wires. Adapter labels are inconsistent.

---

## Chapter 6 — Fit the new disk

1. **Unplug the NAS from the mains.** Not just the power switch.
2. Touch a radiator or tap first to discharge static electricity, then avoid shuffling on
   carpet. Static can silently damage electronics.
3. Open the case. On the HS-DHGL the front panel slides off, revealing screws along the side.
   Work gently — 20-year-old plastic clips are brittle. If you feel real resistance, stop and
   look for another screw.
4. Unplug the two cables from the old disk: a narrow flat **SATA data** cable and a wider
   **SATA power** cable. Both just pull out; some have a small clip to squeeze.
5. Remove the mounting screws, slide the old disk out, slide the new one in, and reconnect
   both cables firmly.
6. **Leave the case open for now.** You will very likely take the disk out again.

### ✅ Check before continuing

Both cables are fully seated, and the old disk is somewhere safe.

---

## Chapter 7 — Partition the disk

Now take the **new** disk back out and connect it to your Linux workstation using the
USB-to-SATA adapter.

> **Jargon**
> - **Partition** — a section of a disk, treated as if it were a separate disk. One physical
>   disk can hold several partitions for different purposes.
> - **Partition table** — the index at the start of the disk that records where each partition
>   begins and ends. **MBR** is the old scheme (max 2 TiB); **GPT** is the modern one. We use
>   MBR because the 2007 bootloader understands it.
> - **`/dev/sdX`** — how Linux names disks: `/dev/sda`, `/dev/sdb`, and so on. The final letter
>   depends on the order things were plugged in, so it is **never the same twice**.

### 🛑 Identify the disk. This is the step where people lose their data.

```bash
lsblk
```

You will see something like:

```
NAME   SIZE TYPE MOUNTPOINTS
sda    476G disk
├─sda1   1G part /boot/efi
└─sda2 475G part /
sdb    1.8T disk
```

Here `sda` is the computer's own disk (it has partitions that are *mounted*, and it is 476G).
`sdb` is the new 2 TB disk — 1.8T, no mount points.

**Confirm it properly:**

```bash
sudo parted /dev/sdb print
```

Check the reported **model and size** match the disk you bought. If anything is unclear,
unplug the adapter, run `lsblk` again, see which entry disappeared, and plug it back in. That
is the definitive test.

**From here on, replace `sdX` with your actual letter in every command.** Getting this wrong
erases your computer.

### Create the partitions

```bash
sudo parted /dev/sdX mklabel msdos
sudo parted -a optimal /dev/sdX mkpart primary ext3       1MiB    1025MiB
sudo parted -a optimal /dev/sdX mkpart primary ext4       1025MiB 13GiB
sudo parted -a optimal /dev/sdX mkpart primary linux-swap 13GiB   15GiB
sudo parted -a optimal /dev/sdX mkpart primary ext4       15GiB   100%
sudo parted /dev/sdX set 1 boot on
```

What you just created:

| Partition | Size | Purpose |
|---|---|---|
| `sdX1` | 1 GB | **`/boot`** — the bootloader reads the startup files from here |
| `sdX2` | 12 GB | **`/`** (root) — Debian itself |
| `sdX3` | 2 GB | **swap** |
| `sdX4` | ~1.8 TB | **`/srv`** — your actual data |

> **Jargon**
> - **Swap** — disk space used as emergency overflow when RAM runs out. With only 128 MB of
>   RAM, this is **not optional**. Without it, programs will be killed at random.
> - **`/` (root)** — the top of the Linux directory tree. Everything hangs off it.
> - **`mklabel msdos`** — confusingly, "msdos" is parted's name for the MBR scheme. Nothing to
>   do with DOS.

### ✅ Check before continuing

```bash
sudo parted /dev/sdX print
```

Four partitions, sizes roughly as in the table, `Partition Table: msdos`.

---

## Chapter 8 — Create the filesystems

A partition is just an empty region. A **filesystem** is the structure written into it that
lets it hold files and folders — the difference between an empty room and a room with shelves.

```bash
sudo mkfs.ext3 -I 128 -L boot /dev/sdX1
sudo mkfs.ext4 -L root /dev/sdX2
sudo mkswap  -L swap /dev/sdX3
sudo mkfs.ext4 -L data /dev/sdX4
```

> ⚠️ **`-I 128` is the single most important character sequence in this guide.**
>
> An **inode** is a small record storing a file's information — size, permissions, where its
> contents live. Modern Linux makes these 256 bytes. The 2007 bootloader can only read
> 128-byte inodes. If you forget this flag, everything will appear to work perfectly right up
> until the NAS refuses to start, with no error explaining why.

Note that `/boot` uses **ext3** (older) while the others use **ext4** (modern). Only `/boot`
has to be readable by the ancient bootloader; the Linux kernel reads the rest, and it
understands ext4 fine.

### Write down the UUIDs

```bash
sudo blkid /dev/sdX1 /dev/sdX2 /dev/sdX3 /dev/sdX4
```

Copy the output into a text file. You need it in Chapter 10.

> **Jargon**
> - **UUID** — *Universally Unique Identifier*. A long random-looking code permanently attached
>   to each filesystem. Because `/dev/sdX` letters change depending on plug order, we identify
>   disks by UUID instead — it never changes.

### ✅ Check

```bash
sudo dumpe2fs -h /dev/sdX1 | grep "Inode size"
```

It must say **128**. If it says 256, run the `mkfs.ext3` command again with `-I 128`.

---

## Chapter 9 — Install the Debian base system

### Choose your Debian version

| | **Debian 12 "Bookworm"** ✅ recommended | **Debian 13 "Trixie"** |
|---|---|---|
| Kernel | Included in Debian itself | **Not included** — needs a third-party kernel |
| Difficulty | Straightforward | Extra steps, extra things to go wrong |
| Supported until | ~2028 | ~2028 |

Debian 13 removed support for this processor family from its own kernel packages — the `armel`
kernels now only cover the Raspberry Pi 1 and Zero. Since both versions reach end of support
at about the same time, **Bookworm is the sensible choice for a first attempt.** The rest of
this guide uses it.

> **Jargon**
> - Debian releases have both a number and a *codename* from Toy Story. Bookworm = 12,
>   Trixie = 13. Commands use the codename.

### Mount the new filesystems

**Mounting** means attaching a filesystem to a folder so you can read and write it.

```bash
sudo mount /dev/sdX2 /mnt
sudo mkdir -p /mnt/boot
sudo mount /dev/sdX1 /mnt/boot
```

`/mnt` is now a window onto your NAS's future root filesystem.

### Download and unpack Debian

```bash
sudo debootstrap --arch=armel --foreign bookworm /mnt http://deb.debian.org/debian
```

- `--arch=armel` — build it for the NAS's processor, not yours.
- `--foreign` — do only the part that can be done on a different processor type. This
  downloads several hundred MB, so allow 10–20 minutes.

### Finish the installation using emulation

```bash
sudo cp /usr/bin/qemu-arm-static /mnt/usr/bin/
sudo chroot /mnt /debootstrap/debootstrap --second-stage
```

> **Jargon**
> - **chroot** — "change root". It makes a folder temporarily behave as if it were the whole
>   system, so you can run the NAS's programs and configure them from your own computer.
> - The `qemu-arm-static` copy is what lets your Intel or ARM64 computer *pretend* to be the
>   NAS's processor. This is why the next steps run slowly — every instruction is being
>   translated.

Expect 10–30 minutes. Some warnings are normal; errors that stop the process are not.

### ✅ Check

```bash
ls /mnt
```

You should see the standard Linux folders: `bin  boot  dev  etc  home  lib  usr  var` and
more.

---

## Chapter 10 — Configure the system

Step inside the new system:

```bash
sudo chroot /mnt
```

Your prompt changes. **Every command from here until you type `exit` runs inside the NAS's
future system.**

### Set a root password

```bash
passwd
```

> **Jargon**
> - **root** — the all-powerful administrator account on Linux. Choose a real password.

### Name the machine and install essentials

```bash
echo linkstation > /etc/hostname
apt update
apt install -y openssh-server ifupdown ca-certificates locales
```

> **Jargon**
> - **SSH** — *Secure Shell*. Lets you open a command line on the NAS from your own computer
>   over the network. This is how you will manage it from now on.

### Tell the system about its disks

Create `/etc/fstab` — the list of which filesystem goes where at startup.

```bash
nano /etc/fstab
```

> **Jargon**
> - **nano** — a simple text editor. Arrow keys to move, type normally, `Ctrl-O` then Enter to
>   save, `Ctrl-X` to quit. (If you have heard people complain about being unable to exit
>   `vim` — this is why we use `nano`.)

Type this, pasting in the UUIDs you saved in Chapter 8:

```
UUID=<root-uuid>  /      ext4  defaults,noatime  0 1
UUID=<boot-uuid>  /boot  ext3  defaults          0 2
UUID=<swap-uuid>  none   swap  sw                0 0
UUID=<data-uuid>  /srv   ext4  defaults,noatime  0 2
```

Each UUID goes in without quotes, exactly as `blkid` printed it.

### Set up networking

```bash
nano /etc/network/interfaces
```

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
```

> **Jargon**
> - **DHCP** — the NAS asks your router for an IP address automatically. Simplest option.
> - **eth0** — the name of the wired network port.

### Tune for 128 MB of RAM

These two settings make a real difference on a machine this small:

```bash
echo 'vm.swappiness=10' > /etc/sysctl.d/99-lowmem.conf
mkdir -p /etc/systemd/journald.conf.d
printf '[Journal]\nStorage=volatile\nRuntimeMaxUse=16M\n' > /etc/systemd/journald.conf.d/lowmem.conf
```

The first tells Linux not to use swap unnecessarily, since swapping to a slow spinning disk
hurts. The second stops the system log from being written to disk and limits it to 16 MB of
memory.

### Enable the serial console

```bash
systemctl enable serial-getty@ttyS0
```

This gives you a login prompt on the serial cable — very useful if networking misbehaves.

---

## Chapter 11 — Install the kernel

Still inside the chroot.

The **kernel** is what actually talks to this specific hardware. Debian's kernel for these
Marvell chips is called `linux-image-marvell`.

```bash
apt install -y linux-image-marvell flash-kernel
```

> **Jargon**
> - **flash-kernel** — a Debian tool that knows about hundreds of small ARM devices. It takes
>   the kernel and converts it into the exact format each device's bootloader expects, then
>   puts it in the right place. It runs automatically whenever the kernel is updated.

If `flash-kernel` cannot work out which device it is on, tell it:

```bash
echo 'Buffalo Linkstation Pro/Live' > /etc/flash-kernel/machine
flash-kernel
```

### ✅ Check — do not skip this

```bash
ls -la /boot/
```

You **must** see these two files:

```
uImage.buffalo
initrd.buffalo
```

If they are missing, the NAS will not start. Consult
[the Debian on Buffalo wiki](https://github.com/1000001101000/Debian_on_Buffalo/wiki), which
is the authoritative, actively maintained source for these devices.

### Leave cleanly

```bash
exit
sudo rm /mnt/usr/bin/qemu-arm-static
sudo umount /mnt/boot
sudo umount /mnt
sudo sync
```

> **Jargon**
> - **umount** — safely detach a filesystem. Removing a disk without this can corrupt it.
> - **sync** — force everything still waiting in memory to be written to disk.

Now unplug the USB adapter.

---

## Chapter 12 — First boot

1. Fit the disk in the NAS and reconnect both cables.
2. Connect the network cable.
3. Connect the serial cable, if you have one, and open `picocom`.
4. Power on.

### What you should see

On the serial console: U-Boot's banner, then messages about loading `uImage.buffalo`, then
several screens of kernel startup text, ending in a login prompt. The whole process takes
1–3 minutes — this is a slow machine, so be patient before concluding it has failed.

### Find it on the network

Open your router's admin page and look for a new device named `linkstation` in the DHCP client
list. Note its IP address, then from your own computer:

```bash
ssh root@192.168.1.123
```

(macOS and Windows both have `ssh` built in — use Terminal or PowerShell.)

The first time it will ask about a "fingerprint". Type `yes`.

### ✅ Check

```bash
uname -a
free -h
df -h
```

These show the kernel version, memory (about 120 MB usable), and your disks with a large
`/srv`. If you see them, **you have Debian running on a 2007 NAS.** That is the hard part
done.

---

## Chapter 13 — Turn it into a NAS

### The best use for this machine

Honestly: an **automated backup target**. It writes big files steadily, which is the one thing
a 400 MHz processor does adequately. Streaming video or running web applications is not
realistic.

### Recommended: restic over SSH

**restic** is a modern backup program. Crucially, it does all the hard work — compression,
encryption, deduplication — **on your Mac or PC**, and asks the NAS only to store the finished
files. That plays to this hardware's one strength.

> **Jargon**
> - **Deduplication** — storing repeated data only once. Ten backups of a mostly unchanged
>   laptop take barely more space than one.

On the NAS:

```bash
mkdir -p /srv/backup
```

On your Mac (install [Homebrew](https://brew.sh/) first if needed):

```bash
brew install restic
restic -r sftp:root@192.168.1.123:/srv/backup init
restic -r sftp:root@192.168.1.123:/srv/backup backup ~/Documents
```

Full documentation: **<https://restic.net/>**

### Optional: Windows/Mac file sharing

```bash
apt install -y samba
```

**Samba** implements the SMB protocol, the one Windows File Explorer and macOS Finder use for
network drives. Edit `/etc/samba/smb.conf` to add a share pointing at `/srv`.

Be aware each connected user costs 15–25 MB of RAM. Three or four simultaneous users will
exhaust this machine.

### ⚠️ A note about Time Machine

It is technically possible to use this as a Time Machine destination, but it is **not
recommended**. Time Machine over a network creates a single large disk-image file written in
thousands of small scattered pieces — the worst possible workload for this hardware, and a
well-known source of corrupted backups. Use a USB disk plugged directly into your Mac for
Time Machine, and this NAS for restic as your *second* copy.

> 💡 **The 3-2-1 rule:** three copies of your data, on two different types of media, one of
> them off-site. This NAS is a good second copy. It should never be your only copy.

---

## Chapter 14 — Keeping it healthy

### Updates

```bash
ssh root@<ip>
apt update && apt upgrade
```

Do this monthly. If a kernel update arrives, `flash-kernel` regenerates the boot files
automatically — but reboot while you are still nearby, not remotely.

### Security

> ⚠️ **Never expose this machine directly to the internet.** Do not forward ports to it from
> your router. Its processor family loses security support around 2028, and the hardware
> cannot run modern protections. On your home network behind a router it is fine.

For remote access, use a VPN such as [WireGuard](https://www.wireguard.com/) or
[Tailscale](https://tailscale.com/) rather than opening ports.

### Watch the disk

```bash
apt install -y smartmontools
smartctl -a /dev/sda
```

**SMART** is the disk's own self-diagnosis system. Rising *Reallocated Sector Count* or
*Pending Sector Count* means the disk is dying — replace it before it fails.

### The other ageing part

The power supply's capacitors are also twenty years old. If you ever see bulging or leaking on
the cylindrical components inside, stop using it. That, rather than the disk, is the most
likely eventual failure.

---

## Appendix A — Glossary

| Term | Meaning |
|---|---|
| **apt** | Debian/Ubuntu's software installer |
| **ARM** | Processor family used in phones and small devices; incompatible with Intel/AMD software |
| **armel** | Debian's name for the older ARM variant this NAS needs |
| **Bootloader** | First program to run at power-on; finds and starts the operating system |
| **chroot** | Temporarily treat a folder as the whole system |
| **CMR / SMR** | Two hard-disk recording methods; SMR is cheaper but far slower at rewriting |
| **debootstrap** | Tool that builds a fresh Debian system inside a folder |
| **DHCP** | Automatic IP address assignment by your router |
| **ext3 / ext4** | Linux filesystems; ext3 is older, ext4 modern |
| **Filesystem** | The structure that lets a partition store files and folders |
| **Firmware** | Software built into a device by its manufacturer |
| **flash-kernel** | Debian tool that formats the kernel correctly for small ARM devices |
| **fstab** | The file listing which filesystem is mounted where at startup |
| **inode** | A record describing one file; must be 128 bytes on `/boot` here |
| **ISO** | A single file containing a complete disc image |
| **JTAG** | Low-level chip debugging connector; not used in this guide |
| **Kernel** | The core of Linux, which talks to the hardware |
| **Live USB** | Bootable stick running an OS without installing it |
| **MBR / GPT** | Old and new partition-table schemes; MBR is limited to 2 TiB |
| **Mount** | Attach a filesystem to a folder so it can be used |
| **NAS** | Network Attached Storage — a network file server |
| **Partition** | A section of a disk treated as a separate unit |
| **QEMU** | Emulator that lets one processor type run another's programs |
| **root** | The administrator account, and also the top of the directory tree |
| **Samba** | Software providing Windows/Mac-style network file sharing |
| **SMART** | A disk's built-in health self-monitoring |
| **SSH** | Encrypted remote command line |
| **sudo** | Run a command with administrator rights |
| **Swap** | Disk space used as overflow when RAM is full |
| **UART / serial console** | Simple 3-wire text connection for watching a device boot |
| **UUID** | Permanent unique identifier for a filesystem |

---

## Appendix B — Troubleshooting

### Nothing happens at power-on; no lights

Power supply or mains cable. Not a software problem.

### Lights come on, but nothing on the network, and no serial console

Almost always the `/boot` partition. In order of likelihood:

1. **Inode size wrong.** Reconnect the disk to your computer and run
   `sudo dumpe2fs -h /dev/sdX1 | grep "Inode size"`. It must be 128.
2. **Boot files missing or misplaced.** `uImage.buffalo` and `initrd.buffalo` must sit
   directly in the top level of partition 1, not in a subfolder.
3. **Wrong partition marked bootable.** Re-run `sudo parted /dev/sdX set 1 boot on`.

### Serial console shows U-Boot, then it stops or retries

U-Boot found the disk but could not read the boot files. Same three causes as above.

If you can interrupt U-Boot (tap a key immediately at power-on) you get a `Marvell>>` prompt.
Then:

```
ide reset
ext2ls ide 0:1
```

If this lists your two `.buffalo` files, U-Boot can read them and the problem is elsewhere. If
it errors, your `/boot` filesystem is not readable — go back to Chapter 8.

### It boots but never appears on the network

Check `/etc/network/interfaces` via the serial console. Try `ip a` to see whether `eth0`
exists and has an address. Some kernels name the port `end0` instead — adjust accordingly.

### Everything is desperately slow, programs get killed

You forgot swap, or `/etc/fstab` has the wrong swap UUID. Check with `free -h`; the swap line
should show about 2 GB.

### I made a mistake and want to start over

Reconnect the disk to your computer and begin again at Chapter 7. **You cannot damage the NAS
this way** — the bootloader lives in a chip you never touch. Starting over costs time, not
hardware.

---

## Appendix C — Sources and further reading

- **Debian on Buffalo** — the actively maintained project for these devices. The authoritative
  source; if it contradicts this guide, believe it.
  <https://github.com/1000001101000/Debian_on_Buffalo>
  and its [wiki](https://github.com/1000001101000/Debian_on_Buffalo/wiki)
- **Buildroot for Buffalo** — alternative minimal custom firmware
  <https://github.com/1000001101000/Buildroot_for_Buffalo>
- **Buffalo NAS community wiki** — hardware details and model-specific notes
  <https://buffalonas.miraheze.org/wiki/Main_Page>
- **Debian armel installation guide**
  <https://d-i.debian.org/manual/en.armel/ch02s01.html>
- **restic backup documentation** — <https://restic.net/>
- **Ubuntu Desktop download** — <https://ubuntu.com/download/desktop>
- **balenaEtcher** — <https://etcher.balena.io/>

---

## Contributing

This guide was written for the **HS-DHGL**. The same procedure should work with only small
changes on other Marvell Orion-based Buffalo devices such as the **LS-GL** (LinkStation Pro)
and the **Kurobox Pro**, since they share the same chipset and bootloader arrangement.

If you complete this on a different model, or find a step that no longer matches reality,
please open an issue or pull request with what you had to change. Hardware guides age quickly;
corrections are genuinely valuable.

## Licence

Released under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). Use it, adapt
it, share it.
