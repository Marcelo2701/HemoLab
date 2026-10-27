**Homelab: Servidor de Ficheiros Samba em LXC (Proxmox VE)**

Este repositório documenta o processo de implantação, configuração e resolução de problemas para a criação de um HomeLab altamente eficiente e estável, com um servidor Samba já implementado, alojado num container Debian 12 LXC dentro do ecossistema Proxmox VE.

O objetivo principal deste servidor Samba, foi particionar o dispositivo físico do homelab (volume LVM de 1TB) em uma partição de 180GB e disponibilizá-lo como um disco de rede persistente (`Z:`) no Windows.

---

**Arquitetura e Especificações**

Hypervisor: Proxmox VE
Tipo de Instância: LXC Container (ID: `100`)
Sistema Operacional: Debian 12 (Bookworm)
Armazenamento: 180 GiB (LVM dedicado montado em `/srv/samba/dados`)
Rede: 
  * Interface Bridge (`vmbr0`)
  * IP Estático: `192.168.15.8/24` (com reserva/pool DHCP ajustado no router)
  * Gateway: `192.168.15.1`
Protocolo de Partilha: Samba / SMB (Porta `445` TCP)
Cliente: Windows 11 Pro (Unidade Mapeada `Z:`)

---

**Passo a Passo de Implementação**

1. Preparação do Armazenamento e Diretório
Para evitar conflitos com a pasta de sistema `lost+found` do ext4/LVM, criou-se uma pasta dedicada para a partilha:

**bash**
mkdir -p /srv/samba/dados

Criar utilizador de sistema
adduser homelab

Definir o proprietário e aplicar permissões recursivas
chown -R homelab:homelab /srv/samba/dados
chmod -R 775 /srv/samba/dados

**Definir uma senha para esse usuário**
smbpasswd -a homelab

Em seguida, editou-se o ficheiro de configuração /etc/samba/smb.conf:
[global]
   workgroup = WORKGROUP
   server string = Samba Server Homelab
   security = user
   map to guest = Bad User

[LVM_SDB]
   path = /srv/samba/dados
   browseable = yes
   writable = yes
   guest ok = no
   valid users = homelab
   create mask = 0775
   directory mask = 0775

   Por fim, alterou-se para o utilizador root para aplicar e reiniciar o serviço:
   su -
   systemctl restart smbd

**Mapeamento no cliente Windows**
net use z: \\192.168.15.8\LVM_SDB /user:homelab

<img width="536" height="183" alt="image" src="https://github.com/user-attachments/assets/09301acd-9274-4e9a-a7f5-fc4ea04622ae" />


**Interface do PROXMOX**

No canto superior esquerdo, temos o container LXC rodando o servidor Samba e abaixo temos os discos já montados, tanto o do sistema (Proxmox)
quanto o disco dedicado para serviços (HD de 1TB)

<img width="1903" height="908" alt="image" src="https://github.com/user-attachments/assets/66d0ca13-e124-428f-8144-57c7efcfc01c" />

**Estatísticas gerais**

<img width="1908" height="907" alt="image" src="https://github.com/user-attachments/assets/ef03c428-7a06-46f6-8c96-5218846d6e3c" />


**Próximos passos**

Após a criação do primeiro container sendo um serviço de armazenamento de arquivos (servidor Samba), as próxmias atualizações envolve a criação
de novos serviços, como por exemplo, um servidor de bloqueio de anúncios via DNS.




