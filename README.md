# Linux on eMMC for Amlogic S905
When you get stubborn about the Amlogic S905...
A few years ago, I purchased an MXQ Pro 4K (extended name: Amlogic S905 with DTB Meson GXBB p201), an Android TV box designed to upgrade an old television by introducing the capabilities of Google's operating system, allowing access to apps and more. Later on, I discovered communities that had made these devices highly efficient by installing minimal Linux distributions on them. My first thanks go to those communities: the people behind LibreELEC (especially kszaq), coreELEC, Kodi, and everyone developing and sharing solutions. Using Kodi directly, without going through Android, made everything smoother, and the countless plugins enabled virtually unlimited expansion of the TV box. For example, tvheadend, when combined with a USB stick (similar to those used for SKY), transformed it into a DVB-T decoder as well.

Over the years, the advent of Android TVs has made this small but great device somewhat obsolete. As I always do when a device no longer meets the minimum requirements for its original purpose, I start looking for an alternative use… Since it's essentially an SBC (Single Board Computer) with decent technical specifications that, for the same price, surpass those of more well-known boards (like RPi, BPi, Odroid, Asus, etc.), its most logical use was as a Linux server. I immediately started looking into the most commonly used distributions for these devices: Armbian, DietPi, Alpine, and so on. That’s how I came across the work of balbes150, hexdump, SteeMan, and many others, to whom I am deeply grateful. Their efforts help recycle and give new life to devices otherwise destined for the trash, which has significant positive impacts both for the environment and for our wallets.

Installing Armbian for the Amlogic S905 on an SD card was simple and convenient, allowing access to a high-quality, highly efficient Debian distribution. I was used to installing the OS on eMMC since my days with LibreELEC, so I was ready to follow the instructions to proceed. This is where the problems began. At first, I couldn’t understand why an internal installation presented so many issues. The more I read, the clearer the bug became. Each time I tried formatting the internal memory, the TV box stopped responding entirely, and the only solution I knew was to flash the original Android image to reset the project and start over.

Everything changed when I found an article on the Armbian forum by pista. He had also faced the same issue, but unlike anyone else at the time, he approached it from a fresh perspective, finding an elegant solution that worked wonderfully. From his article, I learned that Amlogic’s engineers had made an error: the bootloader was placed where it shouldn’t have been, so any attempt to redesign the eMMC partitions resulted in a bricked device.

There is a command (`blkdevparts`) that can be passed as an argument from U-Boot directly to the kernel by setting the "bootargs" environment variable. This allows the kernel to receive an eMMC partition map, which we can’t create traditionally due to the previously mentioned limitations. Pista's original idea was to leverage this command to instruct the kernel on where to find the root partition. His second insight, however, was to use the "mmc read" command instead of the classic "fatload", since the kernel needs to be loaded by U-Boot, which knows nothing about how the internal memory is partitioned. The boot sequence, therefore, involves loading the kernel and device tree, passing the partition map to the former, and then letting it complete the boot process.

```mermaid
graph TD;
    Step 1 U-Boot Initialization U-Boot initializes and prepares the boot process-->Step 2 Load Kernel & Device TreeU-Boot loads the kernel and device tree directly from eMMC using mmc read-->Step 3 Transfer to RAM Kernel and device tree are transferred to RAM-->Step 4 Launch Kernel U-Boot launches the kernel using the bootm command-->Step 5 Pass Partition Map Kernel receives the eMMC partition map via the blkdevparts argument;
```
