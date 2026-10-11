BAI 1: BAO CAO KY THUAT PHAN TICH KIEN TRUC CONTAINER VS VIRTUAL MACHINE

1. MUC TIEU VA BOI CANH DOANH NGHIEP

1.1. Boi canh
Trong qua trinh hien dai hoa ha tang cong nghe thong tin, cac doanh nghiep dang chuyen dich manh me tu mo hinh may chu ao truyen thong (Virtual Private Server - VPS / Virtual Machine - VM) sang nen tang dong goi ung dung bang Container (Docker). Bao cao ky thuat nay phan tich chuyen sau su khac biet ve mat kien truc giua ao hoa phan cung va ao hoa he dieu hanh, dong thoi lam ro cac co che cot loi cua Linux Kernel (Namespaces va Control Groups) giup Container dat duoc hieu nang cao va kha nang cach ly an toan tren may chu thuc te.

1.2. Muc tieu ky thuat
- Phan tich va so sanh chi tiet 5 tieu chi cot loi giua Virtual Machine va Container.
- Trinh bay nguyen ly hoat dong cua Linux Namespaces trong viec co lap tai nguyen tien trinh, mang va he thong tap tin.
- Trinh bay nguyen ly hoat dong cua Linux Control Groups (cgroups) trong viec quan ly va gioi han tai nguyen phan cung (CPU, RAM, I/O).
- Trich xuat va danh gia cac thong so thuc te tu lenh docker info tren he thong VPS Ubuntu 24.04.5 LTS (Noble Numbat).

2. BANG SO SANH KIEN TRUC CONTAINER VS VIRTUAL MACHINE

2.1. Tieu chi 1 - Dung luong Image (Image Size)
- Virtual Machine (VM): Dung luong rat lon, thuong tu 2GB den hang chuc GB. Ly do la moi VM bat buoc phai chua mot ban cai dat day du cua he dieu hanh khach (Guest OS) bao gom kernel rieng, system libraries, system binaries, device drivers va cac goi tien ich he thong.
- Container: Dung luong sieu nhe, thuong tu vai MB (vi du Alpine Linux ~5MB) den vai tram MB (Node.js/Java runtime). Ly do la Container khong chua kernel rieng ma dung chung Linux Kernel cua Host OS, chi dong goi application code va runtime dependencies toi thieu.

2.2. Tieu chi 2 - Thoi gian khoi dong (Startup Time)
- Virtual Machine (VM): Khoi dong cham, mat tu vai chuc giay den vai phut. VM phai trai qua quy trinh boot phan cung ao (Virtual BIOS/UEFI), nap kernel, khoi tao init system (systemd) va khoi dong toan bo he thong background services.
- Container: Khoi dong gan nhu tuc thi trong vai mili-giay den vai giay. Container thuc chat chi la mot process thong thuong tren Host OS duoc cach ly bang namespaces va cgroups, khong can trai qua quy trinh boot he dieu hanh.

2.3. Tieu chi 3 - Hao phi he dieu hanh va tai nguyen (OS Overhead)
- Virtual Machine (VM): Hao phi tai nguyen rat cao. Hypervisor (Type 1 hoac Type 2) phai gia lap toan bo phan cung (CPU, RAM, Network Card, Disk Controller) va duy tri mot he dieu hanh khach chay ngam lien tuc, tieu ton mot luong lon RAM va CPU ngay ca khi ung dung idle.
- Container: Hao phi gan bang 0 (near-zero overhead). Container Engine giao tiep truc tiep voi Host Kernel ma khong can qua lop gia lap phan cung, giup tan dung toi da hieu nang cua may chu vat ly.

2.4. Tieu chi 4 - Muc do co lap va bao mat (Isolation & Security)
- Virtual Machine (VM): Muc do co lap rat cao (Hardware-level isolation). Moi VM hoat dong nhu mot thiet bi vat ly rieng biet nho Hypervisor quan ly vung nho va CPU doc lap. Neu Guest OS bi loi hoac bi tan cong, su co duoc khoanh vung ben trong VM ma khong anh huong den Host OS hoac cac VM khac tren cung may chu.
- Container: Muc do co lap o muc do tien trinh he dieu hanh (OS-level process isolation). Do tat ca container tren host chia se chung mot kernel, neu co lo hong bao mat nghiem trong trong kernel, nguy co container breakout co the xay ra neu khong cau hinh cac lop bao ve bo sung nhu AppArmor, Seccomp, Capabilities va non-root user.

2.5. Tieu chi 5 - Kha nang mo rong va mat do trien khai (Scalability & Density)
- Virtual Machine (VM): Kha nang mo rong cham, mat do dong goi tren mot may chu vat ly thap do bi gioi han boi dung luong RAM va CPU cap phat co dinh cho Guest OS.
- Container: Kha nang mo rong cuc ky linh hoat va nhanh chong, mat do trien khai cao gap 5 den 10 lan so voi VM tren cung mot cau hinh phan cung. Container dac biet toi uu cho kien truc Microservices va cac he thong Cloud Native can auto-scaling theo tai thuc te.

3. CO CHE CO LAP KERNEL VA QUAN LY TAI NGUYEN CUA LINUX CONTAINER

3.1. Linux Namespaces (Co che co lap khong gian lam viec)
Namespaces la tinh nang cot loi cua Linux Kernel cho phan chia cac tai nguyen he thong toan cuc (global system resources) thanh cac khong gian cach ly rieng biet. Mot tien trinh nam trong mot namespace chi co the nhin thay va thao tac tren cac tai nguyen thuoc namespace do:
- PID Namespace (Process ID): Co lap cay tien trinh. Tien trinh chay ben trong container nhan PID 1 va dong vai tro la init process cua container, trong khi tren Host OS tien trinh nay mang mot PID thong thuong hoan toan khac.
- NET Namespace (Network): Co lap toan bo stack mang, bao gom card mang (virtual ethernet veth pair), bang dinh tuyen (routing table), luat tuong lua (iptables/nftables), socket va danh sach port dang mo. Cac container co the bind cung mot port (vi du port 80) ma khong bi xung dot.
- MNT Namespace (Mount Points): Co lap cay thu muc he thong tap tin. Container co mot he thong root filesystem (rootfs) doc lap, cho phep mount/unmount cac phan vung o dia ma khong tac dong den he thong tap tin cua Host OS.
- IPC Namespace (Inter-Process Communication): Co lap cac co che giao tiep giua cac tien trinh nhu System V IPC va POSIX message queues, shared memory, ngan chan cac tien trinh o cac container khac nhau truy cap vung nho chia se cua nhau.
- UTS Namespace (UNIX Timesharing System): Cho phep container so huu hostname va domain name rieng biet voi Host OS.
- USER Namespace (User and Group IDs): Cho phep anh xa user root (UID 0) ben trong container thanh mot user thong thuong (UID khong co quyen) tren Host OS, giam thieu toi da nguy co leo thang dac quyen khi container bi tan cong.

3.2. Linux Control Groups - cgroups (Co che quan ly va gioi han tai nguyen)
Control Groups (cgroups) la co che quan ly, do luong va phan bo tai nguyen phan cung cho tap hop cac tien trinh trong Linux:
- Gioi han bo nho RAM (Memory Limiting): Cho phep thiet lap nguong toi da ve dung luong RAM va Swap ma container duoc phep cap phat (thong qua memory.max hoac cgroup v1 memory.limit_in_bytes). Neu container vuot qua nguong RAM cho phep, Out-Of-Memory (OOM) Killer se cham dut tien trinh ben trong container de bao ve tinh on dinh cua toan bo may chu Host.
- Gioi han va chia se CPU (CPU Shares & CPU Quota): Dieu phoi chu ky xung nhip CPU thong qua CFS (Completely Fair Scheduler), cho phep chi dinh ti le % CPU hoac so luong Core CPU cu the ma container duoc phep su dung, ngan chan tinh trang mot ung dung chiem dung 100% CPU gay nghen he thong.
- Gioi han Block I/O: Kiem soat bang thong doc/ghi va so luong thao tac I/O tren giay (IOPS) toi o dia (blkio controller), ngan ngua cac container ghi log hoac doc file qua nhieu lam anh huong den hieu nang I/O cua cac ung dung khac.
- Cgroup Driver (systemd vs cgroupfs): systemd dong vai tro la Cgroup Driver chuan tren cac he dieu hanh Linux hien dai, dam bao su dong bo trong viec phan chia va quan ly cgroup giua he thong init cua OS va Container Runtime (runc / containerd).

4. TRICH XUAT THONG SO HE THONG TU DOCKER INFO VA DOCKER VERSION (SERVER THUC TE 103.20.102.126)

4.1. Lenh kiem tra phien ban Docker (docker version)
Lenh thuc thi tren may chu VPS bang user thuong duc_devops (khong dung sudo):
```bash
docker version
```
Ket qua thuc te:
```text
Client: Docker Engine - Community
 Version:           29.9.0
 API version:       1.56
 Go version:        go1.26.9
 Git commit:        f415da8
 Built:             Thu Oct  8 20:38:42 2026
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          29.9.0
  API version:      1.56 (minimum version 1.40)
  Go version:       go1.26.9
  Git commit:       a5b58c9
  Built:            Thu Oct  8 20:38:42 2026
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v2.4.1
  GitCommit:        f2551031d7276a770f65f98c9b52e57e7dad07e8
 runc:
  Version:          1.5.2
  GitCommit:        v1.5.2-0-g29dd3dc2
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

4.2. Lenh kiem tra thong tin he thong chi tiet (docker info)
Lenh thuc thi tren may chu VPS bang user thuong duc_devops:
```bash
docker info
```
Ket qua trich xuat cac thong so thuc te tu Terminal:
```text
Client: Docker Engine - Community
 Version:    29.9.0
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.38.0
    Path:     /usr/libexec/docker/cli-plugins/docker-buildx
  compose: Docker Compose (Docker Inc.)
    Version:  v5.6.0
    Path:     /usr/libexec/docker/cli-plugins/docker-compose

Server:
 Containers: 1
  Running: 0
  Paused: 0
  Stopped: 1
 Images: 1
 Server Version: 29.9.0
 Storage Driver: overlayfs
  driver-type: io.containerd.snapshotter.v1
 Logging Driver: json-file
 Cgroup Driver: systemd
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
 CDI spec directories:
  /etc/cdi
  /var/run/cdi
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: f2551031d7276a770f65f98c9b52e57e7dad07e8
 runc version: v1.5.2-0-g29dd3dc2
 init version: de40ad0
 Security Options:
  apparmor
   Profile: default
  seccomp
   Profile: builtin
  cgroupns
 Kernel Version: 6.8.0-31-generic
 Operating System: Ubuntu 24.04.5 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 2
 Total Memory: 1.875GiB
 Name: 10320102126523343
 ID: 8aa8d2a9-12ab-4752-b4c4-b5a317c76e97
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
  ::1/128
 Live Restore Enabled: false
 Firewall Backend: iptables
  EnableUserlandProxy: true
  UserlandProxyPath: /usr/bin/docker-proxy
```

4.3. Phan tich cac thong so trich xuat thuc te tu may chu
- Server Version: 29.9.0 (Docker Engine phien ban moi nhat, on dinh phuc vu moi truong san xuat).
- Cgroup Driver: systemd (Su dung cgroup driver dong bo voi he thong init systemd cua Ubuntu 24.04 LTS).
- Cgroup Version: 2 (Phien ban cgroup v2 voi co che phan cap don nhat, quan ly va gioi han tai nguyen toi uu hon).
- So luong Containers: 1 (Container hello-world da chay thanh cong o che do stopped).
- So luong Images: 1 (Image hello-world:latest duoc luu tru cuc bo).
- Storage Driver: overlayfs (Storage driver ket hop containerd snapshotter cho hieu nang layer read-write toi uu).
- Operating System & Kernel: Ubuntu 24.04.5 LTS voi Linux Kernel 6.8.0-31-generic.
- Cau hinh may chu vat ly/VPS: 2 CPUs, 1.875GiB RAM (dam bao day du tai nguyen de van hanh Docker Engine va cac ung dung Backend).

5. KET LUAN VA KHUYEN NGHI CHO DOANH NGHIEP

Chuyen doi tu VPS sang Container (Docker) mang lai loi ich vuot troi ve mat toi uu chi phi phan cung, toc do deploy, mat do chay ung dung va tinh dong nhat giua cac moi truong. Tuy nhien, doanh nghiep can luu y thiet lap gioi han cgroups cho tung container de tranh loi starvation va ap dung cac quy tac bao mat (non-root container, network policy, rootless mode) de dam bao tinh cach ly an toan cho he thong.
