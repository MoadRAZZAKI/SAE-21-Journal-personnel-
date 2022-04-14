 ### Scéance 1 : configuration du routeur sur GNS3 : 

 durant cette première scéance de SAE j'ai configuré mon routeur cisco sur GNS3, j'ai commencé d'abord par configurer les adresses IP des interfaces de mon routeur : 

> d'abord pour la patte interne du routeur, j'ai décidé de lui donner une adresse IP statiquement en utilisant ces commandes :


        configure terminal 
        interface fastEthernet 0/0
        ip address 192.168.0.1 255.255.255.0 
        no shutdown
        end
        wr mem 

> ensuite pour la patte externe de mon routeur, j'ai pensé à utiliser le service dhcp pour lui attribuer une adresse IP quand il se connecte au cloud (le réseau de la salle )



        configure terminal 
        interface fastEthernet 0/1
        ip address dhcp 
        no shutdown 
        end 
        wr mem

> par la suite,j'ai deployé un service dhcp dans mon routeur pour attribuer automatiquement aux PC de mon réseau interne une adresse IP , alors j'ai utilisé ces commandes : 

        configure terminal 
        ip dhcp pool LAN
        default-router 192.168.0.1
        dns-server 8.8.8.8
        network 192.168.0.0 255.255.255.0
        end
        wr mem

> et enfin j'ai mis en place le protocole OSPF dans mon routeur : 



        configure terminal
        routeur ospf 1
        network 192.168.0.0 0.0.255.255 area 0
        network 10.214.0.0 0.0.255.255 area 0
        end
        wr mem
        
        

### Séance du 08/04/2022 (TD encadrée) configuration des VLAN sur le switch:
---

* Pendant cette séance nous avons configuré le NAT sur le routeur
* Configuration de la patte interne du routeur

        conf t
        interface FastEthernet 0/0
        ip nat inside
        exit

* Configuration externe du routeur 

        conf t
        interface FastEthernet 0/1
        ip nat outside
        exit

* Configuration de l'ACL contenant une liste des adresses source interne qui seront traduite

        access-list 1 permit 192.168.0.0 0.0.255.255

* Configuration du pool d'adresses IP globale

        ip nat pool POOL 10.213.0.0 10.213.255.254 netmask 255.255.0.0

* Activation du NAT dynamique

        ip nat inside source list 1 pool POOL
        exit
        wr mem
        
        
## Configuration des vlan :



* Vlan 1 :  


        conf t
        interface FastEthernet 0/0.1
        encapsulation dot1Q 10
        ip address 192.168.10.1 255.255.255.248
        exit
   
   
* Vlan 2:  


        conf t
        interface FastEthernet 0/0.2
        encapsulation dot1Q 20
        ip address 192.168.20.1 255.255.255.248
        exit
  
  
* Vlan 3  


        conf t
        interface FastEthernet 0/0.3
        encapsulation dot1Q 3
        ip address 192.168.30.1 255.255.255.248
        exit
        

## Configuration du pool DHCP pour chaque vlan :


* Vlan 1 :  


        ip dhcp pool VLAN10
        network 192.168.10.0 255.255.255.248
        default-router 192.168.10.1
        exit
   
   
* Vlan 2:  


        ip dhcp pool VLAN20
        network 192.168.20.0 255.255.255.248
        default-router 192.168.20.1
        exit
  
  
* Vlan 3  


        ip dhcp pool VLAN20
        network 192.168.20.0 255.255.255.248
        default-router 192.168.30.1
        exit
        
        
        
## configuration des ACL pour plus de sécurité : 

* tout d'abord j'ai nommé la régle ACL et ensuite j'ai définit ce qu’elle doit filtrer. 

        R1(config)#ip access-list extended firsttothird ( du vlan 10 vers le vlan 30 )
        R1(config-ext-nacl)#deny ip 192.168.10.0 0.0.0.7 192.168.30.0 0.0.0.7
        R1(config-ext-nacl)#permit ip any any
        
* ensuite pour appliquer la règle sur une interface j'ai utilise la syntaxe suivante : 

        R1(config)#interface fastEthernet 0/0.1
        R1(config-subif)#ip access-group firsttothird in
        R1(config-subif)#ip access-group firsttothird out




## configuration du protocole SSH sur le routeur : 

* On rajoute, en utilisant cette commande, les privilèges à l'utilisateur admin du routeur 

        username admin privile 15 pass admin

* Cette commande permet d'effacer les clefs RSA, en effet, dû à une erreur de ma part, le routeur refusait les connexions des utilisateurs de mon parc informatique 

        crypto key zeroize rsa
        
* La commande suivante permet de créer une nouvelle clef. Je dois préciser que la taille de la clef que j'ai choisi est 1024 bits

        crypto key generate rsa 
        
        
* Le terme "vty" signifie Virtual teletype. VTY est un port virtuel qui est utilisé pour obtenir un accès Telnet ou SSH à l'appareil.VTY est uniquement utilisé pour les connexions entrantes à l'appareil. Ces connexions sont toutes virtuelles et aucun matériel ne leur est associé. En fait, nous pouvons avoir des ports de connexion jusqu'à 16 qui tournent simultanaiment (0 - 15)
 
        line vty 0 15
        
        
        
* La commande “transport input ssh” définit quel protocole a le droit d'utiliser ces lignes vty. Par défaut, tous les protocoles ont le droit dont telnet et ssh. Cette commande permet de restreindre en précisant que seul ssh a le droit d'utiliser les lignes vty.
  
        transport input ssh
        
* Login local, signifie que l'authentification utilise des informations d'identification configurées localement en utilisant la commande "username <admin> privilege <15> password 0 admin" en mode de configuration globale.

 
        login local
 
 
 
 * Ensuite sur chaque PC des trois VLAN j'ai modifié le fichier /etc/ssh/ssh_config est j'ai rajouter les lignes :
 
 
       KexAlgorithms +diffie-hellman-group1-sha1,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group14-sha1
      Ciphers aes256-cbc,aes128-ctr,aes192-ctr,aes256-ctr,aes128-cbc,3des-cbc
 
 
 * En effet, la ligne KexAlgorithms renseigne les méthodes d'échange de clés qui sont utilisées pour générer des clés par connexion . Tandis que la ligne Ciphers permet de configurer les ciphers utilisés pour chiffrer la connexion.
