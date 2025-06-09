# Confidential computing

## AMD Secure Encrypted Virtualization (SEV)

**FEATURE STATE:** KubeVirt v0.49.0 (experimental support)

[Secure Encrypted Virtualization (SEV)](https://developer.amd.com/sev/) is a feature of AMD's EPYC CPUs that allows the memory of a virtual machine to be encrypted on the fly.

KubeVirt supports running confidential VMs on AMD EPYC hardware with SEV feature.

### Preconditions

In order to run an SEV guest the following condition must be met:

- `WorkloadEncryptionSEV` [feature gate](../cluster_admin/activating_feature_gates.md#how-to-activate-a-feature-gate) must be enabled.
- The guest must support [UEFI boot](../compute/virtual_hardware.md#biosuefi)
- SecureBoot must be disabled for the guest VM

### Running an SEV guest

SEV memory encryption can be requested by setting the `spec.domain.launchSecurity.sev` element in the VMI definition:

```yaml
apiVersion: kubevirt.io/v1
kind: VirtualMachineInstance
metadata:
  labels:
    special: vmi-fedora
  name: vmi-fedora
spec:
  domain:
    launchSecurity:
      sev: {}
    firmware:
      bootloader:
        efi:
          secureBoot: false
    devices:
      disks:
      - disk:
          bus: virtio
        name: containerdisk
      - disk:
          bus: virtio
        name: cloudinitdisk
      rng: {}
    resources:
      requests:
        memory: 1024M
  terminationGracePeriodSeconds: 0
  volumes:
  - containerDisk:
      image: registry:5000/kubevirt/fedora-with-test-tooling-container-disk:devel
    name: containerdisk
  - cloudInitNoCloud:
      userData: |-
        #cloud-config
        password: fedora
        chpasswd: { expire: False }
    name: cloudinitdisk
```

### Current limitations

- SEV-encrypted VMs cannot contain directly-accessible host devices (that is, PCI passthrough)
- Live Migration is not supported
- The VMs are not attested

## AMD Secure Encrypted Virtualization - Secure Nested Paging (SEV-SNP)

**FEATURE STATE:** KubeVirt v0.53.0 (experimental support)

[AMD Secure Encrypted Virtualization-Secure Nested Paging (SEV-SNP)](https://www.amd.com/en/developer/sev.html#:~:text=AMD%20Secure%20Encrypted%20Virtualization%2DSecure%20Nested%20Paging%20(SEV%2DSNP)) provides strong memory integrity protection against malicious hypervisor attacks like data replay and memory re-mapping, while offering additional security enhancements for VM isolation, interrupt protection, and side channel attack mitigation.

### Pre-conditions
- `WorkloadEncryptionSEV` [feature gate](../cluster_admin/activating_feature_gates.md#how-to-activate-a-feature-gate) must be enabled.
- The guest must support [UEFI boot](../compute/virtual_hardware.md#biosuefi)
- The host machine must support SEV-SNP

### Running an SEV-SNP guest
```yaml
---
apiVersion: kubevirt.io/v1
kind: VirtualMachineInstance
metadata:
  labels:
    special: vmi-fedora-snp
  name: vmi-fedora-snp
spec:
  domain:
    launchSecurity:
      sev:
        policy:
          secureNestedPaging: true
    firmware:
      bootloader:
        efi:
          secureBoot: false
    devices:
      disks:
      - disk:
          bus: virtio
        name: containerdisk
      - disk:
          bus: virtio
        name: cloudinitdisk
      disableHotplug: true
    resources:
      requests:
        memory: 1024M
  terminationGracePeriodSeconds: 0
  volumes:
  - containerDisk:
      image: quay.io/containerdisks/fedora
    name: containerdisk
  - cloudInitNoCloud:
      userData: |-
        #cloud-config
        chpasswd:
          list: |
            root:fedora
          expire: False
    name: cloudinitdisk
```

### Current Limitations
- No support for explicitly setting the 64-bit security policy defaults to `0x00030000` which allows SMT and sets no minimum ABI version
- Live migrations are not supported
- VMs are currently not attested
