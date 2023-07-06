# Utilsation de Vault avec Terraform sur Debian

## Installation de Vault sur Debian:

```Bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
 echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
 sudo apt update && sudo apt install vault
```
## Vérifiez si Vault est bien installé

```Bash
vault -v
```

## Lancement du serveur Vault:

```Bash
sudo vault server -config /etc/vault.d/vault.hcl
```

 Laissez ce service tourner sur ce terminal et ouvrez-en un autre.

Actuellement l'authentification vers le serveur se base sur TLS mais pour notre introduction nous utiliserons seulement le Token fourni par notre serveur. C'est pourquoi nous désactiverons le contrôle du certificat via l'utilisation de variables d'environnement. 

## Variables d'environnement:

Pour notre utilisation nous aurons besoin de définir 3 variables contenant:
  * L'url du serveur
  * Le Token d'authentification (que nous récupérerons juste après)
  * La désactivation des vérifications du certificat

```Bash
export VAULT_SKIP_VERIFY=true
export VAULT_ADDR='https://127.0.0.1:8200'
```

## Générer les clés d'authentifications

Maintenant que le service est bien en production sur le port 8200, il est nécessaire de débloquer l'accès, car celui-ci est nativement scellé. 5 clés différentes sont générées pour être partagées entre les différents utilisateurs/administrateurs et au moins 3 de ces clés doivent être fournies à Vault pour désactiver le scellement et permettre sa manipulation.

<p>
  <img src="http://93.90.205.194/github/terraform/vault/unseal_img.png" />
</p>

Pour récupérer le Token d'authentification à Vault et les 5 clés de descellement, utilisez la commande suivante:

```Bash
vault operator init
```

<p>
 <img src="https://media.licdn.com/dms/image/D4E12AQFfnQPN-r4r6Q/article-inline_image-shrink_1500_2232/0/1687962613611?e=1694044800&v=beta&t=8YipukfaIUGLaN4n3CAL8nf0bP6R1EcqgN6u5sxoIfI">
</p>
