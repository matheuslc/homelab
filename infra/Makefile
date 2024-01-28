ZFSReleaseName = zfs-csi
ZFSStorageClassReleaseNAme = zfs-driver
DBReleaseName = lab-db
StorageReleaseName = lab-storage
MinioOperatorReleaseName = minio-operator
PrometheusReleaseName = prom-dev

.PHONY: deps zfscsi zfscsi-uninstall zfscsi-driver zfs-driver-uninstall cnpg cnpg-uninstall storage-install storage-uninstall start reset

start:
	$(MAKE) zfscsi
	$(MAKE) zfs-driver
	$(MAKE) cnpg
	@echo "Waiting 30 seconds so the CNPG Operator can start"
	sleep 30s
	$(MAKE) db-install
	$(MAKE) minio-install
	$(MAKE) storage-install
	$(MAKE) prom-install

uninstall-all:
	$(MAKE) prom-uninstall
	$(MAKE) storage-uninstall
	$(MAKE) minio-uninstall
	$(MAKE) db-uninstall
	$(MAKE) cnpg-uninstall
	$(MAKE) zfs-driver-uninstall
	$(MAKE) zfscsi-uninstall

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
	@microk8s helm uninstall $(ZFSStorageClassReleaseNAme)

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
	@echo "Installing the Postgres Database: $(DBReleaseName)"
	@microk8s helm install $(DBReleaseName) ./pg/charts --atomic

db-uninstall:
	@echo "Removing the Postgres Database: $(DBReleaseName)"
	@microk8s helm uninstall $(DBReleaseName)

minio-install:
	@echo "Installing the Object Storage (minio): $(MinioOperatorReleaseName)"
	@microk8s helm repo add minio-operator https://operator.min.io
	@microk8s helm upgrade --install $(MinioOperatorReleaseName) minio-operator/operator -f ./storage/charts/values.yaml --namespace minio-operator --create-namespace --atomic
		
minio-uninstall:
	@echo "Removing the Object Storage (minio): $(MinioOperatorReleaseName)"
	@microk8s helm uninstall $(MinioOperatorReleaseName) -n minio-operator

storage-install:
	@echo "Installing a Minio Tenant"
	@microk8s helm upgrade --install $(StorageReleaseName) minio-operator/tenant  --namespace storage --create-namespace --atomic -f ./storage/charts/tenant.yaml

storage-uninstall:
	@echo "Removing the Minio Tenant"
	@microk8s helm uninstall $(StorageReleaseName) -n storage

prom-install:
	@echo "Installing Prometheus"
	# @microk8s helm repo add prometheus-community https://prometheus-community.github.io/helm-chart
	@microk8s helm upgrade --install prometheus prometheus-community/prometheus

prom-uninstall:
	@echo "Removing Prometheus"
	@microk8s helm uninstall prometheus

prom-expose:
	@microk8s kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-external

grafana-install:
	@echo "Installing Grafana"
	@microk8s helm repo add grafana https://grafana.github.io/helm-charts
	@microk8s helm repo update
	@microk8s helm upgrade --install grafana grafana/grafana --set persistence.enabled=true

grafana-uninstall:
	@echo "Removing Grafana"
	@microk8s helm uninstall grafana

reset:
	@echo "Reseting the cluster"
	@microk8s reset