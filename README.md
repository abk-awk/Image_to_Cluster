------------------------------------------------------------------------------------------------------
ATELIER FROM IMAGE TO CLUSTER
------------------------------------------------------------------------------------------------------
L’idée en 30 secondes : Cet atelier consiste à **industrialiser le cycle de vie d’une application** simple en construisant une **image applicative Nginx** personnalisée avec **Packer**, puis en déployant automatiquement cette application sur un **cluster Kubernetes** léger (K3d) à l’aide d’**Ansible**, le tout dans un environnement reproductible via **GitHub Codespaces**.
L’objectif est de comprendre comment des outils d’Infrastructure as Code permettent de passer d’un artefact applicatif maîtrisé à un déploiement cohérent et automatisé sur une plateforme d’exécution.
  
-------------------------------------------------------------------------------------------------------
Séquence 1 : Codespace de Github
-------------------------------------------------------------------------------------------------------
Objectif : Création d'un Codespace Github  
Difficulté : Très facile (~5 minutes)
-------------------------------------------------------------------------------------------------------
**Faites un Fork de ce projet**. Si besion, voici une vidéo d'accompagnement pour vous aider dans les "Forks" : [Forker ce projet](https://youtu.be/p33-7XQ29zQ) 
  
Ensuite depuis l'onglet [CODE] de votre nouveau Repository, **ouvrez un Codespace Github**.
  
---------------------------------------------------
Séquence 2 : Création du cluster Kubernetes K3d
---------------------------------------------------
Objectif : Créer votre cluster Kubernetes K3d  
Difficulté : Simple (~5 minutes)
---------------------------------------------------
Vous allez dans cette séquence mettre en place un cluster Kubernetes K3d contenant un master et 2 workers.  
Dans le terminal du Codespace copier/coller les codes ci-dessous etape par étape :  

**Création du cluster K3d**  
```
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```
```
k3d cluster create lab \
  --servers 1 \
  --agents 2
```
**vérification du cluster**  
```
kubectl get nodes
```
**Déploiement d'une application (Docker Mario)**  
```
kubectl create deployment mario --image=sevenajay/mario
kubectl expose deployment mario --type=NodePort --port=80
kubectl get svc
```
**Forward du port 80**  
```
kubectl port-forward svc/mario 8080:80 >/tmp/mario.log 2>&1 &
```
**Réccupération de l'URL de l'application Mario** 
Votre application Mario est déployée sur le cluster K3d. Pour obtenir votre URL cliquez sur l'onglet **[PORTS]** dans votre Codespace et rendez public votre port **8080** (Visibilité du port).
Ouvrez l'URL dans votre navigateur et jouer !

---------------------------------------------------
Séquence 3 : Exercice
---------------------------------------------------
Objectif : Customisez un image Docker avec Packer et déploiement sur K3d via Ansible
Difficulté : Moyen/Difficile (~2h)
---------------------------------------------------  
Votre mission (si vous l'acceptez) : Créez une **image applicative customisée à l'aide de Packer** (Image de base Nginx embarquant le fichier index.html présent à la racine de ce Repository), puis déployer cette image customisée sur votre **cluster K3d** via **Ansible**, le tout toujours dans **GitHub Codespace**.  

**Architecture cible :** Ci-dessous, l'architecture cible souhaitée.   
  
![Screenshot Actions](Architecture_cible.png)   
  
---------------------------------------------------  
## Processus de travail (résumé)

1. Installation du cluster Kubernetes K3d ( A réaliser en Séquence 1)
2. Installation de Packer et Ansible

Installer packer 
```
wget https://releases.hashicorp.com/packer/1.10.0/packer_1.10.0_linux_amd64.zip
unzip packer_1.10.0_linux_amd64.zip
sudo mv packer /usr/local/bin/
```

Installer ansible 
```
pip3 install ansible
```
Vérifier le fonctionnement de ansible && packer 
```
ansible --version
packer --version
```
4. Build de l'image customisée (Nginx + index.html)
Se placer dans le dossier mon_projet/packer pour build l'iange customisée en exécutant :

```
packer init .
packer validate validate nginx-image.pkr.hcl
packer build nginx-image.pkr.hcl

```
Vérifier que le service est bien lancé : 


```
kubectl get pods
kubectl get svc | grep my-nginx    # pour voir si le service est bien lancé, il est doit etre en **running**
```
Vous devriez voir my-ngnix comme service
6. Import de l'image dans K3d

Entrer la commande suiavnte pour importer l'**image nginx** précedemment créée vers le cluster k3d :

```
k3d cluster create lab --servers 3 --agents 3 -p "8080:80@loadbalancer"

```
7. Déploiement du service dans K3d via Ansible

Entrer la commande suivante pour deployer le service dans k3d :


```
k3d image import my-nginx:latest -c lab

# Vérifier que l'image est bien présenté dans la liste affichée
k3d image list -c lab | grep my-nginx
```
9. Automatisation du déploiemen avec ansible

Se placer dans le dossier /mon_projet_ansible avec la commande : 

```
cd ansible
```

```
# Tester la connexion
ansible localhost -i inventory.ini -m ping

# Lancer le déploiement
ansible-playbook -i inventory.ini playbook.yml
```
10. Ouverture des ports et vérification du fonctionnement

vérifier que l'application tourne correctment en l'ayant en status running 

```
kubectl get svc | grepp my-nginx
```

Ensuite réalisez le port-forward du 80 pour permettre l'accès de l'application par un numéro de port au choix 

```
kubectl port-forwrad svc/my-nginx 8081:80 & # port 8081 dans mon cas
```
11. Tout en un 

Déployer votre application en exécutant une seule commande : 

Placez vous dans /mon_projet : 

```
cd mon_projet
```

Entrer la commande suivante : 

```
make deploy 
```
---------------------------------------------------
---------------------------------------------------
Séquence 4 : Documentation  
Difficulté : Facile (~30 minutes)
---------------------------------------------------
**Complétez et documentez ce fichier README.md** pour nous expliquer comment utiliser votre solution.  
Faites preuve de pédagogie et soyez clair dans vos expliquations et processus de travail.  
   
---------------------------------------------------
Evaluation
---------------------------------------------------
Cet atelier, **noté sur 20 points**, est évalué sur la base du barème suivant :  
- Repository exécutable sans erreur majeure (4 points)
- Fonctionnement conforme au scénario annoncé (4 points)
- Degré d'automatisation du projet (utilisation de Makefile ? script ? ...) (4 points)
- Qualité du Readme (lisibilité, erreur, ...) (4 points)
- Processus travail (quantité de commits, cohérence globale, interventions externes, ...) (4 points) 


