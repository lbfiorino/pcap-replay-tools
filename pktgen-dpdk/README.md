# Pktgen (Packet Gen-erator)

Instalação da ferramenta Pktgen e do framework DPDK nos sistemas operacionais `Ubuntu 20.04` e `Ubuntu 18.04`.

**DPDK 19.11.6 :** [Site](http://core.dpdk.org/doc/), [Git](http://git.dpdk.org/)  
**Pktgen 21.02.0:** [Site](https://pktgen-dpdk.readthedocs.io/en/latest/index.html), [GitHub](https://github.com/pktgen/Pktgen-DPDK/)

**Linux Drivers:** https://doc.dpdk.org/guides/linux_gsg/linux_drivers.html  
**Bind NIC drivers**: https://dpdk-guide.gitlab.io/dpdk-guide/setup/binding.html  
**IOMMU:** https://dpdk-guide.gitlab.io/dpdk-guide/setup/iommu.html  

:warning: **\*\*\* NOTE \*\*\*** On a 10GB NIC if the transceivers are not attached the screen updates will go very slow.

## Requisitos
 - Sistema operacional: Ubuntu 20.04, Ubuntu 18.04
 - Linux Headers
 - gcc 4.9+
 - cmake
 - pkg-config
 - libpcap-dev
 - meson 0.47.1+
 - ninja-build
 - libnuma-dev
 - python pyelftools
 - python sphinx

### Atualizar o SO.
```bash
apt update
apt full-upgrade
reboot
```
### Instalar requisitos no Ubuntu 20.04
```bash
apt install build-essential cmake pkg-config libpcap-dev meson ninja-build libnuma-dev linux-headers-`uname -r` libhugetlbfs-bin
apt install python3-pip
python3 -m pip install pyelftools sphinx
```

### Instalar requisitos no Ubuntu 18.04
No Ubuntu 18.04 o meson precisa ser instalado via Python Pip, pois a versão do repositório do Ubuntu é mais antiga.  
Caso o meson esteja instalado via apt, remover e reiniciar antes de instalar via Python Pip.
```bash
apt purge meson
reboot
```
Após reiniciar:
```bash
apt install python3-pip
python3 -m pip install meson --upgrade
python3 -m pip install pyelftools sphinx
apt install build-essential cmake pkg-config libpcap-dev ninja-build libnuma-dev linux-headers-`uname -r`
```

### Instalar requisitos no CentOS 8
```bash
dnf group install "Development Tools"
dnf install kernel-devel autogen cmake pkgconf-pkg-config libpcap-devel meson ninja-build numactl-devel

# Dependências do pacote dpdk do repositório CentOS
dnf install ibacm infiniband-diags libibumad libibverbs libmnl-devel librdmacm pciutils rdma-core rdma-core-devel

python3 -m pip install pyelftools sphinx
```


## Configurar Huge pages  
[Documentação DPDK](https://doc.dpdk.org/guides/linux_gsg/sys_reqs.html#use-of-hugepages-in-the-linux-environment)  
[Documentação pktgen](https://pktgen-dpdk.readthedocs.io/en/latest/getting_started.html)

Adicionar no arquivo ` /etc/sysctl.conf` a linha abaixo.   
:warning: Atenção nos cálculos para não consumir toda a memória.
```bash
vm.nr_hugepages=256

## Pktgen Note:
# You can configure the vm.nr_hugepages=256 as required.
# In some cases making it too small will effect the performance of pktgen or cause it to terminate on startup.
```

Para Huge Pages maiores, adicionar no arquivo `/etc/default/grub` os parâmetros de kernel para hugepages no `GRUB_CMDLINE_LINUX` conforme abaixo.
```bash
# Para Huge Pages maiores
# Ex: 4 pages de 1G = 4GB
GRUB_CMDLINE_LINUX="default_hugepagesz=1G hugepagesz=1G hugepages=4"
```

Adicionar a linha abaixo no `/etc/fstab`.
```bash
huge /mnt/huge hugetlbfs defaults 0 0
```

Criar o diretório `/mnt/huge`.
```bash
mkdir /mnt/huge
chmod 777 /mnt/huge
```

Atualizar o GRUB e reiniciar.
```bash
update-grub
reboot
```

Para verificar a configuração.
```bash
# Utilitário hugeadm : lista os pontos de montagem hugetlbfs
apt install libhugetlbfs-bin
```
```
# Verificar o suporte a huge pages
grep -i huge /boot/config-<KERNEL_VERSION>
grep -i huge /proc/meminfo

# Lista os pontos de montagem hugetlbfs
hugeadm --list-all-mounts
```


## Instalar DPDK
**Versão:** 19.11 (Última versão no momento: 19.11.6)  
[DPDK Download](http://core.dpdk.org/download/)  

:warning: Nota:
> Versão 20.11 não gerou o RTE_TARGET: `make install T=x86_64-native-linux-gcc -j`

```bash
cd /root/
wget http://fast.dpdk.org/rel/dpdk-19.11.6.tar.xz
tar xvf dpdk-19.11.6.tar.xz

rm -fr /usr/local/lib/x86_64-linux-gnu # DPDK changed a number of lib names and need to clean up

cd dpdk-stable-19.11.6
meson build
cd build
ninja
ninja install
ldconfig  # make sure ld.so is pointing new DPDK libraries
```

### Criar RTE_TARGET
```bash
cd /root/dpdk-stable-19.11.6
make install T=x86_64-native-linux-gcc -j
```

### Exportar PKG_CONFIG_PATH
```bash
export PKG_CONFIG_PATH=/usr/local/lib/x86_64-linux-gnu/pkgconfig
```
Para persistir a variável PKG_CONFIG_PATH, adicionar a linha abaixo no arquivo `/etc/environment`:
```bash
PKG_CONFIG_PATH=/usr/local/lib/x86_64-linux-gnu/pkgconfig
```

## Instalar Pktgen 21.02.0

### Exportar variáveis RTE_SDK e RTE_TARGET
```bash
# RTE_SDK=<DPDKinstallDir>
# RTE_TARGET=x86_64-native-linux-gcc
export RTE_SDK=/root/dpdk-stable-19.11.6
export RTE_TARGET=x86_64-native-linux-gcc
```

Para persistir as variáveis, adicionar as linhas abaixo no arquivo `/etc/environment`:
```bash
RTE_SDK=/root/dpdk-stable-19.11.6
RTE_TARGET=x86_64-native-linux-gcc
```

### Build Pktgen:
```bash
cd /root
wget http://git.dpdk.org/apps/pktgen-dpdk/snapshot/pktgen-dpdk-pktgen-21.02.0.tar.xz
tar xvf pktgen-dpdk-pktgen-21.02.0.tar.xz

cd pktgen-dpdk-pktgen-21.02.0
make -j
```
Para facilitar a execução, criar o link simbólico `pktgen-dpdk`.
```bash
ln -s root/pktgen-dpdk-pktgen-21.02.0/usr/local/bin/pktgen /usr/local/bin/pktgen-dpdk
```

## Configurar interface de rede para DPDK (Dev Bind)

Em máquinas virtuais (driver `virtio`), utilizar o driver `vfio-pci` sem IOMMU para o bind da placa de rede.
```bash
# Se o módulo já estiver carregado (built-in module):
echo 1 > /sys/module/vfio/parameters/enable_unsafe_noiommu_mode

# Para carregar o módulo:
modprobe vfio enable_unsafe_noiommu_mode=1
```

### Persistir módulo no Ubuntu 20.04 (Kernel 5.4.0) com `buil-in module`:
Adicionar o parâmetro `vfio.enable_unsafe_noiommu_mode=1` no `GRUB_CMDLINE_LINUX` dentro do arquivo `/etc/default/grub`.
```bash
GRUB_CMDLINE_LINUX="vfio.enable_unsafe_noiommu_mode=1"
```
Atualizar o GRUB e reiniciar.
```bash
update-grub
reboot
```

### Persistir módulo no Ubuntu 18.04 (Kernel 4.15):
Editar o arquivo `/etc/modules` e adicionar o módulo `vfio`.
```bash
echo vfio >> /etc/modules
```
Criar o arquivo `/etc/modprobe.d/vfio-noiommu.conf` com o parâmetro `enable_unsafe_noiommu_mode` e reiniciar.
```bash
echo "options vfio enable_unsafe_noiommu_mode=1" > /etc/modprobe.d/vfio-noiommu.conf
reboot
```

### DPDK Bind NIC:

:warning: Criar o link simbólico para `python`, pois o DPDK não encontra o `python3`.
```bash
ln -s /usr/bin/python3 /usr/bin/python
```

Fazer o bind da interface de rede:
```bash
# Shutdown the interface
ifconfig ens4 down
# OR
ip link set dev ens4 down

# Listar as portas 
dpdk-devbind.py --status

# Bind Port with DPDK-compatible driver
# Porta diferente da LAN para não perder conexão
dpdk-devbind.py -b vfio-pci 0000:00:04.0
```

Para voltar ao normal:
```bash
# Bind with SO driver
dpdk-devbind.py -b virtio-pci 0000:00:04.0

# Activate de interface
ifconfig ens4 up
# OR
ip link set dev ens4 up
```

## Replay PCAP

Iniciar o Pktgen:  
Help: `./pktgen -h`
```bash
cd /root/pktgen-dpdk-pktgen-21.02.0/usr/local/bin/

# PARAMS
# -l 0-1 : Corelist - two lcores: core 0 monitoring, core 1 send packets
# -n 1 : One memory channel
# -- : Separate EAL parameters from Pktgen parameters
# -T : Color output
# -P : Promiscuous mode
# -s 0:<pcapfile> : PCAP packet stream file, 'P' is the port number
# -j : Enable jumbo frames of 9600 bytes

./pktgen -l 0-1 -n 1 -- -m 1.0 -T -P -s 0:/root/pcaps/smallFlows.pcap -j

##### A more typical commandline to start pktgen #####
# PARAMS
# -l 0-2 : Corelist - three lcores: core 0 monitoring, core 1 and 2 to send and receive packets
# -n 3 : Three memory channel
# --proc-type auto : Type of this process (primary|secondary|auto), auto = typical
# --socket-mem : Memory to allocate on CPU sockets (comma separated values)
# -v : Show dpdk version
# -- : Separate EAL parameters from Pktgen parameters
# -T : Color output
# -P : Promiscuous mode
# -s 0:<pcapfile> : PCAP packet stream file, 'P' is the port number
# -j : Enable jumbo frames of 9600 bytes
# -m "[1:2].0" : core 1 handles port 0 rx, core 2 handles port 0 tx

./pktgen -l 0-2 -n 3 --proc-type auto --socket-mem 2048 -v -- -T -P -s 0:/root/pcaps/smallFlows.pcap -j -m "[1:2].0"
```

No console do Pktgen:
```bash
# Mostra informações do PCAP
pcap show

# Define o número de pacotes a enviar, neste caso o número de pacotes do arquivo PCAP.
set 0 count <pcap_total_packets>

# Start send packets
start 0
```
:warning: Se não for definido o número de pacotes `set <port_number> count <num_packets>`, o Pktgen fica em loop até a execução do comando `stop <port_number>`.
