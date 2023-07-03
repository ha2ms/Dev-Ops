# Terraform: Redémarrer la machine pendant le provisionning (Remote-Exec) sans erreurs et garder la connexion SSH



Lorsque l'on souhaite faire redémarrer sa machine virtuelle via le "remote-exec" en plein milieu de son provisionnement pour que les nouveaux paramètres soit bien pris en compte sans pour autant perdre la demande de connexion SSH et fatalement générer l'erreur ci-dessous.  
<p>
  <img src="http://93.90.205.194/github/terraform/reboot/reboot_error.png" />
</p>

Le problème est qu'avant de reboot, le serveur SSH renvoi un signal d'interruption à l'exécution de la commande, ce qui est interprété comme une erreur par Terraform et par conséquent interrompt son exécution.



Il est donc nécessaire de découper son code "remote-exec" prévu initialement en 2 bloc "remote-exec" distinct, et d'ajouter entre ses 2 provisionner un 3ème ("local-exec") qui lancera une connexion SSH vers la machine cible dans le seul but de lui faire redémarrer 
Le principal objectif étant seulement "d'ignorer" ce signal d'interruption stoppant l'exécution de Terraform

<p>
  <img src="http://93.90.205.194/github/terraform/reboot/good_script.png" />
</p>



En essaynt de nouveau et au moment du lancement du reboot de la machine, le "local-exec" devrait "absorber" le signal d'interruption faisant planter Terraform puis le prochain "remote-exec" devrait réinitier la conexion vers l'hôte cible.
<p>
  <img src="http://93.90.205.194/github/terraform/reboot/reboot_success_1.png" />
</p>
<p>
  <img src="http://93.90.205.194/github/terraform/reboot/reboot_success_2.png" />
</p>
