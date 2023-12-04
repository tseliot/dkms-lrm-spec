# dkms-lrm-spec
Temporary spec to be merged into u-d-c when finalised

Packaging drivers to be detected by ubuntu-drivers
--------------------------------------------------

#### User space

User space driver packages should expose the following entry in their
debian/control file, followed by the list of the modaliases for the
supported devices:

```
XB-Modaliases: ${modaliases}
```

Filling out the ```debian/$(pkg_name).modaliases``` file with a list of modaliases
using the following formatting (one device per line):

```
alias pci:v0000${vendor}d0000${device}sv*sd*bc03sc*i* ${module_name} $package_name}
```

For example:
```
alias pci:v000010DEd00001340sv*sd*bc03sc*i* nvidia nvidia-driver-545
...
```

Calling ```dh_modaliases``` in the ```debian/rules``` will replace ```${modaliases}``` with
a representation of the modaliases.

#### [Optional] Kernel space

Driver packages consuming pre-built signed kernel modules in the Ubuntu archive should provide a meta-package which conflicts with their dkms package.

Where the DKMS based driver package is ```foo``` (or ```foo-dkms```) and the non-DKMS package is ```foo-altkmod```.

```
Package: foo-dkms
Depends:
 dkms
 ${misc:Depends}
Conflicts:
 foo-altkmod
```


```
Package: foo-altkmod
Depends:
 ${misc:Depends}
Conflicts:
 foo-dkms
XB-UbuntuDrivers-Prebuilt: linux-modules-foo
```


While the former relies on DKMS, the latter will have its matching linux-modules (whose name scheme is described in the ```XB-UbuntuDrivers-Prebuilt``` entry) depending on it, allowing the correct kernel modules to always be updated in sync with the kernel.


The packages providing prebuilt kernel modules will then depend on the ```-altkmod``` package, as follows:

```
linux-modules-foo-$(KERNEL_FLAVOUR):
Depends:
 foo-altkmod (>= $(UPSTREAM_VERSION)), foo-altkmod (<= $(UPSTREAM_VERSION)-1)
```

The versioned dependency is needed when the kernel driver version and the user space version need to match (for compatiblity reasons).
