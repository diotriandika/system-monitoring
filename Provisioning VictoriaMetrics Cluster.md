### Install VictoriaMetrics Cluster

VictoriaMetrics merupakan time-series database yang didesain untuk high-performance monitoring dan analytics. VictoriaMetrics mirip dengan Prometheus disegala aspeknya, akan tetapi memiliki bebarapa kelebihan termasuk perfoma dan skalabilitas yang lebih baik. VictoriaMetrics juga berbasis open-source.

**Jalankan di Node Control Plane**

1. Tambahkan VictoriaMetrics Helm Repository lal update repo

   ```bash
   ## Add VM repo
   $ helm repo add vm https://victoriametrics.github.io/helm-charts/
   
   ## Update Helm repos
   $ helm repo update
   ```

   > `vm` (victoriametrics) bisa disesuaikan dengan keinginan, ini hanya nama repo.

2. Cek available vm charts

   ```bash
   $ helm repo search vm/
   NAME                            CHART VERSION   APP VERSION     DESCRIPTION 
   vm/victoria-logs-single         0.6.3           v0.28.0         Victoria Logs Single version - high-performance...
   vm/victoria-metrics-agent       0.12.2          v1.103.0        Victoria Metrics Agent - collects metrics from ...
   vm/victoria-metrics-alert       0.11.1          v1.103.0        Victoria Metrics Alert - executes a list of giv...
   vm/victoria-metrics-anomaly     1.4.6           v1.15.9         Victoria Metrics Anomaly Detection - a service ...
   vm/victoria-metrics-auth        0.6.0           v1.103.0        Victoria Metrics Auth - is a simple auth proxy ...
   vm/victoria-metrics-cluster     0.13.7          v1.103.0        Victoria Metrics Cluster version - high-perform...
   vm/victoria-metrics-common      0.0.13                          Victoria Metrics Common - contains shared templ...
   vm/victoria-metrics-distributed 0.3.1           v1.103.0        A Helm chart for Running VMCluster on Multiple ...
   vm/victoria-metrics-gateway     0.4.0           v1.103.0        Victoria Metrics Gateway - Auth & Rate-Limittin...
   vm/victoria-metrics-k8s-stack   0.25.17         v1.102.1        Kubernetes monitoring on VictoriaMetrics stack....
   vm/victoria-metrics-operator    0.34.8          v0.47.3         Victoria Metrics Operator                         
   vm/victoria-metrics-single      0.11.2          v1.103.0        Victoria Metrics Single version - high-performa...
   ```

3. Buat custom values seperti dibawah

   ```bash
   $ vim values.yaml
   ```

   ```yaml
   vmselect:
     podAnnotations:
       prometheus.io/scrape: "true"
       prometheus.io/port: "8481"
   
   vminsert:
     podAnnotations:
       prometheus.io/scrape: "true"
       prometheus.io/port: "8480"
   
   vmstorage:
     podAnnotations:
       prometheus.io/scrape: "true"
       prometheus.io/port: "8482"
     persistentVolumeClaim:
       enabled: true
       storageClassName: "nfs-client"
       accessModes:
       - ReadWriteOnce
   
   ```

   > Notes:
   >
   > - `podAnnotations: prometheus.io/scrape: "true"` artinya mengaktifkan scraping metrics dari pod vmselect, vminsert dan vmstorage.
   > - `podAnnotations:prometheus.io/port: "some_port"` artinya mengaktifkan scraping metrics dari Pod vmselect, vminsert, dan vmstorage melalui port masing - masing.
   > - Karena saya menggunakan NFS External Provisioner, maka dari itu saya menambahkan StorageClass `nfs-client` untuk PVC vmstorage. Sehingga akan langsung membuat PV yang diinginka

4. Install VictoriaMetrics Cluster dari Helm Chart.

   ```bash
   $ helm upgrade --install vmcluster vm/victoria-metrics-cluster --create-namespace -n monitoring --values values.yaml
   ```

5. Akan muncul output seperti dibawah, namun untuk saat ini yang perlu diingat hanya url datasourcenya saja.

   ![image-20240922232516924](C:\Users\Dio Tri Andika\AppData\Roaming\Typora\typora-user-images\image-20240922232516924.png)

   ```bash
   http://vmcluster-victoria-metrics-cluster-vmselect.monitoring.svc.cluster.local:8481/select/0/prometheus/
   ```

6. Cek status pod victoriametrics

   ```bash
   $ kubectl get pod -n monitoring
   NAME                                                           READY   STATUS    RESTARTS   AGE
   vmcluster-victoria-metrics-cluster-vminsert-74b848994c-z2ksd   1/1     Running   0          31m
   vmcluster-victoria-metrics-cluster-vminsert-74b848994c-zfr55   1/1     Running   0          31m
   vmcluster-victoria-metrics-cluster-vmselect-9dc9dd7b6-9mbrd    1/1     Running   0          31m
   vmcluster-victoria-metrics-cluster-vmselect-9dc9dd7b6-wsr22    1/1     Running   0          31m
   vmcluster-victoria-metrics-cluster-vmstorage-0                 1/1     Running   0          31m
   vmcluster-victoria-metrics-cluster-vmstorage-1                 1/1     Running   0          30m
   ```

7. Install `vmagent` dengan custom config file untuk dapat mengscrape metrics dari kubernetes.

   ```bash 
   $ helm install vmagent vm/victoria-metrics-agent -f https://docs.victoriametrics.com/guides/examples/guide-vmcluster-vmagent-values.yaml
   ```

8. Cek pod `vmagent`

   ```bash
   $ kubectl get pod | grep vmagent
   vmagent-victoria-metrics-agent-5454fc49ff-c2wk4                   1/1     Running   0              83s
   ```

Referensi:

- https://docs.victoriametrics.com/guides/k8s-monitoring-via-vm-cluster/
- https://medium.com/@seifeddinerajhi/victoriametrics-a-comprehensive-guide-comparing-it-to-prometheus-and-implementing-kubernetes-03eb8feb0cc2
