### 一、安装
进入live，检查网络连接，然后执行`archinstall` 

安装完成后，archinstall 会提示你是否 chroot 进入新安装的系统。通常选择 No，然后重新启动电脑。
   reboot

安装后的配置（可选但推荐）：
 * 登录：
   系统重启后，你会看到引导加载程序菜单，选择 Arch Linux。然后进入命令行登录界面，输入你创建的用户名和密码。
 * 配置中文环境（如果你之前选择了英文）：
   * 编辑 /etc/locale.gen 文件，取消注释以下行：
     en_US.UTF-8 UTF-8
     zh_CN.UTF-8 UTF-8
  或者其他你需要的语言

   * 生成 locale：
     sudo locale-gen

   * 创建或编辑 /etc/locale.conf 文件：
     echo "LANG=en_US.UTF-8" | sudo tee /etc/locale.conf # 保持系统默认语言为英文
  如果你希望系统显示为中文，可以设置为：
  echo "LANG=zh_CN.UTF-8" | sudo tee /etc/locale.conf

     注意：即使 LANG 设为 en_US.UTF-8，只要安装了中文字体，应用程序通常也能正确显示中文。将 LANG 保持为 en_US.UTF-8 有助于避免一些非中文软件的兼容性问题。
   * 安装中文字体：
     sudo pacman -S noto-fonts-cjk # Google Noto CJK 字体，推荐
  或者其他你喜欢的中文字体，例如 wqy-zenhei, opendesktop-fonts
  sudo pacman -S wqy-zenhei

 * 更新系统：
   sudo pacman -Syu

 * 安装 AUR 助手（可选，但非常方便）：
   yay 或 paru 是常用的 AUR (Arch User Repository) 助手，可以方便地安装 AUR 中的软件。
   * 首先安装 git 和 base-devel：
     sudo pacman -S git base-devel

   * 安装 yay 为例：
     git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
cd ..
rm -rf yay

至此，你就完成了 Arch Linux 的安装和基本配置。archinstall 极大地简化了原本复杂的手动安装过程，让更多人能够体验 Arch Linux 的魅力。如果在安装过程中遇到任何问题，可以参考 Arch Wiki，它是 Arch Linux 最权威和详细的文档。

当你手动分区安装 Arch Linux 并只看到 Windows 启动项时，这通常意味着 GRUB 引导加载程序没有正确安装或没有检测到 Windows 启动项，或者 UEFI 固件的启动顺序没有正确设置。
以下是排查和解决这个问题的完整指南：
问题排查步骤：
 * 确认 UEFI 启动模式：
   确保你是在 UEFI 模式下安装的 Arch Linux 和 Windows。Legacy/BIOS 模式和 UEFI 模式的引导方式是不同的。如果你在 UEFI 模式下安装了 Windows，那么 Arch 也必须在 UEFI 模式下安装。
 * 确认 /boot/efi 分区正确挂载：
   * 在 Arch Linux 安装过程中，你的 /boot/efi 分区（即 EFI 系统分区，ESP）必须被正确识别并挂载到 /boot/efi。
   * 这个分区通常是 FAT32 格式，并且设置了 "boot" 或 "esp" 标志。
 * 确认 GRUB 安装到 ESP：
   * GRUB 必须安装到 EFI 系统分区 (ESP) 中。在 archinstall 中，当你选择引导加载程序时，它会提示你选择 ESP。
 * 确认 GRUB 配置生成：
   * GRUB 安装后，需要生成 grub.cfg 文件，这个文件负责检测系统上的所有操作系统（包括 Windows）。
解决方案步骤：
如果你的系统目前只能启动 Windows，你需要再次进入 Arch Linux 的 Live 环境进行修复。
第一部分：进入 Arch Linux Live 环境
 * 准备 Arch Linux 启动盘： 使用你之前安装时用的 Arch Linux ISO 制作的启动盘。
 * 从启动盘启动： 插入启动盘，重启电脑，并在 BIOS/UEFI 设置中选择从 USB/DVD 启动。
 * 进入 Arch Linux Live 环境： 选择 "Boot Arch Linux (x86_64)"。
 * 检查网络连接（可选但推荐）： 确保网络连接正常，以便下载可能需要的工具或更新。
   * ping archlinux.org
第二部分：Chroot 进入已安装的 Arch Linux 系统
这是最关键的一步，你需要将 Live 环境切换到你硬盘上安装的 Arch Linux 系统中。
 * 识别你的分区：
   * 使用 lsblk 命令查看你的磁盘和分区布局。
     lsblk

     你会看到类似这样的输出：
     NAME        MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
sda           8:0    0 931.5G  0 disk
├─sda1        8:1    0   512M  0 part  # 你的 EFI 系统分区 (ESP)
├─sda2        8:2    0 100G    0 part  # 你的 Arch Linux 根分区 (/)
└─sda3        8:3    0 831G    0 part  # 你的 Windows 分区

     请务必确认你的 Arch Linux 根分区 (例如 /dev/sda2) 和 EFI 系统分区 (例如 /dev/sda1)。
 * 挂载根分区：
   将你的 Arch Linux 根分区挂载到 /mnt。
   sudo mount /dev/sdaX /mnt  # 将 /dev/sdaX 替换为你的 Arch Linux 根分区，例如 /dev/sda2

 * 挂载 EFI 系统分区 (ESP)：
   将你的 EFI 系统分区挂载到 /mnt/boot/efi。如果 /mnt/boot/efi 目录不存在，请先创建它。
   sudo mkdir -p /mnt/boot/efi
sudo mount /dev/sdY /mnt/boot/efi # 将 /dev/sdY 替换为你的 EFI 系统分区，例如 /dev/sda1

 * 挂载其他必要文件系统：
   sudo arch-chroot /mnt

   现在你已经进入了你硬盘上的 Arch Linux 环境。命令行提示符会变成 (chroot) #。
第三部分：修复 GRUB 引导加载程序
在 (chroot) # 环境中执行以下步骤：
 * 安装或重新安装 GRUB：
   * 首先，确保 grub 和 efibootmgr 包已经安装。
     pacman -S grub efibootmgr os-prober

     os-prober 是用来检测其他操作系统的（如 Windows）。
   * 对于 UEFI 系统： 重新安装 GRUB 到你的 EFI 系统分区。
     grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ArchLinux --recheck

     * --efi-directory=/boot/efi：指定 EFI 系统分区的挂载点。
     * --bootloader-id=ArchLinux：指定在 UEFI 启动菜单中显示的名称。
 * 生成 GRUB 配置文件：
   grub-mkconfig 会扫描系统上的操作系统并生成 grub.cfg。
   grub-mkconfig -o /boot/grub/grub.cfg

   * 检查输出： 在这个命令的输出中，你应该能看到 GRUB 检测到 Windows 启动项的信息，类似 Found Windows Boot Manager on /dev/sdY。如果没看到，说明 os-prober 没有正确工作。
 * 退出 Chroot 环境并重启：
   exit

   sudo umount -R /mnt
sudo reboot

第四部分：重启后检查
重启后，你应该会看到 GRUB 菜单，其中包含 "Arch Linux" 和 "Windows Boot Manager" (或类似的 Windows 启动项)。
如果仍然没有 Arch Linux 或 Windows 启动项：
 * 检查 BIOS/UEFI 启动顺序：
   * 重启电脑，进入 BIOS/UEFI 设置界面。
   * 找到 "Boot Order" 或 "Boot Priority" 选项。
   * 确保 "ArchLinux" (或你设置的 bootloader-id) 在启动顺序中处于较高位置，或者直接选择它。
   * 如果只看到 "Windows Boot Manager"，但你确定 GRUB 已经正确安装，那么可能是在 UEFI 固件中没有注册 Arch 的引导项。
 * 手动添加 UEFI 启动项（如果 grub-install 失败或未注册）：
   在 Live 环境中 chroot 进入系统后，如果 grub-install 似乎成功但 UEFI 菜单里没有 Arch，可以使用 efibootmgr 检查或添加。
   * efibootmgr -v：查看当前的 UEFI 启动项。
   * 如果 Arch Linux 的引导项不存在，你可以尝试手动添加（但通常 grub-install 会自动完成）。
     efibootmgr -c -d /dev/sdY -p Z -L "Arch Linux" -l "\EFI\ArchLinux\grubx64.efi"

     * /dev/sdY：你的 EFI 系统分区所在的磁盘。
     * Z：你的 EFI 系统分区在磁盘上的分区号（例如，如果 ESP 是 /dev/sda1，则 Z 是 1）。
     * \EFI\ArchLinux\grubx64.efi：GRUB EFI 文件的路径，这个路径在你的 ESP 上。请根据实际情况调整（通常是 \EFI\ArchLinux\grubx64.efi 或 \EFI\GRUB\grubx64.efi）。
重要提示：
 * 仔细识别分区： 在 mount 命令中，务必将 /dev/sdaX 和 /dev/sdY 替换为你的实际分区名称。错误的挂载可能导致数据丢失。
 * 备份重要数据： 在进行任何分区或引导修复操作之前，始终建议备份重要数据。
 * Arch Wiki： 如果遇到更复杂的问题，请参考 Arch Wiki 上的 GRUB 页面：https://wiki.archlinux.org/title/GRUB 和 Dual boot with Windows 页面：https://wiki.archlinux.org/title/Dual_boot_with_Windows
按照这些步骤操作，你应该能够修复 GRUB 引导加载程序，并正确地启动你的双系统。

### 二、输入法
在 Wayland 环境下使用 Google Chrome 时，Fcitx5 输入法可能会遇到一些兼容性问题，这确实是用户常报告的一个痛点。主要原因在于 Chrome 对 Wayland 原生输入法协议（如 `text-input-v3`）的支持还在不断完善中，而 Fcitx5 依赖这些协议来提供完整的输入体验。

以下是一些常见的问题和可能的解决方案：

**常见问题：**

* **无法输入中文/其他语言：** 最常见的问题是无法切换输入法，或者即使切换了也无法输入中文等字符。
* **候选词窗口位置不正确：** 候选词窗口可能显示在屏幕的错误位置，或者根本不显示。
* **焦点切换问题：** 在多个 Chrome 窗口或不同应用之间切换时，输入法可能会失效。
* **重复输入：** 在某些情况下，按键可能会被重复输入。
* **退格/回车键问题：** 某些语言（如韩语）在 Chrome 中使用退格和回车键时可能会出现问题。

**可能的解决方案：**

1.  **启动参数：**
    为了让 Chrome 在 Wayland 下更好地支持输入法，通常需要添加特定的启动参数。这可以通过修改 Chrome 的 `.desktop` 文件或直接从终端启动时添加。

    尝试以下参数组合：
    * `--enable-features=UseOzonePlatform --ozone-platform=wayland --enable-wayland-ime`
    * 如果您的 Wayland 合成器和 Chrome 支持 `text-input-v3` 协议，可以尝试：`--enable-features=UseOzonePlatform --ozone-platform=wayland --enable-wayland-ime --wayland-text-input-version=3`

    **如何修改 `.desktop` 文件：**
    通常，Chrome 的 `.desktop` 文件位于 `/usr/share/applications/google-chrome.desktop` 或 `~/.local/share/applications/google-chrome.desktop`。
    找到 `Exec=` 开头的行，并在 `%U` 前面添加上述参数。
    例如：
    `Exec=/usr/bin/google-chrome-stable --enable-features=UseOzonePlatform --ozone-platform=wayland --enable-wayland-ime %U`

    **注意：**
    * 根据 Chrome 的版本和您的 Wayland 合成器（如 Gnome, KDE Plasma, Sway, Hyprland 等），所需的参数可能略有不同。
    * 较新版本的 Chrome 可能默认已经启用了 Wayland 支持，所以有些 Wayland 相关的参数可能不再需要，但 IME 相关的参数可能仍是必需的。
    * 一些用户报告说，在 Chrome 135 及更高版本中，特定问题已通过启用 `WaylandTextInputV3` 功能解决。您可以在 `chrome://flags` 中查找并启用相关标志。

**对于Xwayland**

编辑 `/etc/environment` 并添加以下几行，然后重新登录：
```
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```

### 三、hyprland自动配置主题

```bash
https://github.com/HyDE-Project/HyDE
```

#### 1.截图工具 
- 安装`hyprshot` 

  `yay -S hyprshot`
- 绑定按键 

  `bind = SUPER, R, exec, hyprshot -m region`

#### 2.Xwayland的cursor问题

1. 在`hyprland.conf`中设置环境变量如下：
```
env = XCURSOR_THEME,Adwaita
env = XCURSOR_SIZE,24
```
### 四、KDE主题配置
**Arch**  
```
sudo pacman -S python python-pipx
```
Download and install Konsave using pipx package manager.  
```
pipx install konsave
pipx inject konsave setuptools
```
Make sure that your “.local/bin” directory is in your current user’s PATH variable:  
```
echo "PATH=$PATH:.local/bin" >> ~/.bashrc && bash
```  
Download my Konsave file  
https://mega.nz/file/DV9FSBjS#QFIfWHFVCorLAbSN1o8iU47mEWwbm70llS3oNgoLWxs  
Import the .knsv file
```
konsave -i ./my-kde-desktop.knsv
```
Load the imported Konsave file
```
konsave -a my-kde-desktop
```
Relogin to apply the settings.

### 五、STM32单片机开发
1. 首先安装 ARM 交叉编译工具链：  
  `sudo pacman -S arm-none-eabi-gcc arm-none-eabi-binutils arm-none-eabi-gdb`
2. 安装 newlib 标准库：  
  `sudo pacman -S arm-none-eabi-newlib`  
3. 使用STM32CubeMX生成Cmake代码
4. 按照仓库中文件配置CmakeLists.txt
5. 清理构建目录并重新构建：  
  ```
  cd /home/justin/Desktop/test/build
  rm -rf *
  cmake ..
  make -j$(nproc)
  ```
6. 使用stlink烧录程序
