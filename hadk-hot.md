Please, do not use browser-in-place translation tools (right-click translate-this) or similar. They overwrite the content on this page.
 
 
---------------------------------------------------------------------------------------------------------------------
2018-03-20: libminisf
 
There's a new library to be built by droidmedia - libminisf. It's invoked from the qt-hwcomposer plugin in lipstick to create the MiniSurfaceFlinger service if it's not already running, and start the binder thread pool. The latter is important on some adaptations that create services from inside hwcomposer, that would previously hang when starting recording. Include 'libminisf_32' in your make command to build it. 
 
2018-03-14: Hardware video codecs in Android 7

Droidmedia now supports video decoding in Android 7, but it relies on a fix in frameworks/av that allows hardware codecs to be selected. This bug seems to have survived for a while in Android, perhaps because the things that use it aren't really used there yet. It's confirmed as present in mer-hybris 14.1. Check that you have it in your tree: https://android.googlesource.com/platform/frameworks/av/+/ca4c68f25eb61f4d4e339e99cf63f863adc52fdd%5E!/#F0

2018-03-08: Droidmedia_32

As many adaptations don't contain the psecial Qualcomm BOARD_QTI_CAMERA_32BIT_ONLY variable, there's now a variable to set in OBS to override this: droidmedia_32bit := 1. The old variable will still work for OBS builds too though.
When building locally, add DROIDMEDIA_32 := true to env.mk

2018-02-12: Native droid vibrator backend is now built as the default. Update submodules (if needed check faq how to do it) and replace %define have_vibrator 1 in droid-hal-version-@DEVICE@.spec with %define have_vibrator_native 1 and change package names in droid-configs patterns as described in templates https://github.com/mer-hybris/droid-hal-configs/commit/aac652aae840a15629c0f4e070275ea128fe088f

2018-02-05: Droidmedia Camera HAL forcing

A new .mk file is now present in droidmedia: env.mk. This parametrizes the droidmedia build for device-specific tweaking. It's been introduced because we encountered a device with a camera that wouldn't work properly using the default camera connection, which connects using the highest declared HAL version, and instead requires a forced HAL1 connection. To try this for camera that aren't behaving properly, insert 'FORCE_HAL:=1' into env.mk

2017-12-05: PulseAudio droid modules configuration update

pulseaudio-modules-droid has been changed so that the default startup creates a profile called "default", which has most relevant sinks and sources enabled. Previously there was separate "combine" mode, which tried to achieve something similar, but was a bit flawed in a way since it required more custom changes to other configuration parts.

To be able to classify these automatically created sinks and sources, the sinks and sources are now marked with properties defining their use. For example, primary sink has droid.output.primary = true, etc.

=== What this means to your port ===

* If you have custom (under sparse/) /etc/pulse/xpolicy.conf, /etc/pulse/xpolicy.conf.d/bluez4.conf or/etc/pulse/xpolicy.conf.d/bluez5.conf or have added any files to /etc/pulse/xpolicy.conf.d directory, you may need to adapt those configurations to the new format.

      * If you use droid card profile "primary-primary" etc, change those to read "default"


* If you have modified /etc/pulse/arm_droid_default.pa or added your own configuration, at least make sure that module-droid-card is not loaded with "combine" argument.

* If you are using the defaults, make sure you have commit 91718954ba84c956c0be6232d1148a760ae0505e (PR https://github.com/mer-hybris/droid-hal-configs/pull/114 ) in your d-h-c submodule, and make sure you are using at least the following package versions:

 pulseaudio 8.0+git13
 pulseaudio-modules-nemo 8.0.24
 pulseaudio-policy-enforcement 8.0.33
 pulseaudio-modules-droid 8.0.63

Also check https://github.com/mer-hybris/pulseaudio-modules-droid/blob/master/README if you want to get a bit more indepth description of droid modules' behaviour.

2017-11-20 All minimalhooks/mm64-rpm/n64-rpm libhybris branches have been merged to master; please switch and check if you need additional defines in your dhd .spec: https://github.com/mer-hybris/droid-hal-device/commit/d32f25ba40f1149379b3c5a18c0965d8a54e45b8

2017-11-02: Sailfish X: droid-flashing-tools has now been provided in nemo:devel:hw:common repo, should unbreak mic builds

2017-10-13 Friday! oooOOOOoo

The wiki I believe has a typo (critical one at that), in the flashing the linux way without being able to use EMMA, the mount line for the blob partition is wrong?

mkdir oem
mount /dev/mmcblk0p28 oem
mkdir blobs
mount /dev/mmcblk0p52 system                <---------this should be blobs? no, because we flash .img to system partition temporarily
rm -rf oem/*
cp -ar blobs/* oem
umount oem
umount blobs
rmdir oem blobs
exit

It probably would be a good idea to have full paths in the mounts as well (so /oem or /root/oem or whatever)


Also it appears that the sed that matches the 'too old' line has changed due to working changing in the flash.sh script. The wiki reads:

MATCH='You have too old Sony Android version ($VMAJOR.$VMINOR) on your device,'

whereas my flash.sh reads:

    "Your Sony Android version ($VMAJOR.$VMINOR) on your device is too old," thanks, I've fixed that in wiki, please delete this case once seen

In addition to that I also had to remove the exit 1; line below and for some reason my md5sum failed too (but that was probably due to build error on my part)    


2017-10-10 Sailfish X wiki updated to blobless builds, and blobs are now under /odm (being independent from /system/vendor); you will need a complete rebuild
(https://sailfishos.org/wiki/Sailfish_X_Build_and_Flash )

    This transition is not OTA-compatible (but i haven't heard from you about how our xperia OTAs have been coming along ;P)

    Thanks for bearing with us during crucial times and helping and testing; there shouldn't be more collossal changes thereafter

 
2017-10-06 Sailfish X: bluez5 with patch against blueborne is now in nemo:devel:hw:common

    exempi backported to nemo:devel:hw:common, should fix ambience and gallery edit crashes for 64bit devices

    * all packages from nemo:devel:hw:common get automagically into an image

 
2017-10-03 Camera hangs on CM14, silently waiting for a 'SchedulingPolicyService'. Avoid by adding 
camera.fifo.disable=1
to build.props
 
2017-10-03 IF YOU TAKE IN DROID-CONFIGS NEWER THAN v0.1.1 YOUR SONY WILL GET STUCK IN BOOTLOOP. Countermeasures (either inside a still working Sailfi

    Thanks for bearing with us during crucial times and helping and testing; there shouldn't be more collossal changes thereafter

 
2017-10-06 Sailfish X: bluez5 with patch against blueborne is now in nemo:devel:hw:common

    exempi backported to nemo:devel:hw:common, should fix ambience and gallery edit crashes for 64bit devices

sh OS, or via recovery chroot if you got less lucky):
    Working SFOS:
devel-su
mkdir ~/oem
mount /dev/disk/by-partlabel/oem ~/oem
rm -rf ~/oem/*
cp -ar /system/vendor/* ~/oem
umount ~/oem
rmdir ~/oem

    Recovery:

mkdir oem
mount /dev/mmcblk0p28 oem


mkdir cache
mount /dev/mmcblk0p24 cache
rm -rf oem/*
cp -ar cache/* oem
umount oem
umount cache
rmdir oem cache
 
2017-09-27: sfosx bluetooth pushed out. If you feel adventurous, resync hybris manifest and rebuild, writing up your findings in hot-hadk etherpad ;) ping jusa on how to contrib for those BT profiles/devices that you find not working
 
Double tap to wake has been pushed out, but is known to cause issues on certain xperia x the revisions. try it with `mcetool --set-doubletap-wakeup=proximity`
 T
Browser video playback has been fixed
 
syspart rebuild should get video recording working too
 
2017-09-24 
 
Section 5.2 in the hadk, the local_manifest URL is incorrect, should be https://github.com/mer-hybris/local_manifests Thanks, will be fixed in HADK >2.0.1
 
 
 
2017-09-23: Sailfish X (Xperia X) GPS fixed. To get into existing builds:
    
    # NOTE: some git repos below will be nuked (rm -rf) and re-cloned. If you've done some coding in them, be careful not to lose it
    PLATFORM_SDK $
    
    cd $ANDROID_ROOT
    sdk-assistant remove sony-f5121-armv7hl
    sdk-assistant create sony-f5121-armv7hl http://releases.sailfishos.org/sdk/targets-1707/Jolla-2.1.1.24-Sailfish_SDK_Target-armv7hl.tar.bz2
    rm -rf rpm/
    git clone --recursive https://github.com/mer-hybris/droid-hal-f5121 rpm
    rm -rf hybris/mw/geoclue-providers-hybris/
    
    # Now go through ONLY ONE code snippet right below "Ignore chapter 7, but do the following instead:" in https://sailfishos.org/wiki/Sailfish_X_Build_and_Flash
    # Then jump to "Now go through chapter 8, but instead of the `sudo mic create fs ...` command, perform the following:"
 
 
2017-09-20 Sailfish X HW adaptation opensourced: https://blog.jolla.com/xperiax-open-source-hw-adaptation/
 
2017-09-15 CUSTOMCONTEXT_NO_BGRA has been renamed to QT_OPENGL_NO_BGRA in the latest qtscenegraph-adaptation (master) - all 64bit devices that had black gallery photos (in fullscreen) can now build from master, but rename that env var in your configs
 
2017-09-15 If you have OOM errors doing the make in the syspart portion of sailfishx try adding a swapfile to your build host:
mkdir /root/swap
chmod 700 /root/swap
dd if=/dev/zero of=/root/swap/swapfile bs=1M count=1024 # only 1G swap?  I build on 8GB with 8GB swap, I think having any largish swap may help 
chmod 600 /root/swap/swapfile
mkswap /root/swap/swapfile
swapon /root/swap/swapfile
 
2017-06-29 droid-hal-init has been updated and now shuts down by properly stopping all android services, instead of issuing kill signals. a 'hybris.shutdown' property triggers this, and so needs to be included in /init.rc. See the script droid-hal-shutdown for details: https://github.com/mer-hybris/droid-hal-configs/pull/95
This replaces the old kill-cgroup.sh script, which was a bit heavy handed and failed to shutdown some services.

    the needed changes will soon be included to mer-hybris repos so better wait until that than to patch per device

 
2017-05-10 qtscenegraph-adaptation has a new home, update your dhd submodule: https://github.com/mer-hybris/droid-hal-device/commit/60eaca23a16319e4ccc1a67b50d5b95e8f8febd9
 
2017-02-17 Are you tired of this error in recent HADK:
    File '/repodata/repomd.xml' not found on medium 'http://repo.merproject.org/obs/nemo:/devel:/hw:/$VENDOR:/$DEVICE/sailfish_latest_armv7hl/' Abort, retry, ignore? [a/r/i/?] (a):
Look no more, but update your rpm/dhd submodule and run `rpm/dhd/helpers/fix_dangling_adaptation-community_repo.sh`
 
2017-01-14 Please update your droid-hal-device ($ANDROID_ROOT/rpm/dhd/) submodule, as it contains latest bluez and Android 6 (64+32bit frankenstein) build fixes
 
2016-12-15 Bluez configs were broken in 2.0.5 ports (and process_patterns.sh due to recent build_package.sh refactorings), please now:

    update your droid-hal-configs submodule for the fix: https://github.com/mer-hybris/droid-hal-configs/pull/65

    remove bluez-configs-sailfish (and if you have obexd-configs-sailfish), from your patterns

    add %define community_adaptation 1 to your droid-config-$DEVICE.spec

 
2016-11-09 Jolla now offers two sb2 targets, one for the Early Access and one 'normal'. This is the reason why downloading of the sb2 target ends with bunch of symbols. The hadk-faq shows how to prevent this in line 608.
 
2016-11-09 Since 2.0.4 you will no longer find much in Settings | Accounts :) Fix:
https://github.com/mer-hybris/droid-hal-configs/commit/c13a6e771d96a81b39337f407e9dc06cc3bcc6a8
 
2016-10-19 <krnlyng> hi, there is a small mistake in libhybris which prevents hybris-13 adaptation to work on mako, namely memcpy should always return dest. with this test_hwcomposer, minimer and the language picker (haven't tested more) of sailfishos work on mako
21:36 < krnlyng> here is the patch for anyone interested: https://github.com/krnlyng/libhybris-ghosalmartin/commit/09d9647c1802a89df3e491ba93bf2ade49068deb i have to note that with ubuntus libhybris only test_hwcomposer works with this patch. minimer and the language picker fail to start up, i used a version of libhybris where i ported the mm linker to manually and there all that i mentioned work 21:36 < krnlyng> in particular if src == null, still dest seems to be the expected return value
21:36 < krnlyng> http://imgur.com/a/Gc6ny
21:36 < krnlyng> https://bpaste.net/show/30844512bdeb excerpt from memcpy.S https://github.com/krnlyng/libhybris/tree/mako_13_mmlinker
 
2016-10-15 Latest droid-hal-configs needs:

    "usb-moded-pc-suite-mode-android" removed from your patterns

    backport buteo-syncfw-qt5 0.8.9 to sfos 2.0.4 (hopefully will work for 2.0.2 ports too)

    and if device reboots when you pull USB cable out during MTP transfers, you might need this:

https://github.com/yacuken/android_kernel_oneplus_msm8974/commit/ebf4afca4f247073b88db1f33f1ce47573e563e0
 
2016-10-08 Building qt5-qpa-hwcomposer-plugin is now broken for 2.0.2 and 2.0.4 ports, please use qt-5.2 branch:

    update submodule of your rpm/dhd (droid-hal-device) which fixes build

    point your qt5-qpa-hwcomposer-pluginwebhooks to use qt-5.2: https://webhook.merproject.org/webhook/finger

     

2016-09-18 - 2.0.2.51 available for the MotoG xt1032 by pigz
 
2016-08-12 If a another party hears their own voice echoing during the call, you might need the new 0.0.5.18 libhybris for your port
 
2016-08-09 Sailfish OS 2.0.1.11 alpha2 released by mal- for Fairphone 2 https://twitter.com/sledgeSim/status/763069388014645248https://wiki.merproject.org/wiki/Adaptations/libhybris/Install_SailfishOS_for_fp2
 
2016-08-02 AsteroidOS (based on mer/sailfish) is on Asus Zenwatch 2 \o/ https://twitter.com/AsteroidOS/status/758762130933940224
 
As for ChromeOS, I can't comment much since I haven't used it myself (unlike Sailfish). Uh, what? Sailfish uses all those other Linux projects, yes. Like any other Linux distribution. In other words, Sailfish is a real Linux distribution and that is why it's so nice. I don't care that there's some high-level proprietary theming going on, it's really not relevant to the core functionality of the distribution.
 
2016-08-01 restored FAQ is now here: http://bit.ly/hadk-faq-v2
 
2016-07-18 Sfdroid multiwindow by krnlyng \o/ https://twitter.com/locusf/status/755334989802274817
 
2016-07-17 AsteroidOS on Sony Smartwatch 3 https://twitter.com/AsteroidOS/status/754619821921333248
 
2016-06-26 PSA: New Sailfish 2.0.1.11 alpha4 releases for 2011 Xperias [Arc/Arc S (anzu), Live (coconut), Neo V (haida), Neo (hallon), Pro (iyokan), Mini Pro (mango), Active (satsuma), Mini (smultron) and Ray (ururhi)] are now available at http://images.devaamo.fi/sfe/. Installation instructions at https://wiki.merproject.org/wiki/Adaptations/libhybris/Install_SailfishOS_for_iyokan.
https://twitter.com/sledgeSim/status/747364122803732480
 
2016-06-26 OnePlus X team (kimmoli+taeem) hit beta! \o/ https://twitter.com/sledgeSim/status/747361512277311488
 
2016-06-23 #sfdroid on CM12.1 based port \o/ https://twitter.com/adampigg/status/746091598182776832 
 
2016-06-18 Samsung Galaxy S4 \o/ https://twitter.com/wickwire2099/status/744297149198962688
 
2016-06-16 New hadk, bugfixes only and section 8.2 shows what's not allowed in the image: https://sailfishos.org/hadk
 
2016-05-27 Sony Xperia Z3 Compact: alpha version released on XDA: http://forum.xda-developers.com/z3-compact/devn9ent/rom-sailfish-os-2-0-1-11-community-port-t3386966
 
2016-04-18 OnePlus X alpha-3 https://twitter.com/LiKimmo/status/722188953202384898
 
2016-04-08 OTA and gamma5 for #Nexus4 by ballock \o/ Sound recordin in camera app, dual mic voicecalls https://twitter.com/bolek1337/status/718397905963495424
 
2016-04-06 #sfdroid on Motorola MotoG 2nd Gen. by Mister_Magister! https://twitter.com/sledgeSim/status/717850289953497088
 
2016-04-01 First steps for Sony Xperia Z3 Compact Tablet by Nokius \o/ https://twitter.com/Nokius/status/715645080435077121
 
2016-03-25 OnePlus X alpha-2 https://twitter.com/LiKimmo/status/713095896104312834/photo/1
 
2016-01-11 Xiaomi Redmi 1S revived by Litew \o/ https://twitter.com/tradiz/status/686409340916502529 -> http://tradiz.org/?sfos-armani
 
2016-03-06 22:07 < eMPee584> @all: I will sponsor a i927 phone (full QWERTY keyboard! 2011, 1GB RAM, 8GB EMMC, Dual-core 1.0 GHz Cortex-A9) for the brave soul that wants to have a go at porting mer on it. it's a worthy successor to the N900. CM11 port available, see XDA. If you're interested, drop me a mail mpartap<ät>gmx.net . Best Regards!
 
One more (olden one), you can still try to contact the offerer: 
[BOUNTY] Will pay for a Sailfish OS port to Moto E 2015 LTE (Name your price.) http://forum.xda-developers.com/moto-e-2015/general/bounty-pay-sailfish-os-port-price-t3252278/post63900427#post6390042
 
2016-03-01 Nexus 4 revival \o/ https://twitter.com/bolek1337/status/704725132825202688
 
2016-03-01 Moto G 1st Gen. (falcon) \o/ https://twitter.com/adampigg/status/704786369269334016 (most requested on http://bit.ly/port-requests)
 
2016-02-28 THIS IS OMG! https://twitter.com/LiKimmo/status/703924632341114881
 
2016-02-18 HTC One M6 gets more working things by stephg! https://twitter.com/chucisteph/status/700380393426575360
 
2016-02-18 14:15 < sledges> PSA: RFC: added OBS and OTA tutorial draft from line 340 http://piratepad.net/hadk-faq
 
2016-02-13 11:36 < r0kk3rz> Ok, any wiki editors, if you look at the first few rows of the glorious HADK table https://wiki.merproject.org/wiki/Adaptations/libhybris you will see a couple of new ways of having the device row. One is to create a device_codename template page like the one i made for the FP2 https://wiki.merproject.org/wiki/Template:Device_fp2, or if you look a bit further down the table you can just use a template to create the row
This enables device hw support status to be shown with a link to its template, e.g.: https://wiki.merproject.org/wiki/Adaptations/libhybris/Install_SailfishOS_for_hammerhead#Hardware_Support\o/
Credits go to Nokius, lbt, and r0kk3rz for this amazing templatism work :)
 
2016-02http://piratepad.net/port-hot-01 http://bit.ly/port-requests 
 
2016-01-24 HTC One \o/ two-porter effort https://twitter.com/rltyseven/status/691232410826993664
 
2016-01-22 OnePlusX \o/ https://twitter.com/taaeem_/status/690475762609168384
 
2016-01-16 OnePlus one calls+SMS \o/ https://twitter.com/vgrade/status/688170150525190145
 
2016-01-10 Samsung Galaxy Tab 2 - p3100
https://twitter.com/AdeenShukla/status/684314355031117824
 
2015-12-26 Motorola Photon Q \o/ https://twitter.com/VEvgeniev/status/680844603130511360
 
2015-12-23 Android as Sailfish OS App! https://twitter.com/sledgeSim/status/679647138305970177
 
2015-12-23 Oppo Find 7s \o/ https://twitter.com/Nokius/status/679573564845101056
 
2015-12-21 Fairphone 2 status \o/ https://wiki.merproject.org/wiki/Adaptations/libhybris
 
2015-12-20 Samsung Galaxy Note \o/ https://twitter.com/adampigg/status/6511516216369152
 
2015-12-08 OnePlus One SFOS 2.0 teaser by smoku https://twitter.com/sledgeSim/status/674020668883734530
 
2015-11-18 mako 2.0.0.10, but no camera yet;) : https://twitter.com/_undressed_/status/666943740263718912
 
2015-11-18 Alpha3 releases of Sailfish OS 2.0.0.10 for Xperia Pro (iyokan), Mini (smultron), Mini Pro (mango), Ray (urushi) and Active (satsuma) now available at http://images.devaamo.fi/sfe, for a partial list of bug fixes see http://forum.xda-developers.com/jolla-sailfish/general/request-port-sailfish-os-to-xperia-2011-t2171283/post63885492#post63885492
 
2015-11-03 to make call recording work, add this to your patterns/device: 
https://github.com/mer-hybris/droid-config-hammerhead/commit/7246d1e8dbb6fac7de79d78c2557ffda609ed4b2
 
2015-10-02 API CHANGE: since 1.1.9 all cellular devices must add this to their patterns:
- pattern:jolla-sailfish-cellular-apps

    - telepathy-ring

 
2015-09-24 PSA: API CHANGE: when you'll decide to update to >=libhybris-0.0.5.12, you'll need to update your $ANDROID_ROOT/hybris/droid-config-$DEVICE/droid-configs-device submodule and regenerate patterns
 
2015-09-22 Alpha2 releases for iyokan (Xperia Pro), mango (Xperia Mini Pro), smultron (Xperia Mini) and urushi (Xperia Ray) are now available at http://images.devaamo.fi/sfe/
 
2015-09-16 Chromium-based browser for Sailfish OS by Tworaz \o/ Check: http://piratepad.net/sailfish-quicksilver
 
2015-09-15 ZTE Open C / Kis 3. :D https://twitter.com/konstatuomio/status/643802228789235716
 
2015-09-14 PSA: new hadk is out! https://sailfishos.org/hadk
 
2015-09-14 API CHANGE! Starting 1.1.9, every port must have this for sensors to work: https://github.com/mer-hybris/droid-config-hammerhead/commit/9dfbac7a4520ceaf1e7755e3b7a86bc00f7ecdd1
 
2015-09-13 Sony Xperia Ray - first blind port ever! https://twitter.com/sledgeSim/status/643037489914097665
 
2015-09-10 PSA: latest dhc contains cleaned up "sailfish-porter-tools" pattern, adjust your hybris/droid-configs/patterns/jolla-*-$DEVICE.yaml like this: https://github.com/mer-hybris/droid-hal-configs/commit/488c9da91e2fda5d9fa26826360dc8cfde4dab87#diff-4ecd3b609a20b29fa8e103412b553279R5
for this you'll have to update the submodule to latest configs, of course
 
2015-09-10 SailfishOS 2.0 on Nexus5: https://twitter.com/alinmelena/status/641901594674622465
 
2015-09-07 Motorola Moto G 1st Gen. (falcon) https://twitter.com/muhammadrefa/status/640920003097706496
 
2015-09-04 Motorola Moto G 2nd Gen. https://twitter.com/sledgeSim/status/639819004132093952
 
2015-09-02 Add yourself here if your port is active! https://wiki.merproject.org/wiki/Adaptations/libhybris/porters
 
2015-08-27 for icon pack lovers updated instructions line 174 http://piratepad.net/hadk-faq
 
2015-08-20 LG G watch! \o/ https://twitter.com/AsteroidOS/status/634496272553115648
 
2015-08-20 Sony Xperia SP \o/ https://twitter.com/sledgeSim/status/634424350708469760
 
2015-08-19 Xiaomi Redmi 1S \o/ https://twitter.com/sledgeSim/status/633992949890682880
 
2015-07-28 Maintainers of all mer packages: http://www.merproject.org/dash src: http://www.mail-archive.com/mer-general@lists.merproject.org/msg01557.html THANKS lbt \o/
 
2015-08-11 hybris-12.1 branch now available for creating cm-12.1 based ports
 
2015-08-06 Jolla Store works again for Nexus 5 and 4 and Xperia Pro!! and soon others! https://twitter.com/sledgeSim/status/629325072646467584 Search for "how to bring Jolla store to your device" in http://piratepad.net/hadk-faq
 
2015-07-26 Sony Xperia Pro release by mal-! https://twitter.com/sledgeSim/status/625235917486338048
 
2015-07-16 apkenv is now in :common: https://build.merproject.org/package/show/nemo:devel:hw:common/apkenv

    game compatibility (you'll need to download apk exactly the version indicated) http://wiki.maemo.org/Apkenv/Game_Compatibility

    tested games list in video description: https://www.youtube.com/watch?v=SEfCvV_mzKk

    if you install it and launch with any compatible android game apk, it should work on nexus4 out of the box

    nexus5 will need to wait until next release though

    (which states that 10.1 ports apkenv works already now, and 11.0 not yet ;P)

     

     

     

2015-07-02 GPS alpha works: http://forum.xda-develop
2015-04-30 Nexus5 alpha11 hotfix: https://twitter.com/alinmelena/status/593667006345404416
 
 
Nexus4 beta6 https://twitter.com/_undressed_/status/593787467146403842
 
2015-04-29 Nexus5 alpha10: https://twitter.com/alinmelena/status/593278121069178880
 
2015-04-24 Samsung Galaxy Nexus u13 build (pre-alpha) nightly thread: http://forum.xda-developers.com/galaxy-nexus/general/august-25th-sailfishos-nightly-galaxy-t3092570
 
2015-04-23 Samsung Galaxy Nexus u11 community build: http://forum.xda-developers.com/galaxy-nexus/general/august-3rd-sailfishos-galaxy-nexus-alpha-t2837096/post60312044#post60312044
 
2015-04-22 23:26 < MSameer> BTW if someone is brave to build gecko, get https://github.com/foolab/gecko-dev and build nemo_embedlite_31-gst1
23:27 < MSameer> to get [youtube] video playback
23:28 < MSameer> if it works be happy. if it does not then stay away from me :P
 
HP Touchpad release: https://twitter.com/JShafer817/status/590723522714009601
Nexus 7 (2013) release: https://twitter.com/vrutkovs/status/590630551981334528
 
Oppo Find5 video & camera: https://twitter.com/Nokius/status/590565167878037505 https://twitter.com/Nokius/status/590959270784536576
 
2015-04-21 Nexus 4 u13 release: https://twitter.com/_undressed_/status/590563744662560769
 
WhatsUp works on SFE devices! \o/ http://forum.xda-developers.com/showpost.php?p=60264278&postcount=137
 
2015-04-20 Nexus4 u13 early access:
image creation succesful
jolla apps in store wouldn't show up. Findable and installable manually is possible -- reproduced on Nexus5. Expect glitches in Jolla Store until tablet comes out
 
2015-04-16 Nexus5 u13 early preview successfully built and booted:
bugs reported R&D style:

    background parallax scrolling is inverted (workaround: set custom picture ambience)

     

2015-04-16 < vgrade> PSA, http://androidcommunity.com/lollipop-not-good-for-nexus-5-and-nexus-7-units-being-bricked-20150413/
 
2015-04-14 Reward$ from community to community: http://forum.xda-developers.com/google-nexus-5/development/rom-sailfish-os-alpha-t2841266/post60108604#post60108604
 
2015-04-06 You can help if you like refactoring: http://piratepad.net/dhd2modular
 
2015-04-07 N5 released with camera: https://twitter.com/alinmelena/status/585337663659630592
 
2015-04-06 Ready to release cameraplus with Nexus5 image after merging https://github.com/foolab/cameraplus/pull/43
almost not very ready as is pretty buggy but better than nothing 
 

    older news: http://bit.ly/port-news

        
      
