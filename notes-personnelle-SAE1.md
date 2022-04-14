 # Scéance 1 : configuration du routeur sur GNS3 : 

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
        
        

### Séance du 08/04/2022 (TD encadrée) :
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
        
        

  
