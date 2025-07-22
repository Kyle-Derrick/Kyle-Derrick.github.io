---
title: 'ESXi7虚拟机克隆脚本实现'
author: ['kyle']
date: '2025-07-22T23:08:01+08:00'
tags:
- Linux
- ESXi
- 虚拟机

keywords:
- Linux
- ESXi
- 虚拟机
---

```sh
#!/bin/sh
datastore_path=/vmfs/volumes/datastore1
template_name=CentOS-Base-250609
nts_template_path="$datastore_path/$template_name"
new_path=$datastore_path/$1
if [ -d "$new_path" ]; then
  echo "exists vm : $1"
  exit 1
fi
mkdir $new_path
echo "clone vmdk..."
vmkfstools -i $nts_template_path/$template_name.vmdk $new_path/$1.vmdk -d eagerzeroedthick
echo "clone vmdk2..."
vmkfstools -i $nts_template_path/${template_name}_1.vmdk $new_path/$1_1.vmdk -d zeroedthick

echo "generate vmx..."
cp $nts_template_path/$template_name.vmx $new_path/$1.vmx
sed -i '/ethernet[0-9]\.generatedAddress/d' $new_path/$1.vmx
sed -i '/annotation = "/d' $new_path/$1.vmx
sed -i '/scsi[0-9]\.sasWWID = "/d' $new_path/$1.vmx
sed -i '/sched\.swap\.derivedName = "/d' $new_path/$1.vmx
sed -i '/migrate\.hostlog = "/d' $new_path/$1.vmx
sed -i '/nvram = "/d' $new_path/$1.vmx
sed -i '/vc\.uuid = "/d' $new_path/$1.vmx
sed -i '/uuid\.location = "/d' $new_path/$1.vmx
sed -i '/uuid\.bios = "/d' $new_path/$1.vmx
sed -i "/displayName = \"/c\displayName = \"$1\"" $new_path/$1.vmx

# custom disk file and ethernet
sed -i "/scsi0:0\.fileName = \"/c\scsi0:0.fileName = \"$1.vmdk\"" $new_path/$1.vmx
sed -i "/scsi0:1\.fileName = \"/c\scsi0:1.fileName = \"$1_1.vmdk\"" $new_path/$1.vmx
if [ -n "$2" ]; then
  sed -i '/ethernet0\.addressType = "/c\ethernet0.addressType = "static"' $new_path/$1.vmx
  sed -i "/ethernet0\.address = \"/c\ethernet0.address = \"$2\"" $new_path/$1.vmx
else
  sed -i '/ethernet0\.addressType = "/c\ethernet0.addressType = "generated"' $new_path/$1.vmx
  sed -i "/ethernet0\.address = \"/d" $new_path/$1.vmx
fi

echo "register vm..."
vim-cmd solo/registervm $new_path/$1.vmx
echo "over."
```