helm repo add longhorn https://charts.longhorn.io
helm repo update
helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace --version 1.9.1
kubectl -n longhorn-system get pod


USER=<USERNAME_HERE>; PASSWORD=<PASSWORD_HERE>; echo "${USER}:$(openssl passwd -stdin -apr1 <<< ${PASSWORD})" >> auth
kubectl -n longhorn-system create secret generic basic-auth --from-file=auth
kubectl -n longhorn-system apply -f longhorn-ingress.yml


######################################################################
######			DEPENDENCIAS PARA PASPBERRY PI 5 K3S			######
######################################################################

1-)  Instalar dependencias en cada nodo
sudo apt-get update
sudo apt-get install -y open-iscsi nfs-common

2) Habilitar y arrancar los servicios
# Algunos sistemas sólo tienen iscsid; en otros también open-iscsi
sudo systemctl enable --now iscsid || true
sudo systemctl enable --now open-iscsi || true

# Verifica
systemctl status iscsid --no-pager
iscsiadm --version

3) (Recomendado) Cargar módulos iSCSI del kernel
# Cargar ahora
sudo modprobe iscsi_tcp
sudo modprobe libiscsi
sudo modprobe scsi_transport_iscsi



# Hacerlo persistente
echo -e "iscsi_tcp\nlibiscsi\nscsi_transport_iscsi" | sudo tee /etc/modules-load.d/iscsi.conf
echo dm_crypt | sudo tee -a /etc/modules

###	En Ubuntu para Raspberry, si algún módulo faltara, instala:
	sudo apt-get install -y linux-modules-extra-raspi


4) (Sólo si usas k3s) asegurarte de que Longhorn pueda “entrar” al host
which iscsiadm
ls -l /sbin/iscsiadm /usr/sbin/iscsiadm
## Si estuviera, pero el contenedor aún no lo encuentra, un atajo es crear un enlace simbólico extra:
# Sólo si fuera necesario
sudo ln -s /usr/sbin/iscsiadm /sbin/iscsiadm || true


5) Reiniciar Longhorn Manager

kubectl -n longhorn-system rollout restart daemonset/longhorn-manager
kubectl -n longhorn-system get pods -w
kubectl -n longhorn-system logs daemonset/longhorn-manager | tail -n +1

6) Comprobación rápida de salud
En el dashboard de Longhorn (o con kubectl -n longhorn-system get volumes), crea un volumen de prueba y móntalo en un Pod.

Verifica que el attacher y los replicas estén Healthy.



