Dans cet exercice, il ne s'agit pas d'une véritable entreprise ni de vraies personnes. Tout ce qui est présenté ici a pour objectif d'être utilisé à des fins éducatives, afin de s'entraîner.


Pour se connecter à distance, il faut d'abord exécuter les commandes suivantes :
sudo openvpn --config akil.ovpn

sudo route -n add -net 192.168.40.0/24 -interface utun3


Ensuite, voici à quoi ressemble mon fichier /etc/hosts :

![Capture d’écran 2024-11-14 à 16 00 35](https://github.com/user-attachments/assets/09392315-4757-494f-98ab-cc74ef0e38c4)

Vous pouvez regarde la suite sur le pdf dans le dossier RED_TEAM_FINAL.
