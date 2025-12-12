# Cụm Kubernetes (control plane đơn) để triển khai Frappe/ERPNext

Tài liệu tóm tắt các bước cài dựng cụm K8s “main” và sẵn sàng triển khai Frappe/ERPNext (tham chiếu hướng dẫn Frappe: https://docs.frappe.io/framework/user/en/installation).

## 1. Chọn môi trường
- Nên dùng Ubuntu 22.04 LTS trên VM hoặc máy vật lý (trên Windows nên dùng WSL2 hoặc VM).
- Control plane: 2 vCPU, 4–8 GB RAM, 30 GB disk. Worker: 2 vCPU, 4–8 GB RAM, 50+ GB disk.

## 2. Chuẩn bị node (mọi node, sudo)
```bash
sudo apt update && sudo apt upgrade -y
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy

cat <<'EOF' | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

cat <<'EOF' | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
sudo sysctl --system

sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl enable --now containerd

sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

## 3. Cài kubeadm/kubelet/kubectl
```bash
sudo apt install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-apt-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## 4. Khởi tạo control plane
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16

mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 5. Cài CNI (Calico)
```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.3/manifests/calico.yaml
```

## 6. Thêm worker nodes
- Lấy lệnh join trên control plane:
  ```bash
  kubeadm token create --print-join-command
  ```
- Chạy lệnh join trên từng worker (đã làm bước 2–3).

## 7. Cài ingress controller (NGINX)
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.3/deploy/static/provider/cloud/deploy.yaml
```
- Cloud: dùng LoadBalancer của cloud.
- On-prem: dùng NodePort + MetalLB để có IP ảo.

## 8. Cài Helm
```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

## 9. Chuẩn bị stack Frappe/ERPNext trên K8s
- MariaDB 10.6+/11.3 (bitnami/mariadb), PVC SSD, có thể bật replication.
- Redis 6 (bitnami/redis) cho cache + realtime.
- Ứng dụng Frappe/Bench: image có Python 3.10/11/12, Node 18/20, wkhtmltopdf. Mount PVC cho `sites`, `assets`, `logs`; đặt env DB/Redis.
- Tạo Ingress trỏ domain (vd. `site1.local` hoặc domain thực), gắn TLS (cert-manager nếu cần).

## 10. Kiểm tra nhanh
```bash
kubectl get nodes -o wide
kubectl get pods -A
```
- Khi deploy Frappe: `kubectl logs -f deploy/<frappe-app>` và thử truy cập qua Service/Ingress.

## 11. Ghi chú Windows
- Không init kubeadm trực tiếp trên Windows native. Dùng WSL2 Ubuntu hoặc VM.
- Có thể dùng k3s/k3d cho môi trường dev nhỏ; sản xuất vẫn nên kubeadm trên Ubuntu.

## 12. Triển khai manifest Frappe/ERPNext trong repo này
1) Chuẩn bị cluster  
   - Đã có storage class mặc định và ingress-nginx.  
   - Có DNS/hosts trỏ `site1.local` về IP ingress (dev: thêm vào `/etc/hosts`).  

2) Cập nhật giá trị cần bảo mật  
   - `k8s/frappe/01-secrets.yaml`: đổi `mariadb-root-password`, `mariadb-password`, `admin-password`, `encryption-key`.  
   - `k8s/frappe/02-configmap.yaml`: đổi `SITE_NAME` nếu khác `site1.local`.  
   - `k8s/frappe/06-ingress.yaml`: đổi host, bật `tls` khi dùng cert-manager.  

3) Cập nhật image ứng dụng  
   - `k8s/frappe/05-frappe-app.yaml`: thay `your-registry/frappe-app:latest` bằng image đã build chứa Python 3.10/11/12, Node 18/20, wkhtmltopdf và mã Frappe/ERPNext (theo yêu cầu trong https://docs.frappe.io/framework/user/en/installation).  

4) Áp dụng manifest (đúng thứ tự hoặc apply cả thư mục)  
   ```bash
   kubectl apply -f k8s/frappe/00-namespace.yaml
   kubectl apply -f k8s/frappe/01-secrets.yaml
   kubectl apply -f k8s/frappe/02-configmap.yaml
   kubectl apply -f k8s/frappe/03-mariadb.yaml
   kubectl apply -f k8s/frappe/04-redis.yaml
   kubectl apply -f k8s/frappe/05-frappe-app.yaml
   kubectl apply -f k8s/frappe/06-ingress.yaml
   # Hoặc: kubectl apply -f k8s/frappe/
   ```

5) Kiểm tra và truy cập  
   ```bash
   kubectl get pods -n frappe
   kubectl get svc -n frappe
   kubectl describe ingress frappe-ingress -n frappe
   kubectl logs -f deploy/frappe-app -n frappe
   ```
   - Đợi MariaDB/Redis ready, pod app Running.  
   - Truy cập `http://site1.local` (hoặc domain bạn đặt). Bật TLS nếu cần (cert-manager).  

6) Lưu ý dữ liệu & PVC  
   - MariaDB PVC 20Gi (`mariadb-pvc`), sites PVC 20Gi (`frappe-sites-pvc`). Đảm bảo storage class cung cấp RWO.  
   - Sao lưu PV/PVC trước khi nâng cấp.  

## 13. Ghi chú bổ sung cho kiến trúc Frappe trên K8s
- Runtime: MariaDB ≥10.6 (UTF8MB4), Redis 6 (cache/queue/socketio), Python 3.10–3.12, Node 18/20, Yarn ≥1.12, wkhtmltopdf 0.12.5+ (patched Qt), Bench CLI, cron.  
- Kiến trúc logic: MariaDB StatefulSet; Redis Deployment (có thể tách cache/queue/socketio); web Deployment; các worker `default/short/long`; socketio Deployment (port 9000); scheduler CronJob; Ingress TLS route `/` -> web, `/socket.io` -> socketio; PVC RWX cho `sites`, RWO cho DB.  
- Secrets/Config: DB root/app, admin password, redis auth (nếu dùng), `site_config.json` (encryption_key, db/redis URLs, host_name, mail), `common_site_config.json` cho bench-wide.  
- Skeleton vai trò:  
  - Init Job (chạy một lần): mount `sites`, `site_config.json`, chạy `bench new-site ...`, `bench migrate`, cài app.  
  - Web: `bench serve --port 8000`; mount `sites`; env DB/Redis.  
  - Workers: `bench worker --queue default|short|long`; mount `sites`.  
  - SocketIO: `bench socketio --port 9000`; mount `sites`.  
  - Scheduler: CronJob `bench schedule`; mount `sites`.  
  - Ingress: TLS qua cert-manager; route `/` và `/socket.io`.  
- Image: có wkhtmltopdf; build từ `frappe/bench` hoặc `frappe/erpnext-worker`; cài app khi build (`bench get-app`, `bench build`); entry script nên tạo site nếu thiếu, migrate, rồi exec theo role (web/worker/socketio).  
- Flow triển khai:  
  1) Apply Secrets/Config/PVC, sau đó MariaDB và Redis.  
  2) Đợi DB/Redis ready.  
  3) Chạy init Job một lần, xác nhận hoàn tất.  
  4) Deploy web/worker/socketio/scheduler, tạo Ingress và DNS/TLS.  
  5) Kiểm thử: port-forward hoặc domain, `bench --site <site> doctor` (Job) cho smoke test.  
- Scaling/replication: web scale bằng Deployment/HPA; workers per queue; build/push image tag mới khi cập nhật app; dùng Helm/values để tham số hóa domain/storageClass/tag/replica.  
- Ops: CronJob backup `bench backup` lên S3/object storage; logs stdout; thêm exporters MariaDB/Redis; bảo mật bằng NetworkPolicy và Secrets/SealedSecrets, vá image định kỳ.  