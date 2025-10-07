---
layout: home
---

<div id="toc">

* TOC
{:toc}

</div>

<div id="content">

# Introduction

L'objectif de ce document est de décrire comment utiliser les services offerts par le système Isograd, soit pour les consommateurs LTI, soit via l'API.

---

# Démarrage rapide

## LTI

### Connexion

La plateforme Isograd est compatible avec les protocoles LTI 1.1 et LTI 1.3.

#### LTI 1.1

Votre système doit inclure une zone pour définir les fournisseurs LTI. Dans cette zone, vous devez :
- Donner un nom au fournisseur LTI Isograd
- Spécifier votre clé client
- Spécifier votre secret client

Votre clé client et votre secret client vous seront fournis par Isograd sur demande.

#### LTI 1.3

**Ce que nous demandons de votre part**

![](/assets/img/platform.png)

**Ce que nous vous fournirons**

![](/assets/img/tool.png)

### Environnements

Il existe deux environnements, l'un pour les tests et l'autre pour la production :
- Test : `https://recette.isograd.com/public/lti.php`
- Production : `https://app.isograd.com/public/lti.php`

### Utiliser un service

La plupart des appels aux services LTI Isograd nécessitent des **paramètres personnalisés** pour indiquer ce que vous souhaitez réaliser. Votre système doit inclure la possibilité d'ajouter des paramètres personnalisés dans vos messages LTI.

![](/assets/img/lti_additional_param.jpg)

### Exemple

Pour tester manuellement vos requêtes, vous pouvez utiliser l'outil d'émulation LTI accessible ici : [https://saltire.lti.app/platform](https://saltire.lti.app/platform). Voici un exemple d'un tel test :

![](/assets/img/lti_message.jpg)
![](/assets/img/lti_user.jpg)

---

## API

### Connexion

L'authentification se fait via [OAuth](https://oauth.net/getting-started/). Pour obtenir un token, vous devez effectuer une requête POST à l'URL suivante :

```
https://auth.isograd.com/oauth2/token?grant_type=client_credentials&scopes=usage
```

Cet appel doit contenir deux champs : `client_id` et `client_secret`. Le contenu de ces champs doit être les deux valeurs que l'équipe Isograd vous a fournies lors de la création de votre compte.

### Environnements

Il existe deux environnements, l'un pour les tests et l'autre pour la production :
- Test : `https://recette.isograd.com/api/usage`
- Production : `https://app.isograd.com/api/usage`

### Utiliser un service

Pour utiliser un service, effectuez une requête POST à l'URL de l'environnement choisi. L'en-tête doit contenir le champ `Authorization: Bearer <votre_token>`.

### Exemple

Voici un script Python récupérant un token d'accès et utilisant le service pour [créer un candidat](#créer-un-candidat) :

```python
import requests

# Récupérer le token d'accès
credentials = {
    'client_id': 's0u2c72bman19nm2c45oi1fpea',
    'client_secret': '1mio0273po6si9tvt2f3ld5t6o4hsg02epq71rlfdhlkombkv8ro',
}
auth_url = 'https://auth.isograd.com/oauth2/token?grant_type=client_credentials&scopes=usage'

r_token = requests.post(auth_url, data=credentials)
access_token = r_token.json()['access_token']

# Utiliser le service
headers = {'Authorization': 'Bearer ' + access_token}
url_dev = 'https://recette.isograd.com/api/usage'

payload = {
    'act_id': 1,
    'gen_id': 3,
    'ema': 'test@isograd.com',
    'fst_nam': 'John',
    'lst_nam': 'Doe',
}

r_action = requests.post(url_dev, data=payload, headers=headers)
print(r_action.json())
```

L'objet JSON retourné ressemble à ceci :

```json
{
    "error_message": null,
    "success": true,
    "can_id": 1381134,
    "lgn_url": "https://recette.isograd.com/continuelogin/CandidateContinueLogin?param=UVR1WjgwRXBwU0"
}
```

---

# Actions

Toutes vos actions doivent contenir un paramètre `act_id` qui indique l'action que vous souhaitez effectuer. Ces actions et les paramètres à envoyer sont décrits dans cette section. Pour chaque paramètre, il y aura une indication de s'il est requis ou non :

- 🟩 Paramètre requis
- 🔷 **API** : Paramètre requis — **LTI** : paramètre déjà envoyé via le protocole LTI, ne pas l'inclure dans les paramètres supplémentaires
- 🟠 Paramètre optionnel

La réponse sera un objet JSON contenant un booléen `success` : si `true`, des propriétés supplémentaires sont envoyées selon l'action. Si `false`, un `error_code` et un `error_message` seront inclus. Voici les messages d'erreur généraux (des codes supplémentaires spécifiques à chaque action sont fournis plus loin dans ce document) :

| error_code | error_message |
|------------|---------------|
| 103 | La propriété act_id n'est pas valide |
| 105 | La propriété "{nom de propriété}" est requise |

---

## Candidats et tests

### Créer un candidat

| Paramètre | Requis | Valeur |
|-----------|--------|--------|
| act_id | 🟩 | 1 |
| gen_id | 🟩 | 1: homme, 2: femme, 3: non renseigné |
| ema | 🔷 | Adresse email du candidat |
| fst_nam | 🔷 | Prénom du candidat |
| lst_nam | 🔷 | Nom du candidat |
| lan_id | 🟠 | Code de langue. Voir l'[annexe](#codes-de-langue) |
| ext_id | 🟠 | Votre identifiant interne pour le candidat. En LTI, le `UserID` LTI est stocké si aucun `ext_id` n'est fourni |
| dep_id | 🟠 | L'identifiant d'un groupe dans le système Isograd auquel le candidat doit être ajouté. Doit être vide si une valeur `ext_dep_id` est fournie |
| ext_dep_id | 🟠 | L'identifiant d'un groupe dans votre système auquel le candidat doit être ajouté. Si aucun groupe avec cet identifiant n'existe, il sera automatiquement créé |
| psw | 🟠 | Le mot de passe du candidat. S'il n'est pas fourni, le système en générera un aléatoirement |
| no_psw_rst | 🟠 | Si défini sur 1, le mot de passe sera haché, il ne sera pas affiché dans les emails d'inscription et le candidat n'aura pas à le changer lors de la première connexion |
| job_id | 🟠 | L'identifiant du profil métier du candidat |
| xtr_tim_fac | 🟠 | Facteur de pourcentage définissant le temps supplémentaire accordé à un candidat (par exemple, 0 pour aucun temps supplémentaire, 50 pour +50% de temps, 100 pour +100% de temps, 200 pour +200% de temps), appliqué à la fois à la durée totale du test et au temps de question dans l'application |

**Réponse :**

La réponse est un objet JSON contenant les propriétés suivantes :
- `can_id` : identifiant entier unique du candidat créé sur la plateforme Isograd
- `lgn_url` : une URL qui peut être utilisée par le candidat pour se connecter à la plateforme sans saisir d'identifiants. Cette URL n'est valable que 30 minutes

**Erreurs :**

| error_code | error_message |
|------------|---------------|
| 201 | Le candidat n'a pas pu être créé (probablement parce que l'un des paramètres n'est pas valide) |
| 202 | Il existe déjà un candidat avec cet email sur votre compte. Dans ce cas, la réponse inclut une propriété `can_id` avec l'identifiant du candidat correspondant |
| 203 | Impossible de trouver le `dep_id` mentionné dans le compte et impossible de créer le candidat |

---

### Ajouter un test à un candidat

| Paramètre | Requis | Valeur |
|-----------|--------|--------|
| act_id | 🟩 | 2: ne pas ajouter le test si le candidat a un test non terminé pour ce `tst_frm_id`<br>9: Créer le test même si le candidat a un test non terminé |
| tst_frm_id | 🟩 | L'identifiant du test. Pour obtenir la liste, connectez-vous à la plateforme Isograd et cliquez sur "Aide" dans le menu de gauche. Pour des raisons de compatibilité, ce paramètre peut être nommé `rea_tst_id` jusqu'au 31/12/2025 |
| ema | 🔷 | Adresse email du candidat |
| nee_ful_scr | 🟠 | 1: Exiger le plein écran pour le test |
| html | 🟠 | 1: La réponse sera une balise HTML `<a>` (avec un attribut class `isograd_start_test_button`) |
| redirect | 🟠 | 1: La réponse sera un statut HTTP 302 redirigeant vers l'URL de la page de début du test |
| ses_id | 🟠 | L'identifiant de session auquel le test doit être associé |
| add_pro | 🟠 | 1: ajouter la surveillance à distance au test. Un coût supplémentaire s'appliquera |
| max_num_tst | 🟠 | Le nombre maximum de tests avec ce `tst_frm_id` que le candidat est autorisé à passer |
| max_num_tst_per | 🟠 | Le nombre maximum de tests avec ce `tst_frm_id` que le candidat est autorisé à passer tous les X jour(s) |
| rtn_pag | 🟠 | L'URL de la page vers laquelle les candidats seront redirigés après avoir soumis leurs commentaires (ou leurs résultats s'ils sont autorisés à les voir) |
| cal_bac_pag | 🟠 | Une URL à laquelle la plateforme soumettra une requête GET lorsque le test est terminé (avant d'afficher la page de commentaires/résultats) |
| cpf_id | 🟠 | L'identifiant du dossier "Compte Personnel de Formation" associé à ce test |
| nb_test_by_credit | 🟠 | Utiliser un pack multi-crédits (2, 3, 4) |

**Réponse :**

La réponse sera un objet JSON (sauf si `html` ou `redirect` sont définis sur 1) contenant les propriétés suivantes :
- `tst_url` : l'URL qui lancera automatiquement le test spécifié
- `pla_tst_id` : identifiant entier de ce test pour ce candidat dans la plateforme Isograd

Note : si le candidat a un test non terminé et que `act_id` est défini sur `2`, la réponse contiendra les détails du test non terminé.

**Erreurs :**

| error_code | error_message |
|------------|---------------|
| 106 | Ce `tst_frm_id` n'est pas autorisé |
| 107 | Ce candidat n'existe pas |
| 301 | Vous n'avez plus de crédits pour ce type de test |
| 402 | Le candidat a déjà passé `max_num_tst` pour ce `tst_frm_id` |

> 💡 **Note** : La plupart des systèmes se connectant à la plateforme Isograd définiront le paramètre optionnel `redirect` sur true car cela permet d'avoir le comportement standard attendu : le candidat clique sur un lien dans le LMS et le test démarre automatiquement.

---

### Créer un candidat et passer un test

| Paramètre | Requis | Valeur |
|-----------|--------|--------|
| act_id | 🟩 | 8 |

Le système effectuera successivement les actions Créer un candidat et Ajouter un test décrites ci-dessus. Par conséquent, la structure de retour sera similaire à l'action Ajouter un test et tous les paramètres requis ou optionnels dans les actions Ajouter un test et Créer un candidat peuvent être fournis.

---

### Ajouter/retirer la surveillance en ligne d'un test

| Paramètre | Requis | Valeur |
|-----------|--------|--------|
| act_id | 🟩 | 16: ajouter la surveillance en ligne<br>17: retirer la surveillance en ligne |
| pla_tst_id | 🟩 | Identifiant du test qui a été retourné lors de la création du test |

La réponse sera un objet JSON ne contenant aucune propriété spécifique.

**Erreurs :**

| error_code | error_message |
|------------|---------------|
| 303 | Vous n'avez plus de crédit de surveillance |

---

### Supprimer un test

| Paramètre | Requis | Valeur |
|-----------|--------|--------|
| act_id | 🟩 | 23 |
| pla_tst_id | 🟩 | Identifiant du test qui a été retourné lors de la création du test |

La réponse sera un objet JSON ne contenant aucune propriété spécifique.

**Erreurs :**

| error_code | error_message |
|------------|---------------|
| 304 | Le test est déjà commencé ou n'existe pas |

---

### Ajouter un test de confirmation à un test terminé

| Paramètre | Requis | Valeur |
|-----------|--------|--------|
| act_id | 🟩 | 58 |
| pla_tst_id | 🟩 | Identifiant du test pour lequel un test de confirmation doit être créé (le test doit être déjà terminé) |
| redirect | 🟠 | Si défini sur 1 : La réponse sera un statut HTTP 302 redirigeant vers l'URL de la page de début du test |

**Réponse :**

La réponse sera un objet JSON (sauf si `redirect` est défini sur 1) contenant les propriétés suivantes :
- `tst_url` : l'URL pour démarrer le test de confirmation

---

### Mettre à jour un candidat

| Paramètre | Requis | Valeur |
|-----------|--------|--------|
| act_id | 🟩 | 51 |
| can_id | 🟩 | Identifiant du candidat qui a été retourné lors de la création du candidat |
| gen_id | 🟠 | 1: homme, 2: femme, 3: non renseigné |
| ema | 🟠 | Adresse email du candidat |
| fst_nam | 🟠 | Prénom du candidat |
| lst_nam | 🟠 | Nom du candidat |
| lan_id | 🟠 | Code de langue. Voir l'[annexe](#codes-de-langue) |

---

### Supprimer un candidat

| Paramètre | Requis | Valeur |
|-----------|--------|--------|
| act_id | 🟩 | 24 |
| can_id | 🟠 | Identifiant du candidat à supprimer, doit être fourni sauf si `ema` est fourni |
| ema | 🟠 | Email du candidat à supprimer, doit être fourni sauf si `can_id` est fourni |

La réponse sera un objet JSON contenant uniquement une propriété `success`.

---

## Emails

### Envoyer un email d'inscription

Cette action permet d'envoyer un email d'inscription au candidat immédiatement ou dans le futur. Un expéditeur d'email spécifique peut être fourni.

| Paramètre | Requis | Valeur |
|-----------|--------|--------|
| act_id | 🟩 | 42 |
| pla_tst_ids | 🟠 | Les `pla_tst_ids` des tests pour lesquels l'email d'inscription sera envoyé. Veuillez noter que si vous ne remplissez pas ce champ et que le candidat est inscrit à plusieurs évaluations, le contenu de l'email inclura tous les tests en attente auxquels le candidat est inscrit |
| can_id | 🟠 | Le `can_id` retourné lors de la création du candidat |
| mai_tem_id | 🟩 | L'identifiant du modèle d'email que vous souhaitez utiliser (peut être trouvé dans l'URL lors de l'édition du modèle sur la plateforme) |
| sen_dat | 🟠 | La date à laquelle l'email doit être envoyé, le format doit être YYYY-MM-DD |
| for_sen | 🟠 | Si la valeur est 1 et qu'un `sen_dat` est fourni et qu'il est aujourd'hui ou dans le passé, alors envoyer l'email immédiatement |
| ema_sen | 🟠 | Un expéditeur d'email approuvé avec un statut vérifié |

Un `can_id` ou un `pla_tst_id` doit être fourni. Si les deux sont fournis, le `pla_tst_id` sera utilisé.

La réponse JSON inclut uniquement un champ `success`.

---

## Résultats

### Obtenir les résultats PDF

| Paramètre | Requis | Valeur |
|-----------|--------|--------|
| act_id | 🟩 | 6 |
| tst_frm_id | 🟩 | L'identifiant du test. Pour des raisons de compatibilité, ce paramètre peut être nommé `rea_tst_id` jusqu'au 31/12/2025 |
| ema | 🔷 | Adresse email du candidat |
| pla_tst_id | 🟠 | L'identifiant qui a été retourné lors de la création du test. Si ce paramètre n'est pas fourni, le système recherchera le dernier test passé |
| lan_id | 🟠 | Code de langue utilisé pour le rapport. Voir l'[annexe](#codes-de-langue) |
| redirect | 🟠 | 1: si vous souhaitez recevoir un statut HTTP 302 vous redirigeant vers l'URL du rapport PDF |

**Réponse :**

La réponse sera un objet JSON (sauf si `redirect` est défini sur 1) contenant les propriétés suivantes :
- `pdf_url` : l'URL du rapport PDF

**Erreurs :**

| error_code | error_message |
|------------|---------------|
| 106 | Cette valeur pour `tst_frm_id` n'est pas autorisée |
| 107 | Le candidat n'existe pas |
| 501 | Le candidat n'a pas passé ce test |
| 502 | Le test n'est pas terminé |
| 601 | Le test doit être une évaluation |

---

### Obtenir les résultats sous forme JSON

| Paramètre | Requis | Valeur |
|-----------|--------|--------|
| act_id | 🟩 | 4 |
| tst_frm_id | 🟩 | L'identifiant du test. Pour des raisons de compatibilité, ce paramètre peut être nommé `rea_tst_id` jusqu'au 31/12/2025 |
| ema | 🔷 | Adresse email du candidat |
| pla_tst_id | 🟠 | L'identifiant qui a été retourné lors de la création du test. Si ce paramètre n'est pas fourni, le système recherchera le dernier test passé |

**Réponse :**

La réponse sera un objet JSON contenant les propriétés suivantes :

- `gra_typ_id` : un entier représentant le type de note (voir les valeurs possibles ci-dessous)
- `num_val` : un nombre représentant la valeur numérique de la note
- `max_val` : un nombre représentant la valeur maximale correspondant au `gra_typ_id`
- `txt_des` : une description textuelle de la note (par exemple "550/1000" ou "Intermédiaire – 3/5")
- `no_sto_tim_spe` : la différence exprimée en secondes entre l'heure de début et l'heure de fin du test (ce temps peut ne pas correspondre au temps passé par le candidat sur le test car il peut avoir la possibilité dans certains cas d'arrêter le test et de le redémarrer plus tard)

**Les valeurs possibles pour `gra_typ_id` sont :**

| Valeur | Description |
|--------|-------------|
| 1 | Niveau sur une échelle de 1 à 5 |
| 2 | Score sur 1000 calculé par la théorie de réponse à l'item |
| 3 | Score sur 1000 calculé en moyennant les scores sur chaque question |
| 4 | Nombre de réponses correctes |
| 5 | Plage de score 1-350, 350-700, 700-1000 calculée en utilisant l'IRT |
| 15 | Score sur 100 calculé en moyennant les scores sur chaque question |

---

### Obtenir les détails des résultats sous forme JSON

| Paramètre | Requis | Valeur |
|-----------|--------|--------|
| act_id | 🟩 | 5 |
| tst_frm_id | 🟩 | L'identifiant du test. Pour des raisons de compatibilité, ce paramètre peut être nommé `rea_tst_id` jusqu'au 31/12/2025 |
| ema | 🔷 | Adresse email du candidat |
| pla_tst_id | 🟠 | L'identifiant qui a été retourné lors de la création du test. Si ce paramètre n'est pas fourni, le système recherchera le dernier test passé |
| des_lan_id | 🟠 | Code de langue utilisé pour les détails. Voir l'[annexe](#codes-de-langue) |

**Réponse :**

La réponse sera un objet JSON contenant les propriétés suivantes :
- `des_lan_id` : la langue utilisée pour les détails des résultats. Si aucune description n'est disponible dans la langue demandée, l'anglais sera utilisé par défaut
- `result_details` : un tableau où les clés sont les identifiants de compétences et les valeurs sont un tableau composé de valeurs dont les clés sont :
    - `des` : description de la compétence
    - (pour les tests basés sur l'IRT) `level` : Nombre décimal représentant le niveau obtenu par le candidat. Une valeur de -1 signifie que le candidat n'a pas été testé sur cette compétence
    - (pour les tests basés sur l'IRT) `level_des` : Brève description textuelle des compétences du candidat à ce niveau
    - (pour les tests basés sur l'IRT) `scalemin` : Valeur minimale du niveau
    - (pour les tests basés sur l'IRT) `scalemax` : Valeur maximale du niveau
    - (pour les tests NON basés sur l'IRT) `successrate` : Taux de réussite sur les questions appartenant à cette compétence. C'est une valeur décimale égale à -1 ou entre 0 et 1. Une valeur de -1 signifie que le candidat n'a pas été testé sur cette compétence

---

### Obtenir le statut d'un test

| Paramètre | Requis | Valeur |
|-----------|--------|--------|
| act_id | 🟩 | 10 |
| pla_tst_id | 🟩 | L'identifiant qui a été retourné lors de la création du test |

**Réponse :**

La réponse sera un objet JSON contenant les propriétés suivantes :
- `sta_id` : un entier représentant le statut du test :
    - 1: non commencé
    - 2: commencé
    - 3: terminé
    - 4: notation en attente (pour les tests qui incluent une notation manuelle)

**Erreurs :**

| error_code | error_message |
|------------|---------------|
| 501 | Ce `pla_tst_id` n'existe pas |

---

## Administration

### Créer une session

| Paramètre | Requis | Valeur |
|-----------|--------|--------|
| act_id | 🟩 | 11 |
| des | 🟩 | Description de la session |
| sta_dat | 🟩 | Date de début de la session (format YYYY-MM-DD hh:mm) |
| end_dat | 🟩 | Date de fin de la session (format YYYY-MM-DD hh:mm) |
| psw | 🟠 | Mot de passe de la session |

**Réponse :**

La réponse sera un objet JSON contenant les propriétés suivantes :
- `ses_id` : Un entier représentant l'identifiant unique de la session

---

### Anonymiser un candidat

| Paramètre | Requis | Valeur |
|-----------|--------|--------|
| act_id | 🟩 | 12 |
| ema | 🟩 | L'email du candidat |

Leur prénom, nom et adresse email seront remplacés par des numéros dans la base de données d'Isograd. Cela affecte également leur(s) diplôme(s).

La réponse sera un objet JSON ne contenant aucune propriété spécifique.

---

### Se connecter en tant qu'administrateur

| Paramètre | Requis | Valeur |
|-----------|--------|--------|
| act_id | 🟩 | 13 |
| use_ema_for_adm | 🟠 | 1: si vous souhaitez vous connecter en tant qu'administrateur spécifique. Si non défini, le système choisira un administrateur aléatoire |
| ema | 🟠 | L'email de l'administrateur si `use_ema_for_adm` est défini sur 1 |

> ⚠️ **Soyez très prudent** de ne pas fournir l'accès à cette requête aux candidats car ils accéderaient à votre compte sur la plateforme et pourraient créer des candidats, voir les résultats, ajouter des tests, etc.
>
> ⚠️ Cette méthode, comme toutes les autres, inclut un paramètre optionnel `sub_com_id` pour les clients qui utilisent des sous-comptes. Si le paramètre `sub_com_id` n'est pas fourni, l'administrateur du sous-compte sera connecté en tant qu'administrateur du compte principal, ce qui pourrait soulever de **graves problèmes de confidentialité des données**.

La réponse sera une redirection HTTP vers le module Admin.

---

### Se connecter en tant que candidat

| Paramètre | Requis | Valeur |
|-----------|--------|--------|
| act_id | 🟩 | 38 |
| ema | 🔷 | Email du candidat |

La réponse sera une redirection HTTP vers la page du compte du candidat, où ils peuvent accéder à :
- Tests à passer
- Diplômes
- Rapports d'évaluation
- Questions et réponses correctes pour les tests d'évaluation

---

### Obtenir la liste des tests disponibles

| Paramètre | Requis | Valeur |
|-----------|--------|--------|
| act_id | 🟩 | 14 |
| lan_id | 🟠 | Code de la langue du candidat. Voir l'[annexe](#codes-de-langue) |

**Réponse :**

La réponse sera un objet JSON contenant les propriétés suivantes :
- `tests` : Un tableau des identifiants des tests autorisés

---

### Obtenir les crédits disponibles

| Paramètre | Requis | Valeur |
|-----------|--------|--------|
| act_id | 🟩 | 15 |

**Réponse :**

La réponse sera un objet JSON contenant les propriétés suivantes :
- `credits` : un tableau qui contient pour chaque type de crédit les propriétés suivantes :
    - `des` : description du type de crédit
    - `amount` : montant du crédit

---

# Annexes

## Codes de langue

Les valeurs suivantes doivent être utilisées pour spécifier les langues :
- 1: Français
- 2: Anglais
- 3: Allemand
- 4: Néerlandais
- 5: Espagnol
- 6: Italien
- 7: Grec
- 8: Arabe

---

## Comptes distributeurs

Les comptes distributeurs peuvent effectuer des actions sur les sous-comptes qui leur appartiennent en utilisant les identifiants du compte distributeur. Dans ce cas, un paramètre supplémentaire `sub_com_id` peut être ajouté à la requête pour spécifier sur quel sous-compte l'action doit être effectuée.

Cet entier `sub_com_id` **peut être ajouté à toutes les actions décrites dans la section Action.**

---

## LTI - Notes via Basic Outcome Service

Le service fournisseur LTI Isograd peut retourner des notes via le LTI 1.1 Basic Outcome Service. Cela permet à votre système de recevoir automatiquement la note du candidat à la fin du test. Si vous souhaitez recevoir la note via Basic Outcome Service, veuillez contacter le support Isograd pour demander son activation sur votre compte.

Le LTI Basic Outcome Service n'autorise qu'une note sous forme de nombre décimal entre 0 et 1. Les différents types de notes sont convertis en un nombre décimal entre 0 et 1 selon les règles ci-dessous :
- LEVEL_GRADE_TYPE = 1, le niveau divisé par 10
- SCORE_GRADE_TYPE = 2, le score divisé par 1000
- AVERAGE_GRADE_TYPE = 3, le score divisé par 1000
- NUM_CORRECT_GRADE_TYPE = 4, le pourcentage de réponses correctes, c'est-à-dire : nombre de réponses correctes / nombre de questions
- RANGE_GRADE_TYPE = 5, la valeur moyenne de la plage pertinente divisée par 1000
- ON_100_AVERAGE_GRADE_TYPE = 15, le score divisé par 1000

</div>
