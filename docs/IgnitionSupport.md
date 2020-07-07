# Ignition Support in MachineConfig

`MachineConfig` objects are the defacto way of customizing the nodes in an
OpenShift cluster. Customizations can be applied "day 1" during the install of
the cluster or "day 2" after the cluster has been installed.

The way the customizations are defined are mostly through the use of the
Ignition specification. However, not all of the specification is supported.
This attempts to codify which parts of the [Ignition 3.1.0 specification](https://github.com/coreos/ignition/blob/master/doc/configuration-v3_1.md)
are supported and provide examples for **day 2** customizations.

The one top-level object which is explicitly **NOT** supported is `networkd`.
The remaining objects have varying levels of support which are discussed below.

## Ignition

The `ignition` object contains metadata about the Ignition configuration. When
used in the scope of the `MachineConfig` object, the `version` key should be
provided at a minimum.

```yaml
# This MachineConfig doesn't actually do anything
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: worker
  name: 99-ignition-version
spec:
  config:
    ignition:
      version: 3.1.0
```

Specifying any other objects under the top-level `ignition` object in a
MachineConfig is **NOT** supported.

## Storage

The top-level `storage` object of the Ignition specification drives how disks,
filesystems, and files are defined on the host. When using a MachineConfig for
"day 2" customizations, **ONLY** the `files` object is supported.

To define a file that will be written to the node, users **MUST** minimally
supply `contents`, `filesystem`, and `path` for the file.

The `contents` of the file **MUST** be specified using the `data:` format. (See
https://tools.ietf.org/html/rfc2397)

```yaml
# This MachineConfig demonstrates the minimal requirements for writing out a
# file to the node
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: worker
  name: 99-ignition-files-minimal
spec:
  config:
    ignition:
      version: 3.1.0
    storage:
      files:
        - contents:
            source: data:text/plain;charset=utf-8;base64,aGVsbG8gd29ybGQK
          filesystem: root
          path: /var/srv/hello-world.txt
```

Users can specify that the contents of the file should overwrite an existing
file.

```yaml
# This MachineConfig demonstrates overwriting an existing file
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: worker
  name: 99-ignition-files-overwrite
spec:
  config:
    ignition:
      version: 3.1.0
    storage:
      files:
        - contents:
            source: data:text/plain;charset=utf-8;base64,Z29vZGJ5ZSB3b3JsZAo=
          filesystem: root
          path: /var/srv/hello-world.txt
          overwrite: true
```

However, the `append` boolean for files is **NOT** supported at this time.

Users can configure the `mode`, `user`, and `group` ownership as well. The mode
can be specified as octal (`0644`) or as decimal (`420`). Users and groups can
be specified by their IDs or by their names.

```yaml
# This MachineConfig demonstrates configuring file attributes
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: worker
  name: 99-ignition-files-attributes
spec:
  config:
    ignition:
      version: 3.1.0
    storage:
      files:
        - contents:
            source: data:text/plain;charset=utf-8;base64,b3duZWQgYnkgcm9vdAo=
          filesystem: root
          path: /var/srv/owned-by-root.txt
          mode: 0644
          user:
            id: 0
          group:
            name: root
```

Optionally, users can provide the `verification` object as part of the file
definition, which will force the node to verify the contents of the file before
it is written to disk. The current supported verification scheme is sha512.

```yaml
# This MachineConfig demonstrates using the verification ability
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: worker
  name: 99-ignition-files-verify
spec:
  config:
    ignition:
      version: 3.1.0
    storage:
      files:
        - contents:
            source: data:text/plain;charset=utf-8;base64,b3duZWQgYnkgY29yZQo=
            verification:
              hash: sha512-5ee67ed4a5d2f4b9fc0039bca54deaf293e7ada3298fee84557761b54324ebcd0979e2753921a3fc9a0ffb2f0b4468487d253a3cae07c12b2051f599dc6819e8
          filesystem: root
          path: /var/srv/owned-by-core.txt
          mode: 420
          user:
            name: core
          group:
            id: 1000
```

# Systemd

XXX: Todo

# Passwd

XXX: Todo