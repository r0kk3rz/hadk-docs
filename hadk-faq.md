https://public.etherpad-mozilla.org/p/faq-hadk
Please, do not use browser-in-place translation tools (right-click translate-this) or similar. They overwrite the content on this page.
 
---------------------------------------------------------------------------------------------------------------------
 
#### So you booted your Sailfish OS? Congrats! No GUI? Oh dear :)
 
both usb0 and rndis0 interfaces might be up, remove usb0 from init-script and rebuild kernel image with make hybris-boot or make hybris-recovery
 
selinux and user session masking are now in HADK v1.1.1 section 9.2.1
 
USB tethering is now in HADK v1.1.1 section 9.2.3

-------------------------------------------------------------------------------------------------------------
 #### sudo: effective uid is not 0, is sudo installed setuid root?
 Just after chroot is entered through the `sdk` command, sudo fails to execute and fails with the previous message
check for suid mount flag in the bind-mounted root: if your home (not chroot home) is mounted with nosuid you need to remount it before chroot

      $HOST
      sudo mount -o remount,suid $HOME

If you're running an encrypted home partition with ecryptfs, you need to remount home anyway, even if nosuid flag is not set

      $HOST
      sudo mount -i -o remount,suid $HOME) 

repo init -u git://github.com/mer-hybris/android.git -b hybris-11.0 fails because of gpg
In sdk chroot gpg command is gpg2

      $MERSDK
      git config --global gpg.program gpg2

lipstick segfaults/no display
As you follow steps below, strace any of the binaries that would fail for non-obvious reasons. You'll need to install strace to do so: zypper in strace
test simple hwc as root:

    EGL_PLATFORM=hwcomposer test_hwcomposer

^^ strace if segfaults
if strace dies after open("/sys/kernel/debug/tracing/trace_marker..., perform

    systemctl mask sys-kernel-debug.mount

test_hwcomposer should not be used as reliable hwc test!! if fails, then try minimer:

    curl -O https://qtl.me/minimer3.tar.gz
    zypper in qt5-qtdeclarative-qmlscene
    tar -xf minimer3.tar.gz; cd minimer
    EGL_PLATFORM=hwcomposer /usr/lib/qt5/bin/qmlscene -platform hwcomposer main.qml

if fails as user, try as root
/system/bin/surfaceflinger
for more info: zypper in gdb
if you get test_hwcomposer, minimer or lipstick segfault, or test_hwcomposer or minimer running but doing nothing (as on m7)
Check if your device uses qcom_display-caf or display-legacy
Look in any of the BoardConfig.mk or BoardConfigCommon.mk in any of the device repos for the device for the variable TARGET_QCOM_DISPLAY_VARIANT. It should be set to either caf or legacy.
The repos included can be determined by looking at the -include device/$VENDOR/*/BoardConfig.mk or device/$VENDOR/*/BoardConfigCommon.mk lines at beginning the .mk files starting from the primary BoardConfig.mk
If you're on display-legacy or display-caf(repo sync before 2015.06.04) patch hwcomposer withhttp://pastebin.com/AfRXPKVA
From HABUILD_SDK recompile android hwcomposer*.so for your device
Find the name of the hwcomposer*.so module: run make modules | grep hwcomposer
If this command complains about missing column command run sudo apt-get install bsdmainutils)
Run `make hwcomposer.module_name` from results above
Once rebuilt, hwcomposer.*.so will be picked up and used by droid hal rebuild, and reside under /usr/libexec/droid-hybris/system/lib/hw
If your apps are crashing (like on flo): Repeat the same for gralloc and copybit
Scream on the IRC if this worked for you
If strace indicates something like:

"Waiting for service display.qservice..."
This error is known only on cm-10.1 base, and will be upstreamed to mer-hybris soon, but we need more tests: applyhttps://github.com/mer-hybris/android_frameworks_native/commit/6ed4a6e834f6c71b2b6bd8ae1134f50b060e70be to this line https://github.com/CyanogenMod/android_frameworks_base/blob/cm-10.1/cmds/servicemanager/service_manager.c#L88 and also apply https://github.com/mer-hybris/android_system_core/commit/34ea48fd3ad7bf47ec0d0524d76bd20e62717773
open("/sys/kernel/debug/tracing/trace_marker", O_WRONLY|O_LARGEFILE) = 
disable debugfs by: https://github.com/mer-hybris/droid-hal-device/commit/8d437fc6f215081d4e1d2baaa6ac23bb94f73154
if it still crashes on gralloc or other gpu related bits, refer to WIP: https://wiki.merproject.org/wiki/Adaptations/libhybris/gpu
 
Devices with Mali GPU
Add this to $ANDROID_ROOT/rpm/droid-hal-$DEVICE.spec before the last line (do not change the last line, ever)
%define android_config \
#define MALI_QUIRKS 1\
%{nil}
Rebuild droid-hal and libhybris packages

-------------------------------------------------------------------------------------------------------------

#### Skip tutorial:
Congratulations if you have got gui working. During the debugging process you will be building and flashing quite a few times, in which tutorial during the setup screen can be annoying. You can skip that by tapping on the each corner of the screen clockwise, while starting from left-top corner.


 -------------------------------------------------------------------------------------------------------------

#### SIM card not detected:
This often causes a bootloop
Cellular Modem bringup is now in HADK v1.1.1 section 13.3
Additional checks:
Replicate /dev/block structure from Android as closely as possible (for rild to be able to access the modem partition)
Run ls -lR /dev/block in CM
Run ls -lR /dev/block in Sailfish OS
diff the two outputs (this is WIP - android's toolbox ls might need more parameters to produce a comparable output)
If you see differences you need to add custom udev rules to create the correct /dev/block structure
(added automatically since 2016-12-10) For devices with /dev/block/platform/msm_sdcc.1/by-name/ paths (msm_sdcc.1 can be different) add to $ANDROID_ROOT/rpm/ these paths and files with contents, and it most probably will help (but still paste your diff to the IRC channel):

https://github.com/mer-hybris-kis3/droid-config-kis3/blob/master/sparse/lib/udev/platform-device
https://github.com/mer-hybris-kis3/droid-config-kis3/blob/master/sparse/lib/udev/rules.d/998-droid-system.rules

(added automatically since 2017-06-03) Some devices (at least all hybris-13.0 based ports) have /dev/block/bootdevice/by-name/ as /dev/block structure in CM in which case you need to add the following line to the end of the 998-droid-system.rules file in the last link:

    ENV{ID_PART_ENTRY_SCHEME}=="gpt", ENV{ID_PART_ENTRY_NAME}=="?*", IMPORT{program}="/bin/sh /lib/udev/platform-device $env{DEVPATH}", SYMLINK+="block/bootdevice/by-name/$env{ID_PART_ENTRY_NAME}"

If you have logcat and journal error messages suggesting that RIL/ofono can't power the modem on and you have a qcom chipset, have a look in your init.qcom.rc for lines that power it on when the boot animation (bootanim) stops. If you have those, try this (paths may need correcting): https://github.com/stephgosling/android_device_htc_m7-common/commit/9f4abdca65356090e6dd6f0356c5cf4a1870aa5f (note the typo there in the chown line!)

If you have pil-q6v5-mss fc880000.qcom,mss: modem: Failed to locate modem.mdt in your dmesg then try this steps:
Mask firmware.mount

add this service to /lib/systemd/system/ https://pastebin.com/9tbUtVnC

create symlink to that service in /lib/systemd/system/local-fs.target.wants/ 

add /usr/bin/droid/extract_firmware.sh with this content https://pastebin.com/bgphKn4z
 
-------------------------------------------------------------------------------------------------------------
 
#### Ofono problems: 
RILD is running but ofono does not work
If ofono is not working properly and shows "ERROR! Can't connect to RILD: No such file or directory" in logs, edit /etc/ofono/ril_subscription.conf to contain

    [ril_0]
    name=RIL1
    socket=/dev/socket/rild
if your device is dual SIM, add also these lines (don't add them otherwise!):

    [ril_1]
    name=RIL2
    socket=/dev/socket/rild2

If it works add your ril_subscription.conf to the droid-config-$DEVICE like done here https://github.com/Nokius/droid-config-find5/commit/3e3e636e7e3973f9102ebca9dea79794c00c9174
Also add the jolla-settings-networking-multisim to patterns like done here https://github.com/mlehtima/droid-config-fp2-sibon/blob/master/patterns/jolla-configuration-fp2-sibon.yaml#L15
Fix remembering manual access point configurations across reboots run the following command before building the image
sed -i "/begin 60_ssu/a chown -R radio:radio /var/lib/ofono" Jolla-@RELEASE@-$DEVICE-@ARCH@.ks

Devices without modem
File /etc/ofono/ril_subscription.conf  should contain

    [Settings]
    EmptyConfig=true

-------------------------------------------------------------------------------------------------------------
 
Phone calls fail but SMS and mobile data work -> HADK v2.0.0
 
-------------------------------------------------------------------------------------------------------------

#### how to bring store to your device:
Your device adaptation should be on Mer OBS (read "Building things on OBS" below)

Do `ssu s`, Device UID should show a unique ID that is:
IMEI for devices with modem, note - your GSM modem should provide a valid IMEI even without an inserted SIM! Always a good cross-check 
that IMEI matches the one on your phone's box or under battery, and in CM/Android
For devices without modem -- WLAN or BT MAC address.

Find another port/phone and prove that unique ID there is different than yours, and that all of them persist across reboots.
If unique ID is OK then ping pketo on #sailfishos-porters with "Device model" line from `ssu s` to enable store for you.

-------------------------------------------------------------------------------------------------------------
 
gstreamer-1.0 support (video and camera): -> HADK v2.0.0

-------------------------------------------------------------------------------------------------------------
 
#### Waiting for service SurfaceFlinger seen in logcat? then read this (read surfaceflinger-hack below)
If you are using a service like lipstick-hack which loads surfaceflinger for a few seconds to init the hardware, then this stops servicemanager seeing minisfservice as SurfaceFlinger becuase it uses the same service name, then dies.
Disable/mask lipstick-hack from being started by systemd
Add lipstickhack to /init.rc and start in the the core class
```
service lipstickhack /usr/bin/droid/lipstick-hack.sh
class core
user system
oneshot
group graphics drmrpc
```
 -------------------------------------------------------------------------------------------------------------

 #### Alternative to lipstick-hack (aka surfaceflinger-hack):

For devices that need the so called lipstick-hack to start surfaceflinger for a few seconds to init the hardware we developed an alternative as the lipstick-hack has proven to be rather unstable during system boots.

This alternative uses a modified version of libsurfaceflinger itself to init the display and then exits it's main run loop and afterwards behaves like minisfservice. This eliminates the timing issues of lipstick-hack.

Implementation:
1. BoardConfig.mk
- in the BoardCommon.mk of your device add the following lines:
```
# SurfaceFlinger init
BOARD_USE_MOTO_SF = true
```
2. libsurfaceflinger
- apply the patches from https://github.com/guhl/android_frameworks_native/commit/ead91374111114fded280abe56484523355ee2cc to $ANDROID_ROOT/frameworks/native/services/surfaceflinger
- in HABUILD_SDK build libsurfaceflinger by doing:
```
 source build/envsetup.sh
 breakfast $DEVICE
 make libsurfaceflinger
 ```

3. 
- apply the patches from https://github.com/guhl/droidmedia/commit/cf176cd1bec2c0e5b633b8d728528edc6133ed7d to $ANDROID_ROOT/external/droidmedia
- build droidmedia as described in HADK v2.0.1 - chapter 13.2

4. init.rc
- add minisfservice as a service to your $ANDROID_ROOT/system/core/rootdir/init.rc like this:
```
service minisf /usr/libexec/droid-hybris/system/bin/minisfservice
    setenv LD_PRELOAD /usr/libexec/droid-hybris/system/lib/libsurfaceflinger.so
    class main
    user system
    group graphics
```

5. rebuild hybris-hal
as described in HADK v2.0.0 - chapter 5.4 and afterwards package Droid HAL and build the Root Filesystem

-------------------------------------------------------------------------------------------------------------
 
#### hls streams and other codecs:
```
devel-su
#not needed anymore: ssu ar common http://repo.merproject.org/obs/nemo:/devel:/hw:/common/sailfish_latest_armv7hl/
#not needed anymore: zypper ref common
#not needed anymore: zypper dup --from common # safe to do full upgrade/downgrade there
ssu ar experimental http://repo.merproject.org/obs/nemo:/devel:/hw:/experimental/sailfish_latest_armv7hl/
zypper ref experimental
zypper dup --from experimental
```
 
-------------------------------------------------------------------------------------------------------------
 
#### monitoring udev events:
udevadm monitor is your friend.

To get it for cyanogenmod, add this repository https://github.com/chombourger/android-udev/ to your manifest as external/usb and make udevadm

To monitor boot-time events, compile the kernel with CONFIG_DEBUG_KOBJECT=y and increase the log buffer size by setting the kernel command line option: log_buf_len=21 (or higher)
 
-------------------------------------------------------------------------------------------------------------
 
#### persistent journalctl:
modify /etc/systemd/journald.conf :

    Storage=volatile --> Storage=automatic

Then do:
```
mkdir /var/log/journal
reboot
```
 
Systemd suppresses journal, and some valuable info might get hidden. To prevent this, set

    RateLimitInterval=0
 
-------------------------------------------------------------------------------------------------------------
 
#### Nothing provides /system/bin/sh:
Add this to your .spec

    %define __provides_exclude_from ^/system/.*$
    %define __requires_exclude ^/system/bin/.*$
    %define __find_provides %{nil}
    %define __find_requires %{nil}
 
-------------------------------------------------------------------------------------------------------------
 
to change pixel ratio on a running device, as user:
```
devel-su dconf update
# PIXEL_RATIO should be close to the value of horizontal_display_resolution/540
# e.g. Nexus 7 (800 x 1280) displays the pixel ratio is 800/540~=1.48
# always round the value up with two decimal precision
PIXEL_RATIO=1.48

# UPDATE! Please test the new formula for pixel ratio calculation:
# diagonal_display_size_inches/4.5 * horizontal_display_resolution/540
# and feedback the outcome to sledges via IRC (better/worse/closer via own trial&error picks?)
# Yet another formula: YourDevicePPI/sbjPPI (245), e.g. OnePlusX PPI 441/245 = 1.8
# Available ICON_RES values are 1.0, 1.25, 1.5, 1.75, and 2.0. Choose the closest one to PIXEL_RATIO:
ICON_RES=1.5
devel-su zypper in jolla-ambient-z$ICON_RES ambient-icons-closed-z$ICON_RES
dconf write /desktop/sailfish/silica/theme_pixel_ratio $PIXEL_RATIO
dconf write /desktop/sailfish/silica/theme_icon_subdir $
# check that everything worked:
dconf read /desktop/sailfish/silica/theme_pixel_ratio
devel-su reboot
# PIXEL_RATIO and ICON_RES are subjects to fine tuning: https://bugs.nemomobile.org/show_bug.cgi?id=814#c1
```
Script to scale your icons https://pastebin.com/mxKRkt7Z

-------------------------------------------------------------------------------------------------------------
 
existence_error (yes, you read that right) when locally building policy-settings-common: 
You get:
ERROR: error(existence_error(procedure, qsave_program/2), context(precompile/0, _G669))
Solution:
```
sb2 -t $VENDOR-$DEVICE-armv7hl -R -msdk-install
cd /usr/lib/swipl-5.6.50/library
rm INDEX.pl
zypper in fakeroot
fakeroot swipl -g true -t 'make_library_index(.)'
```
then rebuild the package again with mb2
 
-------------------------------------------------------------------------------------------------------------
 
rpm/dhd/helpers/build_packages.sh fails building libhybris, ...

    HOST$
    cd $HOME
    sudo mkdir -p $MER_ROOT/devel
    sudo chown -R $USER mer/devel
run the script again 
 
-------------------------------------------------------------------------------------------------------------
 
make[3]: *** [security/commoncap.o] Error 1...
Those errors appears because ANDROID_CONFIG_PARANOID_NETWORK is disabled in your kernel and with it enabled, you can't access internet with Sailfish OS. ( Since hybris-12.1, rild does not work without ANDROID_CONFIG_PARANOID_NETWORK. Add nemo to group inet if it is enabled.)
Check http://forum.xda-developers.com/showpost.php?p=42880275&postcount=104
To resolve this replace in <path of your kernel>/security/commoncap.c :
``` 
if (cap == CAP_NET_RAW && in_egroup_p(AID_NET_RAW))
        return 0;
if (cap == CAP_NET_ADMIN && in_egroup_p(AID_NET_ADMIN))
        return 0;
``` 
With this:
``` 
#ifdef CONFIG_ANDROID_PARANOID_NETWORK
        if (cap == CAP_NET_RAW && in_egroup_p(AID_NET_RAW))
                return 0;
        if (cap == CAP_NET_ADMIN && in_egroup_p(AID_NET_ADMIN))
                return 0;
#endif
```
 
Save the file and recompile the kernel
 
-------------------------------------------------------------------------------------------------------------
 
Failed at step OOM_ADJUST spawning /usr/libexec/mapplauncherd/booster-qt5: Permission denied
Causes for example the failure of startup wizard on first boot
try to revert kernel change in fs/proc/base.c
https://github.com/mer-hybris/android_kernel_oneplus_msm8974/commit/0ed87d7f3cf7d3388f09bd264a856ad9efc564a3
ping on the IRC if this worked for you :)
 
-------------------------------------------------------------------------------------------------------------
 
UI is shown in tablet mode

needed anymore

(this fix will not work when the display has a super high resolution)
Symptoms: event view has two columns, very large icons in app grid
Check if the screen size is recognised correctly
 
journalctl --no-pager | grep QSizeF
 
If the values are not realistic set the screen size in your droid-hal-device.conf
(only works since Sailfish OS 2.0.1)
QT_QPA_EGLFS_PHYSICAL_WIDTH=<in mm>
QT_QPA_EGLFS_PHYSICAL_HEIGHT=<in mm>
 
-------------------------------------------------------------------------------------------------------------

#### Building things on OBS:
Benefits: automated builds, Jolla Store (see above), OTA (see below); local PC is then only needed for Android, dhd, and droidmedia building (which barely happen when port becomes stable), and mic image creation

It makes sense to go OBS as soon as you have polished your code, minimised hacks, and pushed it to github (usuall when display+touch+WLAN and maybe cellular are working)

On IRC ask sledges to create project and get maintainership for your nemo:devel:hw:$VENDOR:$DEVICE (you can try things out in your home repo first)
```
Click on Repositories tab in your nemo:devel:hw:$VENDOR:$DEVICE
Then "Add repositories"
Check "SailfishOS latest"
Click "Add selected repositories" at the bottom of the page
Add a hw:devel:common repo to build against (which contains all important backports for all ports:), you'll need to add it as an additional repo:
Click on Repositories tab in your nemo:devel:hw:$VENDOR:$DEVICE
Click "Edit repository"
Click Add additional path to this repository
Project:    nemo:devel:hw:common
Repository: sailfish_latest_armv7hl
```
Check how other devices are built here e.g. https://build.merproject.org/project/show/nemo:devel:hw:semc:iyokan

Create droid-hal-$DEVICE package manually and upload RPMs for droid-hal-device and droidmedia (and audioflingerglue if device needs it)
For all other packages create webhooks and trigger builds

How to create webhooks: https://wiki.merproject.org/wiki/Packaging/webhooks

Which webhooks will you need for your device: https://webhook.merproject.org/webhook (search for nemo:devel:hw:lge:mako and replicate that structure)

Add cibot as maintainer, then ask lbt via IRC to "patternise" your nemo:devel:hw:$VENDOR:$DEVICE

Build an image successfully on your PC by following HADK but, using .ks file from droid-config-$DEVICE-ssu-kickstarts-*.rpm built on OBS (don't forget to sed the repos and add nemo:hw:devel:common as adaptation1, this will help you more: http://images.devaamo.fi/sfe/mako/gamma6/Jolla-2.0.1.11-mako-armv7hl.ks )
 
-------------------------------------------------------------------------------------------------------------
 
#### Over-the-Air updates (OTA):
Prerequisities:

Your port has stabilised and is ready to face the big public (gets our retweets, you create Sailfish OS port thread on e.g. XDA, evangelise it :)

Good measure is to have bare necessities of a daily-driver for most people: LED, audio, texts, calls, data, WLAN, GPS, camera, light, proximity, accelerometer, vol keys, vibra, power management

You should be building on OBS (guide above)

Then add these two files (change contents apropriately)
https://github.com/mer-hybris/droid-config-hammerhead/blob/master/sparse/var/lib/flash-partition/device-info
Change PART_REAL_1 to match "boot" partition of your device
Change CPUCHECK_STRING to match the Hardware field in /proc/cpuinfo
https://github.com/mer-hybris/droid-config-hammerhead/blob/master/sparse/var/lib/platform-updates/flash-bootimg.sh
 Don't forget to make it executable
Port over to your device this line:
https://github.com/mer-hybris/droid-hal-hammerhead/blob/ca102d255f1b6f274e2768e8cbd4ad9c631890e9/droid-hal-hammerhead.spec#L12
And this commit (only if MultiROM exists or in-the-works for your device):
https://github.com/mer-hybris/droid-config-hammerhead/commit/cb39670de095b914aea23d6ce0e633d295493016
Don't forget to commit and tag so configs rebuild on OBS :)
Simulate OTA on :devel: https://wiki.merproject.org/wiki/Template:SFOS_OTA , see if all is fine (e.g. you can build devel 1.1.9.28 image and OTA it to 2.0)
Then you can test how an updated kernel package flashes itself automatically with an extra reboot, by making some change in kernel, reuploading RPMs and simulating OTA again
For your users to actually use OTA, you should move it to :testing (on IRC ask sledges to create nemo:testing:hw:$VENDOR:$DEVICE), to still be able to play (i.e. break things) in your :devel

Get maintainership on that :testing repo
Add cibot as maintainer, then via IRC ask lbt to "patternise" that repo too
Click on Repositories tab in your nemo:testing:hw:$VENDOR:$DEVICE
Then "Add repositories"
Then "pick one via advanced interface"
Start typing "sailfishos", then pick the version you want OTA to be available for in format "sailfishos:X.Y.Z.W"
Choose "latest_$PORT_ARCH" for your architecture
Make the "Name" to match exactly "sailfishos_X.Y.Z.W"
Add nemo:testing:hw:common to that as additional repo just like you did with :devel: above

Ensure NO webhooks point to :testing ! Cross-check with https://webhook.merproject.org/webhook

Promote by using osc copypac to all your device packages from devel to testing (useful script: http://pastebin.com/GssLRr8e )(How To https://gist.github.com/taaem/53ed3a99893d323d7ab3bd8d07540f50 ) - use this (or simpler "Submit Package" WebUI option) also in future whenever a HW adaptation package needs updating in between sfos releases (PR is being prepared to add device hw version to zip filename, HW Adaptation version is also shown in About Product, and is incremented by 1 each time OBS automatically rebuilds droid-hal-version-$DEVICE whenever any hw package changes ;))

Make an image with adaptation-community repo pointing to testing, adaptation-community-common pointing to common in your .ks file, and start distributing that to the rest of the world
Don't forget to document everything, create a nice installation wiki article for your device (if not yet already), and add such section: https://wiki.merproject.org/wiki/index.php?title=Adaptations/libhybris/Install_SailfishOS_for_mako&action=edit&section=4
Point your existing users to the OTA section of your device's wiki

Once the next Sailfish OS release comes out and your port adopts it, you can create a new repository in OBS with that version and your users will OTA to it.
 
-------------------------------------------------------------------------------------------------------------
 
#### Access Android's virtual SD card (needs more massaging)
Has received mixed feedback of working/not-working. Replicate onto your device accordingly:
https://github.com/mer-hybris/droid-hal-hammerhead/commit/ca102d255f1b6f274e2768e8cbd4ad9c631890e9
https://github.com/mer-hybris/droid-config-hammerhead/blob/master/sparse/usr/bin/droid/android-links.sh
https://github.com/mer-hybris/droid-config-hammerhead/commit/e15591b98380c95e5be96bf9f386278b9825b5f3
 
-------------------------------------------------------------------------------------------------------------
 
#### mic fails during the run
 
If you get error like this
Warning: repo problem: pattern:jolla-hw-adaptation-$DEVICE1-1.noarch requires droid-config-$DEVICE-policy-settings, but this requirement cannot be provided, uninstallable providers: droid-config-$DEVICE-policy-settings-1-1.armv7hl[adaptation0-$DEVICE-2.0.1.7]
 
Or for example
No provider of 'pkgconfig(qofonoext)' found.
 
In these cases the missing dependencies can be added with the command:
 
    MER_SDK $
    sb2 -t $VENDOR-$DEVICE-$PORT_ARCH -R -m sdk-install ssu ar common http://repo.merproject.org/obs/nemo:/devel:/hw:/common/sailfish_latest_armv7hl/
 
Also add the same repo to .ks file before building the installation package with mic
Run the following commands before chapter 8.3 of HADK pdf
 
    MER_SDK $
    cd $ANDROID_ROOT
    MOBS_URI="http://repo.merproject.org/obs"
    HA_REPO="repo --name=adaptation0-$DEVICE-@RELEASE@"
    HA_REPO1="repo --name=common --baseurl=$MOBS_URI/nemo:/devel:/hw:/common/sailfish_latest_@ARCH@/"
    sed -i -e "/^$HA_REPO.*$/a$HA_REPO1" tmp/Jolla-@RELEASE@-$DEVICE-@ARCH@.ks

If MIC fails with 
RuntimeError: Invalid runmode: native 
remove `--runtime=native` from mic command args

If the error message ends with 
CreatorError: <creator>Unable to find pattern: Jolla Configuration $DEVICE
then make sure that you executed 8.4 in the hadk pdf (as of v1.9.9). If it still does not work, try executing it again, then process patterns and build again.
If the ks file generated in $ANDROID_ROOT does not contain the local repo, then add it manually. To the top of the list of repos, add
"repo --name=adaptation-community-$DEVICE-@RELEASE@ --baseurl=file:$ANDROID_ROOT/droid-local-repo/$DEVICE/" 
substitute $DEVICE and $ANDROID_ROOT appropriately. 
Process patterns and build again.
 
----------------------------------------------------------------------------------------------------------

 #### ff-memless haptics 
 
To use memless haptics driver instead of droid-vibrator, you need a kernel haptics driver that supports a memless interface (evdev). This is briefly explained in HADK pdf chapter 13.1.
Reference kernel driver implementation for qpnp vibrator is here;
https://github.com/kimmoli/android_kernel_oneplus_msm8974/pull/1

It needs also vibrator configuration files if defaults are not ok; (this is also in HADK)
https://github.com/kimmoli/droid-config-onyx/commit/dac479716a6b4300be3c5875982265f6914bb498

And depends which evdev index the new ffmemless gets, one might need to change lipstick config;
https://github.com/kimmoli/droid-config-onyx/pull/4/commits/73bb85fcdc5e2627a8cb0cea0fb5fc2ca9d8e814

in droid-hal-version-$DEVICE.spec comment `%define have_vibrator 0` out and add `%define have_ffmemless 1`

add build of qt5-feedback-haptics-ffmemless in build_packages.sh, and comment out other vibrator packages;

    buildmw "https://git.merproject.org/mer-core/qt-mobility-haptics-ffmemless.git" rpm/qt5-feedback-haptics-ffmemless.spec || die
 
----------------------------------------------------------------------------------------------------------

#### perf :)

    MER_SDK $
    cd $ANDROID_ROOT
    mkdir -p perf/rpm
    cd perf
    ln -s $ANDROID_ROOT/kernel/$VENDOR/$DEVICE linux
    curl -o rpm/perf.spec http://pastebin.com/raw/QiW7FD02
    
 Replace string <YOUR_KERNEL_VERSION> in rpm/perf.spec with kernel version for which you're building perf (for example: 3.4.0)
 
    mb2 -s rpm/perf.spec -t $VENDOR-$DEVICE-armv7hl build
    mv RPMS/*.rpm $ANDROID_ROOT/droid-local-repo/$DEVICE/
    createrepo $ANDROID_ROOT/droid-local-repo/$DEVICE
 
"less" package is needed for perf to format its output. You can find it here: http://repo.merproject.org/obs/nemo:/testing:/hw:/common/sailfish_latest_armv7hl/ 
 
----------------------------------------------------------------------------------------------------------

#### FM Radio support

Needs a device with suitable FM radio hardware and a kernel defconfig containing CONFIG_RADIO_IRIS=y and CONFIG_RADIO_IRIS_TRANSPORT=m (or =y)
If your CONFIG_RADIO_IRIS_TRANSPORT is built-in then this is not needed, however if you have problems try building CONFIG_RADIO_IRIS_TRANSPORT as a module: add (adapt to fit your device if needed) 
- https://github.com/mlehtima/droid-config-fp2-sibon/blob/master/sparse/lib/systemd/system/droid-fm-up.service
- https://github.com/mlehtima/droid-config-fp2-sibon/blob/master/sparse/lib/systemd/system/bluetooth.service.wants/droid-fm-up.service

Sometimes device permissions are wrong (root owner), in this case add https://github.com/mlehtima/droid-config-fp2-sibon/blob/master/sparse/lib/udev/rules.d/999-droid-fm.rules to your droid-configs repo (or directly to device for testing)

Add qt5-qtmultimedia-plugin-mediaservice-irisradio to patterns (or install directly to device for testing)

Add https://github.com/mlehtima/droid-config-fp2-sibon/blob/master/sparse/etc/pulse/xpolicy.conf.d/fmradio.conf to your droid-configs repo (or directly to device for testing)

(pre-2.0.2) Update packages from http://repo.merproject.org/obs/nemo:/devel:/hw:/common/sailfish_latest_armv7hl/ (for building new images add this to your .ks file as described elsewhere in FAQ)

Starting from Sailfish OS 2.0.2 FM radio Media app plugin jolla-mediaplayer-radio can be added to patterns.

(pre-2.0.2) For FM radio testing harbour-piratefm can be obtained from http://repo.merproject.org/obs/home:/kimmoli/sailfish_latest_armv7hl/

--------------------------------------------------------------------------------------------------------------

#### Bluetooth for Qualcomm devices

Enable CONFIG_BT_HCISMD in the kernel defconfig. If it is not present in your kernel, then make these changes (https://github.com/adeen-s/android_kernel_cyanogen_msm8916/commit/4627f4f6f5d886433ff1f9639dc18fe8a006fd00 )
Add these files to sparse (or directly to device) and modify them as needed for your device -->
https://github.com/adeen-s/droid-config-wt88047/blob/master/sparse/usr/bin/droid/droid-hcismd-up.sh
https://github.com/adeen-s/droid-config-wt88047/blob/master/sparse/lib/systemd/system/droid-hcismd-up.service
https://github.com/adeen-s/droid-config-wt88047/blob/master/sparse/lib/systemd/system/bluetooth.service.wants/droid-hcismd-up.service
Bluetooth Should now work. If it doesn't then make sure the permissions are set correctly and all paths mentioned in above files point to valid locations.
If you are still having trouble, check to see if there is a service that configures bluetooth and disable/comment it.  Eg, config_bluetooth in init.qcom.rc
 
--------------------------------------------------------------------------------------------------------------

#### Bluetooth for Broadcomm devices
Enable CONFIG_BT_HCIUART_H4 in the kernel defconfig. These devices typically are attached on high speed uart to something like /dev/ttyHS0
Symlink your firmware file to /etc/firmware. 
eg. https://github.com/r0kk3rz/droid-config-scorpion_windy/blob/master/sparse/etc/firmware/BCM4350C0.hcd
You need to make sure the firmware symlink filename matches your bluetooth device name, which can be found by stracing hciattach
Build rfkill middleware and add to patterns
rpm/dhd/helpers/build_packages.sh --mw=https://github.com/mer-hybris/bluetooth-rfkill-event --spec=rpm/bluetooth-rfkill-event-hciattach.spec
add configs: https://github.com/mer-hybris/droid-config-f5121/commit/afa01bdf4bdb8a0d16bbd34996ec7cac34bbbc55

--------------------------------------------------------------------------------------------------------------

Error During end of kernel build
``` 
Exception in thread "main" java.lang.NoClassDefFoundError: org/bouncycastle/jce/provider/BouncyCastleProvider
        at java.lang.Class.getDeclaredMethods0(Native Method)
        at java.lang.Class.privateGetDeclaredMethods(Class.java:2531)
        at java.lang.Class.getMethod0(Class.java:2774)
        at java.lang.Class.getMethod(Class.java:1663)
        at sun.launcher.LauncherHelper.getMainMethod(LauncherHelper.java:494)
        at sun.launcher.LauncherHelper.checkAndLoadMain(LauncherHelper.java:486)
Caused by: java.lang.ClassNotFoundException: org.bouncycastle.jce.provider.BouncyCastleProvider
        at java.net.URLClassLoader$1.run(URLClassLoader.java:366)
        at java.net.URLClassLoader$1.run(URLClassLoader.java:355)
        at java.security.AccessController.doPrivileged(Native Method)
        at java.net.URLClassLoader.findClass(URLClassLoader.java:354)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:425)
        at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:308)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:358)
        ... 6 more
```

Ensure that unzip is installed as its required for BouncyCastle compile

run something like this in $ANDROID_ROOT outside HABUILD_SDK

    java -jar \
    /home/$USER/mer/android/droid/out/host/linux-x86/framework/dumpkey.jar \
    build/target/product/security/testkey.x509.pem \
    build/target/product/security/cm.x509.pem \
    build/target/product/security/cm-devkey.x509.pem > /home/$USER/mer/android/droid/out/target/product/$DEVICE/obj/PACKAGING/ota_keys_intermediates/keys

--------------------------------------------------------------------------------------------------------------

#### Flashlight shortcut

Starting from Sailfish 2.0.2 it's possible to have flashlight shortcut in eventsview. If your device supports flash torch mode add jolla-settings-system-flashlight package to patterns in your droid-configs repo. The shortcut can be enabled in the eventsview settings.

--------------------------------------------------------------------------------------------------------------

Chapter six

SETTING UP SCRATCHBOX2 TARGET

Lately Jolla offers two sb2 targets so the HADK instructions create some strange behaviour when downloading the sb2 target

You have two options now to replace this line

TARBALL=$(curl $TARBALL_URL | grep "$PORT_ARCH.tar.bz2" | cut -d\" -f4)

TARBALL=$(curl $TARBALL_URL | grep "$PORT_ARCH.tar.bz2" | cut -d\" -f4 | tail -n1)
will give you the sb2 for the latest Early Access 

TARBALL=$(curl $TARBALL_URL | grep "$PORT_ARCH.tar.bz2" | cut -d\" -f4 | head -n1)

will give you the sb2 for latest non EA SFOS Version 

--------------------------------------------------------------------------------------------------------------

#### Fix remembering Bluetooth state on reboot

Add this https://github.com/mlehtima/droid-config-fp2-sibon/commit/265310c24e254ba102211b6ea398f9ef2b68d523

-----------------------------------------------------------------------------------------------------------------

qemu gives segmentation fault error in Ubuntu 16.10, instead, use Ubuntu 16.04.1 LTS, or earlier versions.how 

----------------------------------------------------------------------------------------------------------------

If the pm-service complains about no permissions its because PARANOID_NETWORK is required for your kernel config

----------------------------------------------------------------------------------------------------------------

when building for 2.1 and qt5-qpa-hwcomposer-plugin fails with the error "pkgconfig(Qt5PlatformSupport)"  update dhd submodule

----------------------------------------------------------------------------------------------------------------

Building geoclue-providers-hybris fails

Building geoclue-providers-hybris fails with the error  locationsettings.h for local builds update dhd submodule and in case of OBS build change the branch to jb36857

----------------------------------------------------------------------------------------------------------------

#### Kernel changes needed for updated systemd in Sailfish 2.1.1.X

Apply this to all devices with 3.4 kernel https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=0640113be25d283e0ff77a9f041e1242182387f0

----------------------------------------------------------------------------------------------------------------

#### Building droid-config fails with: Segmentation fault      (core dumped) /usr/lib/qt5/bin/kmap2qmap .........

try updating the packages in the target with 
sb2 -t $VENDOR-$DEVICE-armv7hl -R -m sdk-install zypper ref
sb2 -t $VENDOR-$DEVICE-armv7hl -R -m sdk-install zypper dup

or

add the file /system/build.prop to the target with the contents 


----------------------------------------------------------------------------------------------------------------

libdsyscalls is cause of segfault after r or minimer

Usually means that in your device repo, its enabling clang somewhere, do a grep and disable clang and rebuild :)

----------------------------------------------------------------------------------------------------------------

#### Build Wlan Driver as Module

Most devices require the wlan driver to be built and loaded as a module during startup
Ensure you have CONFIG_MODULES=y in your kernel config

Find your wifi driver in your kernel config, it should already be set to `y` and have something like WLAN in the name.
Set it to m
eg.
CONFIG_BCMDHD=m
CONFIG_PRIMA_WLAN=m
CONFIG_PRONTO_WLAN=m

Add the wlan-module-load.service to your droid-configs sparse directory
https://github.com/mer-hybris/droid-config-onyx/blob/master/sparse/lib/systemd/system/wlan-module-load.service

and add a symlink to enable to service on startup:
https://github.com/mer-hybris/droid-config-onyx/blob/master/sparse/lib/systemd/system/multi-user.target.wants/wlan-module-load.service

---------------------------------------------------------------------------------------------------------------

#### Audio not routed to headphones:

run evdev_trace from mce-tools package and find /dev/input/eventX that detects headphones connection. It will be the one with SW_HEADPHONE_INSERT*  and SW_MICROPHONE_INSERT* like here:
```
----====( /dev/input/event0 )====----
Name: "sensorprocessor"
ID: bus 0x0, vendor, 0x0, product 0x0, version 0x0
Type 0x00 (EV_SYN)
Type 0x01 (EV_KEY)
         KEY_VOLUMEDOWN KEY_VOLUMEUP KEY_POWER KEY_CAMERA KEY_MEDIA KEY_VOICECOMMAND
Type 0x05 (EV_SW)
         SW_LID SW_HEADPHONE_INSERT* SW_MICROPHONE_INSERT*
```

Add this https://github.com/mlehtima/droid-config-fp2-sibon/blob/master/sparse/etc/ohm/plugins.d/accessories.ini file and replace jack-match and jack-device with values from evdev_trace:
jack-match matches Name: field and
ack-device matches /dev/input/eventX
where X is your device input number

Optional way for devices without headphone connector event device:

If your device doesn't have event device for the headphone jack then it might have a switch in /sys/class/switch/h2w/ or similar path
If the state file in the  /sys/class/switch/h2w/ or similar path reacts to headphone connection by changing the value it can be used for headphone detection
Add file /etc/ohm/plugins.d/accessories.ini with the following content (replace switch name with the name found in the path on your device)

    model = uevent
    switch = h2w

If the headphone detection works then add the file to your config repo sparse for future builds

------------------------------------
#### Notes on LOS14.1 Porting
XXX: remove the word "exec" from the last line of /usr/bin/droid/droid-hal-startup.sh, to make this permanent add a modified copy of the file to $ANDROID_ROOT/hybris/droid-configs/sparse/usr/bin/droid/droid-hal-startup.sh (this has to be fixed properly at some point)

On mixed 32/64bit devices, LD_LIBRARY_PATH could be wrong.  If logcat shows services aborting with SIG 6 due to wrong arch, try removing the LD_LIBRARY_PATH from /init.environ.rc

Run this script in $ANDROID_ROOT http://paste.opensuse.org/40869869

Details of what the script does:
Symlinks for services:

    sh-3.2# ls -lh /usr/libexec/droid-hybris/system/etc/init/  
    total 4.0K 
    lrwxrwxrwx 1 root root   26 Oct  6 20:52 atrace.rc -> /system/etc/init/atrace.rc 
    lrwxrwxrwx 1 root root   28 Oct  6 20:52 bootstat.rc -> /system/etc/init/bootstat.rc 
    lrwxrwxrwx 1 root root   29 Oct  6 20:52 debuggerd.rc -> /system/etc/init/debuggerd.rc 
    lrwxrwxrwx 1 root root   29 Oct  6 20:52 drmserver.rc -> /system/etc/init/drmserver.rc 
    lrwxrwxrwx 1 root root   29 Oct  6 20:52 dumpstate.rc -> /system/etc/init/dumpstate.rc 
    lrwxrwxrwx 1 root root   31 Oct  6 20:52 gatekeeperd.rc -> /system/etc/init/gatekeeperd.rc 
    lrwxrwxrwx 1 root root   30 Oct  6 20:52 init-debug.rc -> /system/etc/init/init-debug.rc 
    lrwxrwxrwx 1 root root   28 Oct  6 20:52 installd.rc -> /system/etc/init/installd.rc 
    lrwxrwxrwx 1 root root   27 Oct  6 20:52 logcatd.rc -> /system/etc/init/logcatd.rc 
    lrwxrwxrwx 1 root root   24 Oct  6 20:52 logd.rc -> /system/etc/init/logd.rc 
    lrwxrwxrwx 1 root root   30 Oct  6 20:52 mediacodec.rc -> /system/etc/init/mediacodec.rc 
    lrwxrwxrwx 1 root root   34 Oct  6 20:52 mediadrmserver.rc -> /system/etc/init/mediadrmserver.rc 
    lrwxrwxrwx 1 root root   34 Oct  6 20:52 mediaextractor.rc -> /system/etc/init/mediaextractor.rc 
    lrwxrwxrwx 1 root root   24 Oct  6 20:52 mtpd.rc -> /system/etc/init/mtpd.rc 
    lrwxrwxrwx 1 root root   29 Oct  6 20:52 perfprofd.rc -> /system/etc/init/perfprofd.rc
    lrwxrwxrwx 1 root root   26 Oct  6 20:52 racoon.rc -> /system/etc/init/racoon.rc 
    lrwxrwxrwx 1 root root   24 Oct  6 20:52 rild.rc -> /system/etc/init/rild.rc
    lrwxrwxrwx 1 root root   29 Oct  6 20:52 superuser.rc -> /system/etc/init/superuser.rc 
    lrwxrwxrwx 1 root root   27 Oct  6 20:52 uncrypt.rc -> /system/etc/init/uncrypt.rc 
    lrwxrwxrwx 1 root root   23 Oct  6 20:52 vdc.rc -> /system/etc/init/vdc.rc
    lrwxrwxrwx 1 root root   23 Oct  6 20:52 vold.rc -> /system/etc/init/vold.rc

NOTE, no audioserver and mediaserver links!
NOTE, bootanim was removed in the updated script, also vold was added

if NINJA builds are not working, export USE_NINJA=false

For actdead charging animation, see changes here https://github.com/kimmoli/sfos-onyx-issues/issues/29 but also add 'trigger late-start' to 'on charging' in init.rc


---------------------------------------------------------------------------------------------------------------

#### hwcomposer fails to run with  `atomic commit failed ret:-22` in dmesg
You may need to add the following commits
https://github.com/zhxt/android_kernel_xiaomi_msm8996/commit/ab8e2349bae3a0971b237b744465089d6f22f8a1
https://github.com/zhxt/android_kernel_xiaomi_msm8996/commit/c48eee07ace04204cf6c670ddfcf8c694fd88db4

--------------------------------------------------------------------------------------------------------------

Devices that have qseecomd usually have issues getting to UI so its best to disable it in the init.$DEVICE.rc

--------------------------------------------------------------------------------------------------------------

#### Updating submodules
Submodule locations:
rpm/dhd
hybris/droid-configs/droid-configs-device
hybris/droid-hal-version-fp2-sibon/droid-hal-version
In the each folder check remote name using git remote -v 
Run (replace remote_name with the name you found out in previous step)
git fetch remote_name
git pull remote_name master

--------------------------------------------------------------------------------------------------------------

#### Issues with ngfd or ngfd-plugin-droid-vibrator or pulseaudio
Update submodules as described above
Downgrade hybris/droid-configs/droid-configs-device by going to the folder and running 

    git reset --hard 769864929261d14ba2380323ddced4e325d5c819
    
Replace `%define have_vibrator 1` in droid-hal-version-@DEVICE@.spec with `%define have_vibrator_native 1`
Change package names in droid-configs patterns as described in templates https://github.com/mer-hybris/droid-hal-configs/commit/aac652aae840a15629c0f4e070275ea128fe088f
Downgrade ngfd plugin:
Go to hybris/mw/ngfd-plugin-droid-vibrator and run:

    git reset --hard 3e2b4fb5b03a6d3db9ca5a41c7091e771f99cc4f

IN PLATFORM_SDK:

     $PLAFORM_SDK
     rpm/dhd/helpers/build_packages.sh -b hybris/mw/ngfd-plugin-droid-vibrator -s rpm/ngfd-plugin-native-vibrator.spec
    
     sb2 -t $VENDOR-$DEVICE-$PORT_ARCH -m sdk-install -R zypper rm ngfd-plugin-droid-vibrator
     rpm/dhd/helpers/build_packages.sh -b hybris/mw/ngfd-plugin-droid-vibrator --spec=rpm/ngfd-plugin-native-vibrator.spec
 
when you run the whole build_packages.sh after this skip the ngfd-plugin-native-vibrator build

--------------------------------------------------------------------------------------------------------------

#### Issues with pulseaudio module build
downgrade hybris/droid-configs/droid-configs-device as described above by going to the folder and running "git reset --hard 769864929261d14ba2380323ddced4e325d5c819"
build_packages.sh --configs

--------------------------------------------------------------------------------------------------------------

#### Determine which is the touch event
use command "getevent" as super user in adb shell. The event which spams most outputs on the screen when the screen is touched is the touch event.

--------------------------------------------------------------------------------------------------------------

#### libminisf.so not found
Add libminisf to droidmedia make command like this in HABUILD_SDK:

    make -jXX libdroidmedia minimediaservice minisfservice libminisf

Also update rpm/dhd submodule in case you have an older version

--------------------------------------------------------------------------------------------------------------

#### No installroot directory after droid-configs build when preparing .ks file
rpm2cpio droid-local-repo/$DEVICE/droid-configs/droid-config-$DEVICE-ssu-kickstarts-1-1.armv7hl.rpm | cpio -idmv
in the sed command use $ANDROID_ROOT/usr/share/kickstarts/$KS instead of $ANDROID_ROOT/hybris/droid-configs/installroot/usr/share/kickstarts/$KS

--------------------------------------------------------------------------------------------------------------

#### Updating local build target
Change release version in the command if needed
In Platform SDK:

    sb2 -t $VENDOR-$DEVICE-$PORT_ARCH -m sdk-install -R ssu release 2.1.4.14
    sb2 -t $VENDOR-$DEVICE-$PORT_ARCH -m sdk-install -R zypper ref
    sb2 -t $VENDOR-$DEVICE-$PORT_ARCH -m sdk-install -R zypper dup

--------------------------------------------------------------------------------------------------------------

#### Using backported Bluetooth drivers in 3.4 kernel for devices with Qualcomm bluetooth chip using hci_smd driver
Generic guide: https://bluez-android.github.io/#building-own-kernel
Sailfish specific guide:
Build your kernel with patches from https://github.com/bluez-android/misc/tree/master/patches-kernel and with following flags defined in defconfig

    CONFIG_BT=m
    CONFIG_CRYPTO_CMAC=y
    CONFIG_CRYPTO_USER_API=y
    CONFIG_CRYPTO_USER_API_HASH=y
    CONFIG_CRYPTO_USER_API_SKCIPHER=y

NOTE: Patches may not be required for >= 3.18
In your local_manifest, add 
<project name="mlehtima/backports-bluetooth" path="external/backports-bluetooth" revision="master" />
run repo sync in HABUILD_SDK
Build backported drivers by running make backports in HABUILD_SDK while in $ANDROID_ROOT folder
if you get `"external/backports-bluetooth/drivers/bluetooth/hci_smd.c:35:26: fatal error: mach/msm_smd.h: No such file or directory" `error change 
#include <mach/msm_smd.h> to #include <soc/qcom/smd.h> in that file
IMPORTANT: if you rerun make hybris-hal at any time you will always have to rerun make backports after that
Package droid-hal as usual
Change your config repo to use bluez5 https://github.com/mlehtima/droid-config-fp2-sibon/commit/1cba868fdcfebaffc14a084c5d82fbf2e4339173
Rebuild config rpms and image

--------------------------------------------------------------------------------------------------------------

#### Graphics performance improvements
Test framerate display (can be enabled in Settings->Developer mode) when using some apps like gallery
If the top view is mostly red try to set QPA_HWC_IDLE_TIME=5 in /var/lib/environment/compositor/droid-hal-device.conf
Run systemctl restart user@100000 using devel-su
Test framerate display again and if you see more green than before you should use the value
Different values can be tested but value 5 has been found to be helping on some devices
On some devices also setting QPA_HWC_BUFFER_COUNT=3 in /var/lib/environment/compositor/droid-hal-device.conf helps with graphics performance

--------------------------------------------------------------------------------------------------------------

#### Black gallery pictures and no browser content/browser crash:
Add this to droid-hal .spec file and rebuild droid-hal and libhybris packages (remove the sources from hybris/mw/libhybris to make sure a clean rebuild is done):
%define android_config \
#define WANT_ADRENO_QUIRKS 1\
%{nil}

--------------------------------------------------------------------------------------------------------------

#### Problems with tfa9890:
Copy /system/etc/firmware to /etc/firmware. Symlink or mount doesn't work! (But why?)

--------------------------------------------------------------------------------------------------------------

make[1]: *** No rule to make target `XXX_defconfig'. Stop.
This was seen as an error while making hybris-hal on wingray
Open hadk/device/*/*/BoardConfig.mk and comment out the line "TARGET_KERNEL_SOURCE=XXX"
Re-run source build/envsetup.sh and breakfast $DEVICE
Re-run make -jXX hybris-hal
Kernel should build properly at this point but you may get an error later along the lines of "svn: command not found"
"sudo apt-get install subversion" should fix it
if that doesn't work try un-commenting the"TARGET_KERNEL_SOURCE=XXX" line for what you are doing and then if you have to rebuild kernel again re-comment it


--------------------------------------------------------------------------------------------------------------
#### Anbox Information
https://public.etherpad-mozilla.org/p/anbox-sailfishos

