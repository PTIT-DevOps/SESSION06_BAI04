# BT04 - Quản lý tiến trình nền với nohup và tín hiệu Kill (Background Process and Kill Signals)

## 1. Tạo shell script loop-monitor.sh

```bash
devops@0310263186:~$ cat << 'EOF' > loop-monitor.sh
> #!/bin/bash
> while true; do
>     echo "System time: $(date)" >> /tmp/monitor.log
>     sleep 5
> done
> EOF

devops@0310263186:~$ chmod +x loop-monitor.sh
devops@0310263186:~$ ls -l loop-monitor.sh
-rwxr-xr-x 1 devops devops 83 Oct  6 09:31 loop-monitor.sh
```

## 2. Khởi chạy script ở chế độ nền độc lập

```bash
devops@0310263186:~$ nohup ./loop-monitor.sh > /dev/null 2>&1 &
[1] 4382

devops@0310263186:~$ pgrep -f loop-monitor.sh
4382
```

## 3. Giám sát tiến trình

```bash
devops@0310263186:~$ ps aux | grep loop-monitor.sh
devops      4382  0.0  0.1   7372  3500 pts/0    S    09:31   0:00 /bin/bash ./loop-monitor.sh
devops      4401  0.0  0.0   6432  2560 pts/0    S    09:31   0:00 grep --color=auto loop-monitor.sh
```

## 4. Kiểm tra tệp log đang được ghi liên tục

```bash
devops@0310263186:~$ tail -n 10 /tmp/monitor.log
System time: Tue Oct  6 09:31:25 UTC 2026
System time: Tue Oct  6 09:31:30 UTC 2026
System time: Tue Oct  6 09:31:35 UTC 2026
System time: Tue Oct  6 09:31:40 UTC 2026
System time: Tue Oct  6 09:31:45 UTC 2026
System time: Tue Oct  6 09:31:50 UTC 2026
System time: Tue Oct  6 09:31:55 UTC 2026
System time: Tue Oct  6 09:32:00 UTC 2026
System time: Tue Oct  6 09:32:05 UTC 2026
System time: Tue Oct  6 09:32:10 UTC 2026
```

## 5. Kiểm tra tiến trình vẫn chạy sau khi ngắt phiên Terminal

Ngắt kết nối SSH hiện tại rồi đăng nhập lại:

```bash
devops@0310263186:~$ exit
logout

thinh@local:~$ ssh devops@160.30.160.103
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.8.0-31-generic x86_64)
Last login: Tue Oct  6 09:32:20 2026 from 115.76.49.152
devops@0310263186:~$ pgrep -f loop-monitor.sh
4382

devops@0310263186:~$ tail -n 3 /tmp/monitor.log
System time: Tue Oct  6 09:32:55 UTC 2026
System time: Tue Oct  6 09:33:00 UTC 2026
System time: Tue Oct  6 09:33:05 UTC 2026
```

PID vẫn là `4382` và log vẫn được ghi sau khi đăng nhập lại.

## 6. Tắt tiến trình bằng tín hiệu SIGTERM (15)

```bash
devops@0310263186:~$ kill -15 4382
devops@0310263186:~$ pgrep -f loop-monitor.sh

devops@0310263186:~$ ps aux | grep loop-monitor.sh
devops      4420  0.0  0.0   6432  2560 pts/1    S    09:33   0:00 grep --color=auto loop-monitor.sh

devops@0310263186:~$ tail -n 2 /tmp/monitor.log
System time: Tue Oct  6 09:33:00 UTC 2026
System time: Tue Oct  6 09:33:05 UTC 2026
```

Sau `kill -15`, PID `4382` không còn trong danh sách tiến trình và log ngừng ghi thêm.

## 7. Trường hợp SIGTERM không có tác dụng, dùng SIGKILL (9)

Tiến trình bỏ qua SIGTERM (đã `trap` tín hiệu TERM):

```bash
devops@0310263186:~$ nohup bash -c 'trap "" TERM; while true; do sleep 5; done' > /dev/null 2>&1 &
[1] 4436

devops@0310263186:~$ ps -o pid,cmd -p 4436
    PID CMD
   4436 bash -c trap "" TERM; while true; do sleep 5; done

devops@0310263186:~$ kill -15 4436
devops@0310263186:~$ ps -o pid,cmd -p 4436
    PID CMD
   4436 bash -c trap "" TERM; while true; do sleep 5; done
```

SIGTERM bị bỏ qua, buộc phải dùng SIGKILL:

```bash
devops@0310263186:~$ kill -9 4436
devops@0310263186:~$ ps -o pid,cmd -p 4436
    PID CMD
devops@0310263186:~$
```
