# Laptop Host physical constraints

The Laptop Host is a Dell Inspiron 14 7420 2-in-1: no Ethernet jack, one USB-A, two USB-C. USB-C carries charger and Uplink; USB-A carries the Library Disk. It runs closed, on AC, not in tablet mode, with lid-close sleep disabled. Internal 500 GB SSD is formatted ext4+LVM, not ZFS. Later agents must not assume a built-in NIC or spare USB-A ports.