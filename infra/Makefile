ZFSReleaseName = zfs-csi
ZFSStorageClassReleaseNAme = zfs-driver

.PHONY: deps zfscsi zfscsi-uninstall zfscsi-driver zfs-driver-uninstall cnpg cnpg-uninstall

deps:
	@echo "Installing microk8s addons"
	@microk8s enable dns


zfscsi:
	@echo "Adding the helm repo for the openebs local zfs driver"
	@microk8s helm repo add openebs-zfslocalpv https://openebs.github.io/zfs-localpv
	@microk8s helm repo update
	@echo "Installing zfs-localpv from OpenEBS"
	@echo "Release name ${ZFSReleaseName}"
	@microk8s helm install ${ZFSReleaseName} openebs-zfslocalpv/zfs-localpv --create-namespace --set zfsNode.kubeletDir="/var/snap/microk8s/common/var/lib/kubelet/"

zfscsi-uninstall:
	@echo "Uninstalling zfs-localpv from OpenEBS"
	@microk8s helm uninstall $(ZFSReleaseName)

zfs-driver:
	@echo "Installing the ZFS Driver StorageClass"
	@microk8s helm install $(ZFSStorageClassReleaseNAme) ./zfs/charts --atomic

zfs-driver-uninstall:
	@echo "Removing the ZFS Driver StorageClass: $(ZFSStorageClassReleaseNAme)"
	@microk8s helm uninstall $(ZFSStorageClassReleaseNAme) ./zfs/charts

cnpg:
	@echo "Installing the Cloud Native Postgres Operator"
	@microk8s helm repo add cnpg https://cloudnative-pg.github.io/charts
	@microk8s helm upgrade --install cnpg \
		--namespace cnpg-system \
		--create-namespace \
		cnpg/cloudnative-pg

cnpg-uninstall:
	@echo "Removing the Cloud Native Postgres Operator"
	@microk8s helm uninstall cnpg --namespace cnpg-system

db-install:
	@echo "Installing the Postgres Database"
	@microk8s helm install lab-db ./pg/Charts --atomic