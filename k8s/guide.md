# Tìm hiểu chi tiết về Kubernetes (K8s)

Kubernetes (thường được viết tắt là K8s) là một hệ thống mã nguồn mở mạnh mẽ được thiết kế để tự động hóa việc triển khai, mở rộng và quản lý các ứng dụng được đóng gói (containerized applications). Được phát triển bởi Google dựa trên kinh nghiệm vận hành hệ thống nội bộ Borg của họ, Kubernetes đã trở thành tiêu chuẩn công nghiệp cho việc điều phối container.

## I. Các Khái niệm Cơ bản trong Kubernetes Cluster

Một **Kubernetes Cluster** là một tập hợp các máy tính (vật lý hoặc ảo) được cấu hình để hoạt động như một đơn vị duy nhất, cùng nhau chạy các ứng dụng của bạn. Cluster này được chia thành hai loại nút chính: **Master Node** và **Worker Nodes**.

### 1. Node (Nút)

Trong Kubernetes, một **Node** là đơn vị phần cứng máy tính nhỏ nhất, đại diện cho một máy chủ vật lý hoặc máy ảo trong cụm. Mỗi Node cung cấp các tài nguyên tính toán như CPU và RAM, sẵn sàng để chạy các tác vụ. Các Node cần cài đặt Kubernetes và một công cụ runtime container (như Docker hoặc containerd) để có thể quản lý và chạy các container.

### 2. Master Node (Nút Điều khiển / Control Plane)

**Master Node** (hay còn gọi là Control Plane) là "bộ não" của cụm Kubernetes. Nó chịu trách nhiệm quản lý và điều phối toàn bộ hoạt động của cụm, đảm bảo rằng trạng thái thực tế của cụm luôn khớp với trạng thái mong muốn mà bạn đã định nghĩa.

Các thành phần chính của Master Node bao gồm:

*   **kube-apiserver:** Đây là giao diện chính của Kubernetes API. Mọi tương tác (từ người dùng, các công cụ dòng lệnh như `kubectl`, hoặc các thành phần khác trong cụm) đều phải thông qua API server. Nó xử lý các yêu cầu REST, xác thực, ủy quyền và cập nhật trạng thái của các đối tượng Kubernetes.
*   **etcd:** Một cơ sở dữ liệu key-value phân tán, nhất quán cao, được sử dụng để lưu trữ toàn bộ dữ thái cấu hình của cụm Kubernetes. Mọi dữ liệu quan trọng về cụm, bao gồm cấu hình ứng dụng, trạng thái của Pods, Services, v.v., đều được lưu trữ tại đây.
*   **kube-scheduler:** Thành phần này giám sát các Pod mới được tạo mà chưa được gán vào một Node nào. Dựa trên các yếu tố như yêu cầu tài nguyên (CPU, RAM), các ràng buộc về phần cứng/phần mềm, chính sách, và khả năng sẵn có của Node, scheduler sẽ chọn Node phù hợp nhất để chạy Pod đó.
*   **kube-controller-manager:** Chạy nhiều bộ điều khiển (controllers) khác nhau, mỗi bộ điều khiển chịu trách nhiệm duy trì một trạng thái mong muốn cụ thể trong cụm. Ví dụ:
    *   **Node Controller:** Giám sát trạng thái của các Node, thông báo khi một Node không hoạt động.
    *   **Replication Controller:** Đảm bảo số lượng bản sao (replicas) của Pods luôn đúng như mong muốn.
    *   **Endpoints Controller:** Kết nối Services với Pods.
    *   **Service Account & Token Controllers:** Quản lý các tài khoản dịch vụ và token API.
*   **cloud-controller-manager (tùy chọn):** Thành phần này tích hợp Kubernetes với các API của nhà cung cấp dịch vụ đám mây (ví dụ: AWS, Google Cloud Platform, Azure). Nó quản lý các tài nguyên đám mây như load balancers, persistent volumes, và các Node trong môi trường đám mây.

### 3. Worker Node (Nút Làm việc)

**Worker Node** là nơi các ứng dụng của bạn thực sự chạy. Mỗi Worker Node nhận lệnh từ Master Node và thực thi các tác vụ được giao, chẳng hạn như chạy Pods, quản lý mạng và lưu trữ cho các Pods đó.

Các thành phần chính của Worker Node bao gồm:

*   **kubelet:** Một agent chạy trên mỗi Worker Node. Kubelet giao tiếp với Master Node, nhận các định nghĩa Pod và đảm bảo rằng các container được chỉ định trong Pod đó đang chạy và khỏe mạnh trên Node của nó. Nó cũng báo cáo trạng thái của Node và Pods về Master Node.
*   **kube-proxy:** Duy trì các quy tắc mạng trên các Node. Kube-proxy xử lý việc chuyển tiếp lưu lượng mạng đến các Pods, cho phép giao tiếp mạng giữa các Pods trong cụm và từ bên ngoài cụm đến các Pods. Nó hoạt động như một bộ cân bằng tải phần mềm cơ bản.
*   **Container Runtime:** Đây là phần mềm chịu trách nhiệm chạy các container. Ví dụ phổ biến là Docker, nhưng Kubernetes cũng hỗ trợ các runtime khác như containerd hoặc CRI-O.

### 4. Pods (Đơn vị triển khai nhỏ nhất)

**Pods** là đơn vị triển khai nhỏ nhất và cơ bản nhất trong Kubernetes. Một Pod đại diện cho một instance duy nhất của một ứng dụng đang chạy.

*   **Mối quan hệ với Container:** Một Pod có thể chứa một hoặc nhiều container. Điều quan trọng là tất cả các container trong cùng một Pod luôn được triển khai và chạy cùng nhau trên cùng một Worker Node. Chúng được coi là một đơn vị quản lý duy nhất.
*   **Tài nguyên chia sẻ:** Các container trong cùng một Pod chia sẻ các tài nguyên sau:
    *   **Network Namespace:** Chúng chia sẻ cùng một địa chỉ IP và các cổng mạng. Điều này có nghĩa là các container trong cùng một Pod có thể giao tiếp với nhau bằng cách sử dụng `localhost` và các cổng đã được cấu hình.
    *   **Storage Volumes:** Các container trong Pod có thể truy cập và chia sẻ cùng một hoặc nhiều volume lưu trữ, cho phép chúng chia sẻ dữ liệu hoặc duy trì trạng thái.
*   **Mục đích:** Pods được sử dụng để nhóm các container có mối quan hệ chặt chẽ, cần chia sẻ tài nguyên và giao tiếp gần gũi. Ví dụ điển hình là một container ứng dụng chính và một "sidecar" container đi kèm để thực hiện các tác vụ phụ trợ như logging, giám sát, hoặc đồng bộ hóa dữ liệu.
*   **Khả năng mở rộng (Scaling):** Khi bạn cần mở rộng ứng dụng để xử lý nhiều tải hơn, Kubernetes sẽ tạo thêm các bản sao của toàn bộ Pod đó, chứ không phải từng container riêng lẻ. Điều này giúp việc quản lý và mở rộng ứng dụng trở nên đơn giản và hiệu quả.

## II. Các Chức năng Cơ bản của Kubernetes

Kubernetes cung cấp một bộ công cụ và khả năng toàn diện để quản lý vòng đời của ứng dụng container. Các chức năng cơ bản bao gồm:

*   **Create Kubernetes Cluster (Tạo cụm Kubernetes):** Kubernetes cho phép bạn dễ dàng khởi tạo một cụm hoàn chỉnh, từ một Node duy nhất (như với Minikube) đến các cụm đa Node phức tạp trên môi trường on-premise hoặc đám mây.
*   **Deploy an App (Triển khai ứng dụng):** Tự động hóa quá trình đưa ứng dụng của bạn lên cụm. Bạn chỉ cần cung cấp định nghĩa ứng dụng (thường dưới dạng file YAML), và Kubernetes sẽ đảm bảo ứng dụng được triển khai, chạy và duy trì theo cấu hình mong muốn.
*   **Explore App (Khám phá ứng dụng):** Cung cấp các công cụ để kiểm tra trạng thái hiện tại của ứng dụng, xem log từ các container, kiểm tra tài nguyên sử dụng, và khắc phục sự cố.
*   **Expose App Publicity (Công khai ứng dụng):** Kubernetes cung cấp các cơ chế như `Service` (ClusterIP, NodePort, LoadBalancer) và `Ingress` để ứng dụng của bạn có thể truy cập được từ bên ngoài cụm, cho phép người dùng cuối tương tác với nó.
*   **Scale Up App (Mở rộng ứng dụng):** Tự động hoặc thủ công tăng/giảm số lượng bản sao (Pods) của ứng dụng để đáp ứng nhu cầu tải. Kubernetes có thể tự động mở rộng dựa trên các chỉ số sử dụng tài nguyên (CPU, Memory) hoặc các chỉ số tùy chỉnh khác.
*   **Update App (Cập nhật ứng dụng):** Thực hiện các bản cập nhật ứng dụng một cách an toàn và không gây gián đoạn (rolling updates). Kubernetes sẽ dần dần thay thế các phiên bản cũ của Pods bằng các phiên bản mới, đảm bảo ứng dụng luôn sẵn sàng trong quá trình cập nhật.

---

## III. Tìm hiểu về Minikube (Phần 2)

Minikube là một công cụ giúp bạn cài đặt và chạy một cụm Kubernetes (K8s) đơn nút trên máy tính cục bộ của mình, rất phù hợp cho việc học tập và nghiên cứu. Nó hỗ trợ nhiều hệ điều hành như Linux, macOS và Windows.

### 1. Yêu cầu cấu hình để cài đặt Minikube:
*   Tối thiểu 2 CPU.
*   2GB RAM trống.
*   20GB dung lượng ổ cứng trống.
*   Cài đặt một trong các trình điều khiển ảo hóa như Docker, Hyperkit, Hyper-V, KVM, Parallels, Podman, VirtualBox hoặc VMWare.
*   Kết nối Internet.

### 2. Các lệnh Minikube cơ bản:

*   **Khởi động Minikube:**
    ```bash
    minikube start
    # Hoặc với driver cụ thể, ví dụ:
    minikube start --driver=docker
    ```
*   **Truy cập Dashboard:**
    ```bash
    minikube dashboard
    ```
    Lệnh này sẽ tự động mở giao diện Dashboard của Kubernetes trong trình duyệt.
*   **Tạm dừng/Tiếp tục cluster:**
    ```bash
    minikube pause
    minikube unpause
    ```
*   **Dừng/Khởi động lại cluster:**
    ```bash
    minikube stop
    minikube start
    ```
*   **Thay đổi cấu hình (ví dụ: bộ nhớ):**
    ```bash
    minikube config set memory XXXXX
    ```
    (Cần khởi động lại Minikube để áp dụng thay đổi).
*   **Xóa tất cả các cluster:**
    ```bash
    minikube delete --all
    ```
*   **Triển khai ứng dụng (Deployment và Service):**
    ```bash
    kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4
    kubectl expose deployment hello-minikube --type=NodePort --port=8080
    ```
*   **Kiểm tra trạng thái dịch vụ:**
    ```bash
    kubectl get services
    ```
*   **Truy cập dịch vụ:**
    ```bash
    minikube service hello-minikube
    ```
*   **Tạo LoadBalancer Service:**
    ```bash
    kubectl create deployment hello-minikube-balance --image=k8s.gcr.io/echoserver:1.4
    kubectl expose deployment hello-minikube-balance --type=LoadBalancer --port=8080
    ```
    Sau đó, chạy `minikube tunnel` trong một cửa sổ terminal mới để truy cập LoadBalancer qua External IP.
