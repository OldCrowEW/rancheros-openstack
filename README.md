This repo gets you going with RancherOS on OpenStack.

[Packer](http://packer.io) is used with the qemu builder to create the image which can be imported to glance. It's expected all of these requirements are in place beforehand.

## Hacks in place:
Rancher has removed the default username/password for what appears to be security reasons. https://github.com/rancher/os/issues/1198 Until a better solution is in place we must set a password at boot by doing the following:
```
"qemuargs": [
        ["-m", "1024M"],
        ["-kernel", "/mnt/boot/vmlinuz"],
        ["-initrd", "/mnt/boot/initrd"],
        ["-append", "quiet rancher.autologin=tty1 rancher.autologin=ttyS0 rancher.password=rancher"]
      ]
```
Qemu requires -kernel -initrd when using -append. Update the paths for correct location of kernel and initrd. 

## Build
```
packer build packer-rancher.json
```

## Import
Packer will create a qcow2 type image in `output_rancher` dir. You can import it into glance:

```
  glance image-create --name RancherOS \
  --container-format bare \
  --disk-format qcow2 \
  --file output_rancher/rancheros
```

## Usage
We hard code an insecure key to our cloud_config.yml file so we can ssh into the instance. **You should replace these keys before building image**
```
ssh -i insecure_keys/id_rsa_rancheros rancher@{{ instance_ip }}
```

## Upgrade ISO checksum

If a new version is released, you have to update the `iso_checksum`. ```md5sum rancheros.iso``` is your friend