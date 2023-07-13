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

Ici vous retrouverez les 5 clés "Unseal" (descellement), ainsi que le Root Token, pensez bien à récupérer ces valeurs et à les stocker dès maintenant. Nous allons également en profiter pour ajouter le Root Token à notre variable d'environnement:

```Bash
export VAULT_TOKEN="hvs.13Xal16..."
```
Ou en l'ajoutant dans le fichier [ .bashrc ]


Si vous souhaitez que les modifications apportées à votre fichier [ .bashrc ] soit appliquées sur votre terminal actuel, effectué la commande [ source /home/'user'/.bashrc ].


Nous pouvons sans plus tarder desceller avec la commande : [ vault operator unseal ], vous devrez renouveler cette commande au moins 3 fois pour en utilisant à chaque fois une clé "Unseal" différente. Vous saurez que Vault est descellé lorsque le paramètre "Sealed" renvoyé par la commande sera à "false" comme surligné en jaune dans l'image ci-dessous.

<p>
 <img src="https://media.licdn.com/dms/image/D4E12AQFTjmHLVP6p-g/article-inline_image-shrink_1500_2232/0/1687964035068?e=1694044800&v=beta&t=JPitwE8UB3C6Eo0DKADs9T2HfXDpUmoR4QNUVvCR3YY" />
</p>
 Vault se scellera de nouveau dès lors que le service sera coupé puis redémarré, il faudra donc recommencer cette opération.

Vault est enfin opérationnel ! 
<p>
 <img src="https://media.licdn.com/dms/image/D4E12AQG7tasw5uvSOg/article-inline_image-shrink_1500_2232/0/1688070733890?e=1694044800&v=beta&t=sUahibcXqhCU3jLtWirz_WBcb2Rp-592Cd2AR6_YD9s" />
</p>

## Création d'un nouveau secret et d'un ensemble de clés

Pour imager le stockage des données sensibles par Vault, on peut penser à une organisation en répertoire (appelé path), où sont conservés les fichiers (appelés secrets), contenant chacun un ensemble de clé=valeur (appelé keys).

Pour créer ce fameux path (répertoire), utilisez la commande suivante:
```Bash
vault secrets enable -path=esxi-lille -version=1 kv
```
Nous pouvons dès à présent ajouter un secret avec les keys qu'il contient dans le path (répertoire) que nous venons de créer:
```Bash
vault kv put esxi-lille/auth user='Mehdi' password='MyP\@ssw0rd'
```
 L'ajout de caractères spéciaux (comme le @) nécessite d'être précédés d'un '\'.

L'on peut maintenant vérifier le contenu de nos données chiffrées via la commande: 
```Bash
vault kv get esxi-lille/auth
```

<p>
 <img src="https://media.licdn.com/dms/image/D4E12AQFZ8Xwfl0El_w/article-inline_image-shrink_1500_2232/0/1688071815916?e=1694044800&v=beta&t=sk4w2hbWkmUMkopronM74Zgglk6IoNIppoKapkw1yJE" />
</p>

Nous pouvons également lister les secrets créés de notre path. Ici nous avons seulement le secret [ auth ] de défini:

```Bash
vault kv list esxi-lille
```
<p>
 <img src="">
</p>

Nos données sont maintenant stockées et prêtes à l'emploi !

## Importation des données sensibles depuis Terraform:
