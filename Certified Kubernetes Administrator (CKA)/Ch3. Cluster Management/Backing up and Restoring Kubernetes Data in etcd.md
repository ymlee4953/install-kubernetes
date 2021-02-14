# **Backing up and Restoring Kubernetes Data in etcd**
## **Introduction**
Backups are an important part of any resilient system. Kubernetes is no exception. In this lab, you will have the opportunity to practice your skills by backing up and restoring a Kubernetes cluster state stored in etcd. This will help you get comfortable with the steps involved in backing up Kubernetes data.

## **Solution**
Log in to the provided lab server using the credentials provided:

    ssh cloud_user@<PUBLIC_IP_ADDRESS>

### **Back Up the etcd Data**
1. Look up the value for the key cluster.name in the etcd cluster:

        ETCDCTL_API=3 etcdctl get cluster.name \
        --endpoints=https://10.0.1.101:2379 \
        --cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \
        --cert=/home/cloud_user/etcd-certs/etcd-server.crt \
        --key=/home/cloud_user/etcd-certs/etcd-server.key

    The returned value should be beebox.

2. Back up etcd using etcdctl and the provided etcd certificates:

        ETCDCTL_API=3 etcdctl snapshot save /home/cloud_user/etcd_backup.db \
        --endpoints=https://10.0.1.101:2379 \
        --cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \
        --cert=/home/cloud_user/etcd-certs/etcd-server.crt \
        --key=/home/cloud_user/etcd-certs/etcd-server.key

3. Reset etcd by removing all existing etcd data:

        sudo systemctl stop etcd

        sudo rm -rf /var/lib/etcd

### **Restore the etcd Data from the Backup**
1. Restore the etcd data from the backup (this command spins up a temporary etcd cluster, saving the data from the backup file to a new data directory in the same location where the previous data directory was):

        sudo ETCDCTL_API=3 etcdctl snapshot restore /home/cloud_user/etcd_backup.db \
        --initial-cluster etcd-restore=https://10.0.1.101:2380 \
        --initial-advertise-peer-urls https://10.0.1.101:2380 \
        --name etcd-restore \
        --data-dir /var/lib/etcd

2. Set ownership on the new data directory:

        sudo chown -R etcd:etcd /var/lib/etcd

3. Start etcd:

        sudo systemctl start etcd
4. Verify the restored data is present by looking up the value for the key cluster.name again:

        ETCDCTL_API=3 etcdctl get cluster.name \
        --endpoints=https://10.0.1.101:2379 \
        --cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \
        --cert=/home/cloud_user/etcd-certs/etcd-server.crt \
        --key=/home/cloud_user/etcd-certs/etcd-server.key

    The returned value should be beebox.

## **Conclusion**
Congratulations on successfully completing this hands-on lab!