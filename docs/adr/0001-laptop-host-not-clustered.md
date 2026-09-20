# Laptop Host is not a Proxmox cluster node

The Laptop Host is a permanent light Host, not a trial box and not the first node of a future cluster. Clustering two uneven machines would leak into storage, networking, and upgrades now, and the Desktop Host is unspecified. Later agents must not “prepare” this install for clustering.
