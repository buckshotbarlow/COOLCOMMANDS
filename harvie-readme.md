kubectl get vm -A
kubectl patch vm <vm-name> -n <namespace> --type merge -p '{"spec":{"runStrategy":"RerunOnFailure"}}'
virtctl start <vm-name> -n <namespace>

Example:
harv-srv1:/home/rancher # kubectl get vm -A
NAMESPACE   NAME                    AGE    STATUS    READY
default     kalilinux2025-4         263d   Stopped   False
untagged    borg-srv1               580d   Running   True
untagged    ops-srv1                587d   Running   True
untagged    rock-srv1               579d   Running   True
untagged    ubuntu-2204-gold-copy   588d   Stopped   False
untagged    ubuntu-2410-gold-copy   586d   Stopped   False
untagged    wazu-srv1               586d   Running   True
harv-srv1:/home/rancher # kubectl patch vm kalilinux2025-4 -n default --type merge -p '{"spec":{"runStrategy":"RerunOnFailure"}}'
virtualmachine.kubevirt.io/kalilinux2025-4 patched
harv-srv1:/home/rancher # kubectl get vm kalilinux2025-4 -n default
NAME              AGE    STATUS     READY
kalilinux2025-4   263d   Starting   False

