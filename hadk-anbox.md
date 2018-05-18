Anbox for Sailfish OS

Work in progress

Status

    Working:

    Single-window UI

    Touchscreen input

    Networking

    Audio (has 1-2 second delay)

    Gamezzz (:P)

    Not working

    Probably everything else


Requirements:

    Sailfish OS device with at least 3.10 kernel with needed patches applied

    Patches from https://github.com/anbox/anbox/tree/master/kernel/patches

    Suitable patch from https://github.com/adilinden/overlayfs-patches

    defconfig changes from (might require some other defconfig changes also, use lxc-checkconfig to check the configuration) https://github.com/mlehtima/android_kernel_sony_msm/commit/947b4eb5c26c0eb22c708d73ca712b04f1a1d960

    Packages from https://build.merproject.org/project/show/home:mal:anbox

    anbox-sailfishos

    One of the following

    anbox-sailfishos-image-32bit

    anbox-sailfishos-image-mixed-32-64-bit (if your device has TARGET_USES_64_BIT_BINDER := true in BoardConfig.mk)


Changes from upstream anbox:

    GPU is utilized directly without pipes through sockets, since we are using libhybris anyways, the way through the pipe will always be slower.

    one could argue it's not as "clean" (i think it's cleaner, but it's definately not a clean implementation, hacked together in a few days), or not as "secure" but afaik anbox runs the container as privileged anyhow (NOT SURE IF STILL TRUE!) and many kernel modules/drivers don't implement namespaces.

    means we added wayland support into the rootfs

    means we had to implement input differently

    there are a few of hacks to make it simple

    overlayfs is used (can be avoided)

    in some cases we hacked out boost dependancies just to make it compile (should be fixed)

    Spaghetti taste better than shoes

    because they're actual food
