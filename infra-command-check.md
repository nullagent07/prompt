1️⃣ GPU (самое важное)
nvidia-smi


И:

nvidia-smi -q | grep -E "Product Name|FB Memory Usage|BAR1 Memory Usage|Compute Mode"


И:

nvidia-smi topo -m

2️⃣ CPU
lscpu

3️⃣ RAM
free -h


И:

cat /proc/meminfo | grep MemTotal

4️⃣ Диск
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT


И:

df -h

5️⃣ ОС
uname -a


И:

cat /etc/os-release

6️⃣ Docker
docker --version

docker info | grep -E "Cgroup|Runtimes"

7️⃣ ulimit (важно для vLLM)
ulimit -a

8️⃣ Swap
swapon --show

Если коротко (минимум для расчёта моделей)

Если хочешь отправить только самое критичное, достаточно:

nvidia-smi
lscpu
free -h
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT
