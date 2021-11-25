# Domain Driven Design


### Data Modelling Dept :
 Prenons l'exemple de deux systèmes chargés de gérer un état donné d'un rotor d'hélicoptère.
- Un système legacy (:= SystèmeA) est chargé de gérer la phase de conception.
- Vous devez mettre en œuvre un nouveau système (:= SystèmeZ) chargé de gérer la phase de maintenance.

L'ensemble applicatif informatique partage le même identifiant commun (:= partId) sauf SystemA qui n'en a pas connaissance. Au lieu de partId, SystemA gère son propre identifiant (:= systemAId). Puisque SystemZ doit appeler 
SystemA en utilisant systemAId, une heuristique pourrait être d'intégrer systemAId comme partie du modèle de données de SystemZ.

> Vous avez simplement corrompu votre modèle de données en raison d'une situation à court terme.

Solution : **Anti-Corruption Layer**

SystemZ aurait pu mettre en œuvre son propre format de données (sans corruption externe). Il aurait alors appartenu à une couche d'intermédiation de gérer la traduction entre le partId et le systemAId.

> [source](https://teivah.medium.com/why-is-a-canonical-data-model-an-anti-pattern-441b5c4cbff8)



