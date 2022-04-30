 ## *Scéance 1 ( Jeudi 23/03/2022 ) , Découvrir la SAE 21 :*
 
 <br/>
 
- Dans cette première scéance de SAE 21 , notre professeur nous a expliqué un peu l'objectif de cette SAE ainsi que le type du livrable que nous devons fournir, ensuite j'ai essayé de faire un schéma de cette manipulation en utilisant l'extension draw.io sur vscode :
 <br/>
 
<img src=https://github.com/MoadRAZZAKI/SAE-21-memo-personnelle-/blob/main/SAE21.png width=2000px margin-right=-5px>
 <br/>
par la suite on s'est réparti les tâches, je me suis occupé de la partie virtuelle vu que je suis un peu à l'aise avec le logiciel de virtualisation GNS3 que l'on a utilisé pour la ressource R201, j'ai choisi GNS3 parceque ça permet d'installer tous les appareils autres que CISCO.
 
 
 <br/>
 <br/>
 
 ## *Scéance 2 ( Vendredi 08/04/2022 ) , configuration du routeur sur GNS3 :* 

- Durant cette première scéance de SAE j'ai configuré mon routeur cisco sur GNS3, j'ai commencé d'abord par configurer les adresses IP des interfaces de mon routeur : 

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
        
        
<br/> 

## *Séance 3 ( Jeudi 14/04/2022 ) (TD encadré ), configuration des VLAN sur le switch :*
---

* Pendant cette séance j'ai configuré le NAT pour chaque VLAN sur le routeur
* Configuration de la patte interne du routeur

        conf t
        interface FastEthernet 0/0.1 ##### je précise mon interface, dans ce cas c'est le VLAN10
        ip nat inside
        exit

* Configuration externe du routeur 

        conf t
        interface FastEthernet 0/1
        ip nat outside
        exit

* Configuration de l'ACL contenant une liste des adresses source interne qui seront traduite

        access-list 1 permit 192.168.10.0 0.0.0.7 ######## ici je précise la plage d'adresse pour laquelle je vais activer mon NAT

* Configuration du pool d'adresses IP globale

        ip nat pool POOL 10.213.0.0 10.213.255.254 netmask 255.255.0.0  ####### et ensuite j'ai spécifié la plage inclusive d'adresses dans le pool NAT
        
* Activation du NAT dynamique

        ip nat inside source list 1 pool POOL
        exit
        wr mem
    
<br/>    
        
## *Configuration des vlan sur le routeur :*

<br/>

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
        
        
<br/>        

## *Scéance 4 ( Vendredi 15/04/2022 ) (TD non-encadré + TP encadré ) Configuration du pool DHCP pour chaque vlan :*

<br/>

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
       
  
<br/>

## *Configuration des ACL pour plus de sécurité :* 

<br/>

* tout d'abord j'ai nommé la régle ACL et ensuite j'ai définit ce qu’elle doit filtrer. 

        R1(config)#ip access-list extended firsttothird ( du vlan 10 vers le vlan 30 )
        R1(config-ext-nacl)#deny ip 192.168.10.0 0.0.0.7 192.168.30.0 0.0.0.7
        R1(config-ext-nacl)#permit ip any any
        
* ensuite pour appliquer la règle sur une interface j'ai utilise la syntaxe suivante : 

        R1(config)#interface fastEthernet 0/0.1
        R1(config-subif)#ip access-group firsttothird in
        R1(config-subif)#ip access-group firsttothird out

<br/>

## *configuration du protocole SSH sur le routeur :* 

<br/>

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
       

 <br/>
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

 
 <br/>
 
## *Scéance 5 ( Vendredi 21/04/2022 ) (TD non-encadré + encadré ) mise à jour des règles ACL sur le routeur du réseau virtuel :*
 
 
       conf t
       ip access-list extended VLAN10INOUT
       permit udp any any eq 67  ##### cette commande permet de laisser passer tout les paquet UDP qui proviennent du port 67 
       permit udp any any eq 68   ##### cette commande permet de laisser passer tout les paquet UDP qui proviennent du port 68
       permit ip host 192.168.10.1 any      ##### pour laisser passer les paquets dont l'adresse IP est 192.168.10.1
       permit tcp 192.168.20.0 0.0.0.7 192.168.10.0 0.0.0.7 eq 22 ##### autoriser l'accès des paquets TCP provenant du port 22 et dont l'adresse du réseau est 192.168.20.0
       deny ip 192.168.30.0 0.0.0.7 192.168.10.0 0.0.0.7 ##### interdire le passage des paquets dont l'adresse source est 192.168.30.0/29 et de destination le réseau 192.168.10.0/29
       permit tcp any any eq 80 ##### autoriser l'accès à tout les paquets TCP provenant du port 80 ( http )
       permit tcp any any eq 443 ##### autoriser l'accès à tout les paquets TCP provenant du port 443 ( https )
       permit icmp any any ##### autoriser l'accès à tout les paquets ICMP
       permit ip any any ##### laisser passer le reste des paquets IP ( j'ai remarqué que lorsque j'enlevais cette condition j'arrivais pas à avoir accès à Internet donc je l'ai ajouté ) 
       permit tcp 192.168.10.0 0.0.0.7 192.168.20.0 0.0.0.7  established ######## autoriser l'accès aux paquets TCP d'adresse source 192.168.10.0/29 et de destination le réseau 192.168.20.0/29 
       permit tcp any any eq 22  ##### autoriser l'accès à tout les paquets TCP de port source 22
       exit
 
 
       ip access-list extended VLAN20INOUT
       permit udp any any eq 67
       permit udp any any eq 68
       permit ip host 192.168.20.1 any
       permit tcp 192.168.10.0 0.0.0.7 192.168.20.0 0.0.0.7  eq 22 established
       permit tcp 192.168.30.0 0.0.0.7 192.168.20.0 0.0.0.7  eq 22 established
       permit tcp 192.168.10.0 0.0.0.7 any established
       permit tcp 192.168.30.0 0.0.0.7 any established
       permit tcp any any eq 80
       permit tcp any any eq 443
       permit icmp any any
       permit ip any any
       permit tcp any any eq 22
       exit

 
       ip access-list extended VLAN30INOUT
       permit udp any any eq 67
       permit udp any any eq 68
       permit ip host 192.168.30.1 any
       permit tcp 192.168.20.0 0.0.0.7 192.168.30.0 0.0.0.7 eq 22
       deny ip 192.168.10.0 0.0.0.7 192.168.30.0 0.0.0.7
       permit tcp any any eq 80
       permit tcp any any eq 443
       permit icmp any any 
       permit ip any any
       permit tcp 192.168.30.0 0.0.0.7 192.168.20.0 0.0.0.7  established
       permit tcp any any eq 22
       exit
 
 
       ip access-list extended VLAN40INOUT
       permit udp any any eq 67
       permit udp any any eq 68
       permit ip host 192.168.40.1 any
       permit tcp 192.168.10.0 0.0.0.7 any eq 80
       permit tcp 192.168.20.0 0.0.0.7 any eq 80
       permit tcp 192.168.30.0 0.0.0.7 any eq 80
       permit icmp any any
       permit ip any any
       permit tcp any any established
       exit


       interface FastEThernet 0/0.1
       ip access-group VLAN10INOUT in ###### activation de l'accèss liste VLAN10INOUT sur l'entrée de la sous interface 0/0.1
       ip access-group VLAN10INOUT out  ###### activation de l'accèss liste VLAN10INOUT sur la sortie de la sous interface 0/0.1
       interface FastEThernet 0/0.2
       ip access-group VLAN20INOUT in
       ip access-group VLAN20INOUT out
       interface FastEThernet 0/0.3
       ip access-group VLAN30INOUT in
       ip access-group VLAN30INOUT out
       interface FastEThernet 0/0.4
       ip access-group VLAN40INOUT in
       ip access-group VLAN40INOUT out
       end
       wr mem

 
 
 <br/>
 
 ## *Configuration du serveur web :*
 
 <br/>
 
 * ÉTAPE 1 : j'ai mis mon code HTML dans un répertoire spécifique en utilisant la commande mkdir pour créer un nouveau répertoire pour le déplacer après vers le répertoire /etc/www/html :
 
 
 
       mkdir /repertoire 
      
       nano cyberbridge.html ########## pour écrire le fichier html dans le répertoire 
 
       mv /repertoire /var/www/html
 
 
 
 * ÉTAPE 2 : ensuite j'ai renomme mon fichier HTML en index.html
 
       cd /var/www/html/repertoire 
     
       mv cyberbridge.html index.html 

 
 * ÉTAPE 3 : on recopie le fichier de configuration /etc/apache2/sites-available/000-default.conf et on le renomme en cyberbridge.conf en utilisant la commande cp :
 
 
       cp 000-default.conf cyberbridge.conf
 
 
 * ÉTAPE 4 : ensuite, on modifie notre site en utilisant la commande, nano ,on change le server name par le nom de serveur que l’on veut et on change le répertoire de notre site en modifiant la ligne DocumentRoot :

 
       <VirtualHost *:80>
        
        ServerName www.cyberbridge.fr          
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

       </VirtualHost>

 * ÉTAPE 5 : on active ensuite notre site en utilisant la commande a2ensite :
 
        a2ensite cyberbridge.conf
     
        systemctl reload apache2 ##### on utilise cette commande pour permettre la relecture des fichiers de configuration.
 
 
 * ÉTAPE 6 : ensuite, on modifie notre répertoire /etc/hosts , et on met à la place de localhost notre nom de serveur que l’on a choisis tout à l’heure : 
 
 
        #127.0.0.1       localhost
        127.0.0.1        www.cyberbridge.fr
        10.213.22.1     232-22

        # The following lines are desirable for IPv6 capable hosts
        ::1     localhost ip6-localhost ip6-loopback
        ff02::1 ip6-allnodes
        ff02::2 ip6-allrouters
 
 * ÉTAPE 7 : on reload le service apache2 en utilisant la commande systemctl reload :
 
 
        systemctl reload apache2
 <br/>
 
 ## *Scéance 6 ( Lundi 25/04/2022 ) configuration du Firewall du routeur :*
 
 <br/>
* Dans cette scéance , je me suis concentré sur la configuration du firewall de mon routeur mikrotik qui est relié au réseau de la salle, donc j'ai commencé par déterminer les trames qu'il faut laisser traverser notre routeur externe ainsi que les ports de ces paquets , ensuite j'ai regardé une vidéo sur les actions et les chain dans les filtres :
 
 <br/>
 <br/>
 
* Comme on peut le remarquer, lorsque l'on tape la commande export sur le terminal du mikrotik,ça nous affiche toute les configuration du routeur , et si on regarde la partie IP Firewall Filter , on peut remarquer que j'ai laissé passer les paquets TCP port source : 80,443 ( pour les trames http/https ) , UDP port : 53 ( pour les trames DNS ) , UDP port source : 67 ( source des réponses du serveur DHCP ) , UDP port destination : 68 ( port du client DHCP ) , et j'ai laissé passer tous les paquets ICMP .

 <br/>
 
 * Ensuite comme vous pouvez le remarquer , pour tout les paquets que j'ai laissé traverser mon routeur j'ai choisi comme chain forward : car quand j'ai fait mes recherches j'ai compris qu'il existe trois types de chaine sur le firwall du routeur mikrotik :

 <br/>
 
 - INPUT (paquets à destination du firewall embarqué),
 - OUTPUT (paquets émis par le firewall),
 - FORWARD (paquets qui traversent le firewall d'une interface vers une autre).
 
  <br/>
 
 > ensuite, en ce qui concerne l'état de la connexion , j'ai accepté que les paquets avec un état de connexion nouveau et établis.
 
 <img src=https://github.com/MoadRAZZAKI/SAE-21-memo-personnelle-/blob/main/router_config_moad_razzaki.png width=2000px margin-right=-5px>
 
 
 
<br/>
 
<br/>
 
 ## *Configuration du serveur DNS :*




## Configuration du fichier db.cyberbridge.fr ( notre fichier de zone ) :
 
 
> tout d'abord on se déplace vers le répertoire /etc/bind : 

> ensuite en utilisant la commande MV on renomme notre fichier db.local en db.cyberbridge.fr :
 
 
        mv db.local db.cyberbridge.fr 
 
 
 > après avoir changé le nom de notre fichier , on le modifie en utilisant la commande nano : 
 
 


        ----------------------------------------------------------------------
        ;
        ; BIND data file for local loopback interface
        ;
        $TTL    604800
        @       IN      SOA     213-13.cyberbridge.fr. root.cyberbridge.fr. (
                                2         ; Serial
                           604800         ; Refresh
                            86400         ; Retry
                          2419200         ; Expire
                           604800 )       ; Negative Cache TTL
        ;
        @       IN      NS      213-13  // le hostname de notre machine 
        213-13  IN      A       192.168.88.252 // l'adresse ip de notre serveur WEB 
        www     IN      A       192.168.88.252
        -----------------------------------------------------------------------

## Configuration du fichier /etc/bind/named.conf.local :




        ---------------------------------------------------------
        //
        // Do any local configuration here
        //

        // Consider adding the 1918 zones here, if they are not used in your
        // organization
        //include "/etc/bind/zones.rfc1918";

          zone "cyberbridge.fr" {                  // le nom de zone de recherche directe
                  type master;                            // le type de la zone de recherche directe
                  file "/etc/bind/db.cyberbridge.fr";     // le fichier de configuration de la zone directe
          };

          zone "13.213.10.in-addr.arpa." {      // l'adresse de la zone de recherce inversée
                  type master;                          // le type de la zone de recherche inversée
                  file "/etc/bind/inverse";    // le fichier de configuration de la zone de recherche inversée
          };

        -------------------------------------------------------  


## Configuration du fichier /etc/bind/named.conf.options :




         --------------------------------------------------------
         options {
         directory "/var/cache/bind";

         // If there is a firewall between you and nameservers you want
         // to talk to, you may need to fix the firewall to allow multiple
         // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

         // If your ISP provided one or more IP addresses for stable
         // nameservers, you probably want to use them as forwarders.  
         // Uncomment the following block, and insert the addresses replacing
         // the all-0's placeholder.

         forwarders { 8.8.8.8; }; // On y indique les adresses des serveurs DNS externes ( GOOGLE ) dans la section forwarders

         //========================================================================
         // If BIND logs error messages about the root key being expired,
         // you will need to update your keys.  See https://www.isc.org/bind-keys
         //========================================================================
         dnssec-validation auto;

         listen-on-v6 { any; };
         };
         ----------------------------------------------------------



> après avoir finir toute les configurations, on tape la commande systemctl restart bind9 pour redémarrer notre service DNS



 
 
 
<center><p>2022 &copy; Moad RAZZAKI -<span style="color:#3DFF05"> Tous droits réservés</span></p></center>
