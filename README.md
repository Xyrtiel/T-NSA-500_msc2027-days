# JOUR 1 : T-NSA-DAY01_Tasks
# 💻 Installation de Debian 12 "Bookworm" avec partitions manuelles

Ce guide détaille l'installation de Debian sans interface graphique avec des partitions manuelles, conformément aux spécifications demandées pour un projet DevOps ou tout autre contexte éducatif.

---

## 🚀 Prérequis

Avant de commencer, assurez-vous d'avoir :
- [VirtualBox](https://www.virtualbox.org/) installé sur votre machine.
- Une ISO de **Debian 12 "Bookworm"** (téléchargeable sur [Debian.org](https://www.debian.org/)).

---

# Etape 1 : Préparation de la machine virtuelle

Créer une machine virtuelle avec :
1. **20 Go de disque dur virtuel.**
2. **1 Go de RAM.**
3. Debian installé **en anglais**, sans interface graphique (GUI).
4. Partitions manuelles :
   - `swap`
   - `/` (root)
   - `/home`
   - `/var`

---

## 📝 Étapes détaillées

### 1️⃣ **Création de la machine virtuelle**
1. Ouvrez VirtualBox et cliquez sur **New**.
2. Configurez comme suit :
   - **Nom :** `Debian12-VM` (ou tout autre nom).
   - **Type :** Linux.
   - **Version :** Debian (64-bit).
   - **RAM :** 1024 Mo (1 Go).
   - **Disque dur :** Créez un nouveau disque de 20 Go.
3. Cliquez sur **Create** pour finaliser.

---

### 2️⃣ **Démarrage de l’installation**
1. Insérez l'ISO Debian en cliquant sur **Settings > Storage > Empty > Add Disk > Choisissez l'ISO.**
2. Démarrez la machine et choisissez **Graphical Install**.
3. **Langue :** Sélectionnez **English**.
4. **Région :** Sélectionnez votre région (ou United States).
5. **Keymap :** US (ou votre préférence).
6. Suivez les étapes jusqu'à **Partition Disks**.

---

### 3️⃣ **Configuration des partitions manuelles**
Lorsque vous arrivez à l'écran **Partition Disks**, choisissez :
1. **Manual** pour le partitionnement.

---

#### **Créer les partitions manuellement**
Vous devez configurer les partitions comme suit :

| Partition | Taille       | Type      | Point de montage | Système de fichiers |
|-----------|--------------|-----------|------------------|----------------------|
| Swap      | 2 GB         | Primaire  | -                | swap area           |
| /         | 8 GB         | Primaire  | /                | ext4                |
| /home     | 7 GB         | Primaire  | /home            | ext4                |
| /var      | Reste (~3 GB)| Primaire  | /var             | ext4                |

---

[📄 Configuration réalisée sur la VM étape par étape](T-NSA_Image01-Installation.pdf)

---

# Etape 2 : Mise en place des partitions

#### **Étapes pour chaque partition**
1. **Sélectionnez `FREE SPACE`** et appuyez sur **Entrée**.
2. Choisissez **Create a new partition**.
3. Entrez la taille requise (ex. 2 GB pour swap).
4. Sélectionnez **Primary**.
5. Placez la partition **At the beginning**.
6. Configurez chaque partition :
   - Pour `swap` :
     - Changez **Use as** en **swap area**.
   - Pour `/`, `/home` et `/var` :
     - Changez le **Mount point** en fonction (`/`, `/home`, ou `/var`).
     - Gardez le système de fichiers en **ext4 journaling**.
7. Cliquez sur **Done setting up the partition**.

---

### 4️⃣ **Finalisation des partitions**
1. Une fois toutes les partitions configurées, choisissez **Finish partitioning and write changes to disk**.
2. Confirmez avec **Yes** pour appliquer les changements.

---

### 5️⃣ **Finaliser l'installation**
1. Suivez les étapes suivantes :
   - Configurez un **mot de passe root**.
   - Créez un **compte utilisateur standard**.
2. Lorsque demandé, **ne pas installer de GUI (interface graphique)**.
3. À la fin de l'installation, redémarrez la machine.

---

### 🎯 Vérification des partitions
Une fois connecté à votre machine, vous pouvez vérifier vos partitions avec la commande :

```bash
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   20G  0 disk
├─sda1   8:1    0    2G  0 part [SWAP]
├─sda2   8:2    0    8G  0 part /
├─sda3   8:3    0    7G  0 part /home
└─sda4   8:4    0    3G  0 part /var
```

La machine est prête à être utilisée. 

# Tâche 3 : Mise en place des utilisateurs

# Créer le groupe H2G2 avec GID 42400
sudo groupadd -g 42400 H2G2

# Ajouter l'utilisateur marvin au groupe H2G2
sudo usermod -aG H2G2 marvin

# Créer l'utilisateur zaphod avec les caractéristiques spécifiées
sudo useradd -m -d /home/zaphod -c "Zaphod Beeblebrox" -u 4200 -g 42400 zaphod

# Définir le mot de passe de l'utilisateur zaphod
echo "zaphod:ZappyBibicy" | sudo chpasswd

# Créer le dossier /home/HeartOfGold et le lier au groupe H2G2
sudo mkdir /home/HeartOfGold
sudo chown :H2G2 /home/HeartOfGold
sudo chmod 770 /home/HeartOfGold

# Task 04 - SSH Configuration

# Installer le service SSH
sudo apt update
sudo apt install -y openssh-server

# Modifier la configuration du fichier /etc/ssh/sshd_config
sudo sed -i 's/^#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo sed -i 's/^#PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config
sudo sed -i 's/^#Port 22/Port 4242/' /etc/ssh/sshd_config

# Redémarrer le service SSH pour appliquer les changements
sudo systemctl restart ssh

# Task 05 - Restrict SSH for zaphod

# Ajouter une règle pour interdire l'accès SSH à l'utilisateur zaphod
echo "DenyUsers zaphod" | sudo tee -a /etc/ssh/sshd_config

# Redémarrer le service SSH pour appliquer les changements
sudo systemctl restart ssh

# Task 06 - Install and Configure Fail2Ban

# Installer Fail2Ban
sudo apt install -y fail2ban

# Configurer Fail2Ban en modifiant /etc/fail2ban/jail.local
sudo tee /etc/fail2ban/jail.local > /dev/null <<EOF
[sshd]
enabled = true
port = 4242
maxretry = 3
bantime = 30m
findtime = 5m
EOF

# Redémarrer Fail2Ban pour appliquer les changements
sudo systemctl restart fail2ban

# Task 07 - Configure iptables

# Créer les règles iptables dans /etc/iptables/rules.v4
sudo tee /etc/iptables/rules.v4 > /dev/null <<EOF
*filter

# Accepter les connexions SSH (port 4242)
-A INPUT -p tcp --dport 4242 -j ACCEPT
-A OUTPUT -p tcp --dport 4242 -j ACCEPT

# Autoriser les connexions HTTP/HTTPS sortantes
-A OUTPUT -p tcp --dport 80 -j ACCEPT
-A OUTPUT -p tcp --dport 443 -j ACCEPT

# Autoriser les requêtes DNS sortantes
-A OUTPUT -p udp --dport 53 -j ACCEPT

# Bloquer toutes les autres connexions
-A INPUT -j DROP
-A OUTPUT -j DROP

COMMIT
EOF

# Charger les règles iptables
sudo iptables-restore < /etc/iptables/rules.v4

# Installer le paquet pour rendre les règles iptables persistantes au démarrage
sudo apt install -y iptables-persistent

# Sauvegarder les règles pour qu'elles soient appliquées au reboot
sudo netfilter-persistent save
sudo netfilter-persistent reload


# JOUR 2 : T-NSA-DAY02_Tasks

Tâche 1 : Même VM mis à part qu'il ne faut pas sélectionner "Graphical Install"

## Task 02 - Network Name

1. **Change the name of the Gateway machine** to `vm-gateway`:
    ```bash
    sudo hostnamectl set-hostname vm-gateway
    ```

2. **Check that the hostname has been set correctly**:
    ```bash
    hostname
    ```

---

## Task 03 - Packages Installation

1. **Install the following packages** on `vm-gateway`:
    - `tcpdump`
    - `isc-dhcp-server`
    - `bind9`
    - `nmap`
    - `netstat`

    To install these packages, run the following:
    ```bash
    sudo apt update
    sudo apt install tcpdump isc-dhcp-server bind9 nmap net-tools -y
    ```

---

## Task 04 - Configure DHCP

1. **Configure ISC DHCP Server** on `vm-gateway`:
    - **Network:** `192.168.42.0/24`
    - **Netmask:** `255.255.255.0`
    - **Private Interface IP:** `192.168.42.254` (not in DHCP mode)
    - **DHCP Range:** `192.168.42.100-192.168.42.150` (50 hosts)

    Open the DHCP configuration file:
    ```bash
    sudo nano /etc/dhcp/dhcpd.conf
    ```

    Add or modify the following lines:
    ```bash
    # Global configuration
    authoritative;
      
    subnet 192.168.42.0 netmask 255.255.255.0 {
    range 192.168.42.100 192.168.42.150;  
    option routers 192.168.42.254;        
    option domain-name-servers 8.8.8.8, 8.8.4.4; 
    option domain-name "epi42.lan";       
    }
    ```

2. **Set static IP for the private interface** (`eth1` in this example):
    ```bash
    sudo nano /etc/network/interfaces
    ```

    Add the following configuration:
    ```bash
    iface eth1 inet static
        address 192.168.42.254
        netmask 255.255.255.0
    ```

3. **Restart the DHCP service** to apply the configuration:
    ```bash
    sudo systemctl restart isc-dhcp-server
    ```

4. **Test DHCP on the client machine** (`vm-client`) to make sure it receives an IP in the specified range:
    - Check the IP address with:
    ```bash
    ip a
    ```

---

## Task 05 - Router Configuration

1. **Configure the router on `vm-gateway` using iptables**:
    - **Bridge interface** should be set in DHCP mode.
    - **Private network interface** should be on the same network as `vm-client`.

2. **Enable IP forwarding** by editing `/etc/sysctl.conf`:
    ```bash
    sudo nano /etc/sysctl.conf
    ```

    Uncomment or add the line:
    ```bash
    net.ipv4.ip_forward=1
    ```

    Apply the change:
    ```bash
    sudo sysctl -p
    ```

3. **Set iptables rules** in `/etc/iptables/rules.v4` to allow the following:
    - Allow all **output traffic** from the private network.
    - Allow **DHCP** traffic to pass from `vm-client`.

    Edit the iptables file:
    ```bash
    sudo nano /etc/iptables/rules.v4
    ```

    Add the following rules:
    ```bash
    *filter
    # Allow DHCP
    -A INPUT -p udp --dport 67 -j ACCEPT
    -A OUTPUT -p udp --dport 67 -j ACCEPT
    
    # Allow all output from the private network
    -A INPUT -s 192.168.42.0/24 -j ACCEPT
    -A OUTPUT -d 192.168.42.0/24 -j ACCEPT
    
    # Block all other traffic
    -A INPUT -j DROP
    -A OUTPUT -j DROP
    COMMIT
    ```

4. **Save and apply the iptables rules**:
    ```bash
    sudo iptables-save > /etc/iptables/rules.v4
    ```

---

## Task 06 - DNS Configuration

1. **Configure BIND9 DNS** on `vm-gateway`:

    - Open the DNS configuration file for editing:
    ```bash
    sudo nano /etc/bind/named.conf.local
    ```

    Add the following zone configuration for the domain `epi42.lan`:
    ```bash
    zone "epi42.lan" {
        type master;
        file "/etc/bind/db.epi42.lan";
    };
    ```

2. **Create the DNS zone file**:
    ```bash
    sudo nano /etc/bind/db.epi42.lan
    ```

    Add the following records:
    ```bash
    $TTL    604800
    @       IN      SOA     gateway.epi42.lan. root.gateway.epi42.lan. (
                          2024010101 ; Serial
                          604800     ; Refresh
                          86400      ; Retry
                          2419200    ; Expire
                          604800 )   ; Minimum TTL
    ;
    @       IN      NS      gateway.epi42.lan.
    gateway IN      A       192.168.42.254
    dhcp    IN      CNAME   gateway.epi42.lan.
    ```

3. **Check BIND9 configuration**:
    ```bash
    sudo named-checkconf
    sudo named-checkzone epi42.lan /etc/bind/db.epi42.lan
    ```

4. **Restart BIND9 to apply changes**:
    ```bash
    sudo systemctl restart bind9
    ```

5. **Test the DNS resolution** from `vm-client`:
    ```bash
    nslookup gateway.epi42.lan
    nslookup dhcp.epi42.lan
    ```

---

## Conclusion

Tu as maintenant configuré avec succès un **serveur DHCP**, un **routeur** via `iptables`, et un **serveur DNS** avec BIND9 pour les machines dans le réseau privé. Le client (`vm-client`) peut obtenir une adresse IP via DHCP et résoudre les noms via DNS.

---





