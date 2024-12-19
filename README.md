# README - Connexion au Réseau VPN pour Exercice Éducatif

Ce guide fournit les étapes nécessaires pour se connecter à un réseau VPN dans le cadre d'un exercice éducatif. Ce projet est fictif et a pour seul objectif de permettre un entraînement à des fins pédagogiques.

---

## **1. Configurer la Connexion VPN**

Pour établir la connexion au réseau VPN :

1. Ouvrez un terminal sur votre machine.
2. Exécutez la commande suivante pour lancer le VPN avec le fichier de configuration fourni :
   ```bash
   sudo openvpn --config akil.ovpn
   ```

3. Ajoutez une route réseau avec la commande suivante :
   ```bash
   sudo route -n add -net 192.168.40.0/24 -interface utun3
   ```

Ces étapes établissent la connexion avec le réseau VPN et permettent d'accéder à l'environnement distant.

---

## **2. Fichier de Configuration /etc/hosts**

Assurez-vous que votre fichier `/etc/hosts` contient les informations nécessaires pour résoudre les noms de domaine associés à l'exercice. Voici un exemple de configuration :

![Fichier /etc/hosts](https://github.com/user-attachments/assets/09392315-4757-494f-98ab-cc74ef0e38c4)

Pour modifier ce fichier :
1. Ouvrez-le avec un éditeur de texte (exemple : nano) en mode super-utilisateur :
   ```bash
   sudo nano /etc/hosts
   ```
2. Ajoutez les entrées nécessaires comme dans l'exemple ci-dessus.
3. Sauvegardez et quittez l'éditeur.

---

## **3. Documentation Supplémentaire**

Pour plus de détails sur l'exercice :
- Consultez le fichier PDF **ReportTemplate_AKIL** pour un rapport détaillé.
- Suivez les instructions contenues dans le fichier **LettreMission** pour les étapes spécifiques de la mission.

Ces documents fournissent les informations nécessaires pour compléter l'exercice.

---

## **4. Avertissement**

- **Ce projet est fictif :** Les entreprises, personnes et données mentionnées sont inventées.
- **Utilisation éducative uniquement :** Les outils et techniques décrits ne doivent être utilisés qu'à des fins d'apprentissage dans un environnement contrôlé.

Si vous avez des questions ou besoin d’assistance, veuillez vous référer aux documents supplémentaires fournis ou contactez votre instructeur.

