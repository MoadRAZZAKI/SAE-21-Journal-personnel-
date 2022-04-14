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
        exit
   
   
* Vlan 2:  


        ip dhcp pool VLAN20
        network 192.168.20.0 255.255.255.248
        exit
  
  
* Vlan 3  


        ip dhcp pool VLAN20
        network 192.168.20.0 255.255.255.248
        exit
        
        
        

