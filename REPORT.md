BAI 1: BAO CAO KY THUAT PHAN TICH KIEN TRUC CONTAINER VS VIRTUAL MACHINE

1. MUC TIEU VA BOI CANH DOANH NGHIEP

1.1. Boi canh
Trong qua trinh hien dai hoa ha tang cong nghe thong tin, cac doanh nghiep dang chuyen dich manh me tu mo hinh may chu ao truyen thong (Virtual Private Server - VPS / Virtual Machine - VM) sang nen tang dong goi ung dung bang Container (Docker). Bao cao ky thuat nay phan tich chuyen sau su khac biet ve mat kien truc giua ao hoa phan cung va ao hoa he dieu hanh, dong thoi lam ro cac co che cot loi cua Linux Kernel (Namespaces va Control Groups) giup Container dat duoc hieu nang cao va kha nang cach ly an toan.

1.2. Muc tieu ky thuat
- Phan tich va so sanh chi tiet 5 tieu chi cot loi giua Virtual Machine va Container.
- Trinh bay nguyen ly hoat dong cua Linux Namespaces trong viec co lap tai nguyen tien trinh, mang va he thong tap tin.
- Trinh bay nguyen ly hoat dong cua Linux Control Groups (cgroups) trong viec quan ly va gioi han tai nguyen phan cung (CPU, RAM, I/O).
- Trich xuat va danh gia cac thong so thuc te tu lenh docker info tren he thong.

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

4. TRICH XUAT THONG SO HE THONG TU DOCKER INFO VA DOCKER VERSION

4.1. Lenh kiem tra phien ban Docker (docker version)
Lenh thuc thi:
```bash
docker version
```
Ket qua thuc te:
```text
Client: Docker Engine - Community
 Version:           27.3.1
 API version:       1.47
 Go version:        go1.22.7
 Git commit:        ce12230
 Built:             Fri Sep 20 11:41:00 2024
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          27.3.1
  API version:      1.47 (minimum version 1.24)
  Go version:       go1.22.7
  Git commit:       4123218
  Built:            Fri Sep 20 11:41:00 2024
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.7.22
  GitCommit:        7f7fdf5fed64eb6a7caf99b3e12efcf9d60e311c
 runc:
  Version:          1.1.14
  GitCommit:        v1.1.14-0-g2c9f560
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

4.2. Lenh kiem tra thong tin he thong chi tiet (docker info)
Lenh thuc thi:
```bash
docker info
```
Ket qua trich xuat cac thong so co ban:
```text
Client: Docker Engine - Community
 Version:    27.3.1
 Context:    default
 Debug Mode: false

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 1
 Server Version: 27.3.1
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Using metacopy: false
 Logging Driver: json-file
 Cgroup Driver: systemd
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
 Operating System: Ubuntu 22.04.5 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 4
 Total Memory: 7.742GiB
 Docker Root Dir: /var/lib/docker
 Live Restore Enabled: false
```

4.3. Phan tich cac thong so trich xuat
- Server Version: 27.3.1 (Docker Engine phien ban on dinh phuc vu moi truong san xuat).
- Cgroup Driver: systemd (Su dung cgroup driver dong bo voi he thong quan ly service systemd cua Ubuntu 22.04).
- Cgroup Version: 2 (Phien ban cgroup v2 moi nhat voi co che phan cap don nhat va quan ly tai nguyen toi uu hon).
- So luong Containers: 0 (Chua co container nao dang chay tren host).
- So luong Images: 1 (He thong da co 1 image cuc bo duoc luu tru).
- Storage Driver: overlay2 (Storage driver mac dinh toi uu cho hieu nang layer read-write tren Linux).

5. KET LUAN VA KHUYEN NGHI CHO DOANH NGHIEP

Chuyen doi tu VPS sang Container (Docker) mang lai loi ich vuot troi ve mat toi uu chi phi phan cung, toc do deploy, mat do chay ung dung va tinh dong nhat giua cac moi truong. Tuy nhien, doanh nghiep can luu y thiet lap gioi han cgroups cho tung container de tranh loi starvation va ap dung cac quy tac bao mat (non-root container, network policy, rootless mode) de dam bao tinh cach ly an toan cho he thong.
