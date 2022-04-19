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
        
        
        
* Vlan 4  


        conf t
        interface FastEthernet 0/0.4
        encapsulation dot1Q 4
        ip address 192.168.40.1 255.255.255.248
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


        ip dhcp pool VLAN30
        network 192.168.30.0 255.255.255.248
        default-router 192.168.30.1
        exit
        
* Vlan 4 

        ip dhcp pool VLAN40
        network 192.168.40.0 255.255.255.248
        default-router 192.168.40.1
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



* voici les options que j'ai ajouté au routeur pour configurer le service ssh :

> La commande suivante permet d'activer le service ssh sur le routeur :
 
        ip ssh version 2

> les évènements associés aux connexions ssh sont enregistrés dans les logs: 
        
        ip ssh logging events
        
        
> Un timeout de 60 secondes est ajouté en cas d'inactivité durant l'authentification.

        ip ssh time-out 60

> On laisse trois essais pour la connexion au routeur. Suite à ces essais, la connexion est fermée.
        
        ip ssh authentication-retries 3



> ensuite j'ai ajouté cette option pour crypter le mot de passe:

        service password-encryption



* On crée un utilisateur admin avec le mot de passe admin

        username admin password 0 admin



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
 
 --------------------------------------------------------------------------------------
 
 * Ensuite sur chaque PC des trois VLAN j'ai modifié le fichier /etc/ssh/ssh_config est j'ai rajouter les lignes :
 
 
       KexAlgorithms +diffie-hellman-group1-sha1,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group14-sha1
       Ciphers aes256-cbc,aes128-ctr,aes192-ctr,aes256-ctr,aes128-cbc,3des-cbc
 
 
 * En effet, la ligne KexAlgorithms renseigne les méthodes d'échange de clés qui sont utilisées pour générer des clés par connexion . Tandis que la ligne Ciphers permet de configurer les ciphers utilisés pour chiffrer la connexion.
 
 
 
 
 
 ------------------------------------------------------------------------------------
 
 
 
 * Par la suite, je me suis rendu sur le fichier /etc/ssh/sshd_config et j'ai décommenté les lignes suivantes pour permettre aux utilisateurs de mon réseau interne de faire des connexions à distance en utilisant le service ssh.
 
 
       PubkeyAuthentication yes
       PasswordAuthentication yes
       ChallengeResponseAuthentication no
 
 * Il faut noter qu'après chaque modification des fichiers ssh, soit ssh_config ou sshd_config, il est absolument nécessaire d'éxecuter la commande suivante pour que les modifications puissent être prises en compte, et puis vérifier le status du service ssh sur la machine configurée :
 
 
       service ssh restart
       service ssh status
       
 * En fait, la commande "service" et "systemctl" fonctionnent de la même manière, la seule différence ici est la compatibilité de la commande avec les utilitaires qui sont responsables du fonctionnement du système d'exploitation de mes machines. Après des recherches que j'ai fait sur internet, j'ai trouvé que la commande service permet d'exécuter le script d'initialisation de SystemV qui est utilisé par les anciennes distributions Linux.
       

 
## Restreindre les utilisateurs qui peuvent se connecter en SSH :
   
 
 > d'abord on ouvre le fichier sshd_config.
 
       nano /etc/ssh/sshd_config
 
 
> ensuite on ajoute ces lignes à la fin de notre fichier.
 
       DenyUsers test
       DenyGroups test

       DenyUsers root
       DenyGroups test

       PermitRootLogin no

 > enfin on enregistre le fichier et on redémarre sshd.
 
       service sshd restart

 
 > remarque : il faut ajouter ces configurations pour les PC du VLAN (30) système d'information

 
 
 ## configuration du serveur web :
 
 
 * ÉTAPE 1 : j'ai mis mon code HTML dans un répertoire spécifique pour le déplacer après vers le répertoire /etc/www/html
 
 * ÉTAPE 2 : ensuite je renome mon fichier HTML en index.html
 
 * ÉTAPE
