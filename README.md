# script permettant de récupérer les données des dirigeants des entreprises (données opendata provenant de l'INPI)

Données récupérées sur data.cquest.org (qui proviennent du FTP de l'INPI).

Les données sont réparties entre différents greffes. Pour gagner plus de performance, nous avons fait un pré-traitement permettant de regrouper en un seul fichier csv par jour, les différents fichiers relatifs aux représentants.

Le process (sur des notebooks jupyter - inpi-processing.ipynb) :

- récupération du stock initial du 4/5/2017
  - ingestion des données dans une base sqlite. Deux tables : rep_pp (personnes physiques) et rep_pm (personnes physiques)
- pour chaque jour depuis le 5/5/2017 :
  - vérifier si un fichier stock existe, si oui:
    - pour tous les siren présent dans les fichiers, supprimer dans la base sqlite les données relatives à ces siren
    - ajouter dans la base sqlite l'ensemble des représentants (en dispatchant selon PP ou PM dans la bonne table)
  - vérifier si un fichier de flux rep existe, si oui:
    - ajouter dans la base sqlite l'ensemble des représentants (en dispatchant selon PP ou PM dans la bonne table)
  - vérifier si un fichier de flux rep modifie ou nouveau, si oui:
    - ajouter dans la base sqlite l'ensemble des représentants (en dispatchant selon PP ou PM dans la bonne table)
  - vérifier si un fichier de flux rep partant, si oui:
    - supprimer les dirigeants un à un en parcourant le fichier

Il y a beaucoup de duplicats (notamment car beaucoup d'info entre les fichiers flux rep et flux rep nouveau modifie sont les mêmes). A ce stade, pas plus d'investigation réalisée. Cela peut se résoudre via de la déduplication a posteriori de l'insertion en base.
Nous avons opté pour cette option car plus performante en temps (elle évite de checker ligne à ligne si un dirigeant est déjà présent). Ce n'est pas parfait néanmoins. A voir si cela peut être modifié.

pour générer les fichiers csv PP et PM :
- export des tables sqlite vers un dataframe pandas
- dédoublements de données rigoureusement les mêmes
- groupement par siren, noms, prenoms pour les PP et siren, denomination pour les PM pour qu'un dirigeant ayant plusieurs qualite n'ait qu'une ligne (ex : patrick dupont, Administrateur ET Président ET Commissaire aux comptes)
- enregistrement csv

Les données sont actuellement stockées sur https://files.data.gouv.fr/annuaire-entreprises/inpi-dirigeants/ et auront vocation à être intégrer dans le moteur de recherche de l'annuaire des entreprises.

TODO :
- Process de mise à jour
- Branchement au FTP INPI et plus data.cquest
- intégration des données dans le workflow annuaire-entreprise pour ingestion des données dans notre base elasticsearch


## Les liens (vraiment utiles)

- [Données](https://data.cquest.org/inpi_rncs/imr/)
- [Specs INPI](https://www.inpi.fr/sites/default/files/doc_tech_imr_novembre_2020_v1.5.2.pdf)
- [Article Christian Quest](https://cq94.medium.com/le-rncs-en-quasi-opendata-57446c6fc8dd)
- [Discussion Forum](https://teamopendata.org/t/quasi-opendata-ou-opendatafail-pour-le-rncs-diffuse-par-linpi/1603/9)
- [Repo API entreprise, Lire le readme !!](https://github.com/etalab/rncs_worker_api_entreprise)
