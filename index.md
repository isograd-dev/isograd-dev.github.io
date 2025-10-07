# Documentation API Isograd

## Vue d'ensemble

L'API Isograd permet de gérer des candidats, créer et administrer des tests, et récupérer des résultats via une interface REST.

**URL de base:**
- Test: `https://recette.isograd.com/api/usage`
- Production: `https://app.isograd.com/api/usage`

---

## Authentification

L'API utilise OAuth 2.0 avec le flux Client Credentials.

### Obtenir un token d'accès

```http
POST https://auth.isograd.com/oauth2/token?grant_type=client_credentials&scopes=usage
Content-Type: application/x-www-form-urlencoded

client_id={votre_client_id}
client_secret={votre_client_secret}
```

**Réponse:**
```json
{
  "access_token": "eyJhbGc...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

### Utiliser le token

Incluez le token dans l'en-tête Authorization de toutes vos requêtes:

```http
Authorization: Bearer {access_token}
```

---

## Gestion des candidats

### Créer un candidat

Crée un nouveau candidat dans le système Isograd.

**Endpoint:** `POST /api/usage`

**Paramètres:**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `act_id` | integer | ✅ | ID de l'action (1 pour créer un candidat) |
| `ema` | string | ✅ | Adresse email du candidat |
| `fst_nam` | string | ✅ | Prénom du candidat |
| `lst_nam` | string | ✅ | Nom du candidat |
| `gen_id` | integer | ✅ | Genre (1: homme, 2: femme, 3: non renseigné) |
| `lan_id` | integer | ❌ | Code langue (voir [codes langues](#codes-langues)) |
| `ext_id` | string | ❌ | Votre identifiant interne pour le candidat |
| `dep_id` | integer | ❌ | ID du groupe Isograd |
| `ext_dep_id` | string | ❌ | ID du groupe dans votre système |
| `psw` | string | ❌ | Mot de passe (généré aléatoirement si non fourni) |
| `no_psw_rst` | integer | ❌ | Si 1: le mot de passe est haché et non modifiable |
| `job_id` | integer | ❌ | ID du profil métier |
| `xtr_tim_fac` | integer | ❌ | Pourcentage de temps supplémentaire (ex: 50 pour +50%) |

**Exemple de requête:**

```bash
curl -X POST https://recette.isograd.com/api/usage \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "act_id=1" \
  -d "ema=john.doe@example.com" \
  -d "fst_nam=John" \
  -d "lst_nam=Doe" \
  -d "gen_id=1"
```

**Réponse réussie (200):**

```json
{
  "success": true,
  "can_id": 1381134,
  "lgn_url": "https://recette.isograd.com/continuelogin/CandidateContinueLogin?param=UVR1WjgwRXBwU0",
  "error_message": null
}
```

**Codes d'erreur:**

| Code | Message |
|------|---------|
| 201 | Le candidat n'a pas pu être créé (paramètre invalide) |
| 202 | Un candidat avec cet email existe déjà |
| 203 | Le `dep_id` spécifié est introuvable |

---

### Modifier un candidat

Met à jour les informations d'un candidat existant.

**Endpoint:** `POST /api/usage`

**Paramètres:**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `act_id` | integer | ✅ | 51 |
| `can_id` | integer | ✅ | ID du candidat à modifier |
| `gen_id` | integer | ❌ | Genre (1: homme, 2: femme, 3: non renseigné) |
| `ema` | string | ❌ | Nouvelle adresse email |
| `fst_nam` | string | ❌ | Nouveau prénom |
| `lst_nam` | string | ❌ | Nouveau nom |
| `lan_id` | integer | ❌ | Code langue |

---

### Supprimer un candidat

Supprime définitivement un candidat du système.

**Endpoint:** `POST /api/usage`

**Paramètres:**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `act_id` | integer | ✅ | 24 |
| `can_id` | integer | ➡️ | ID du candidat (requis si `ema` non fourni) |
| `ema` | string | ➡️ | Email du candidat (requis si `can_id` non fourni) |

---

### Anonymiser un candidat

Anonymise les données personnelles d'un candidat (RGPD).

**Endpoint:** `POST /api/usage`

**Paramètres:**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `act_id` | integer | ✅ | 12 |
| `ema` | string | ✅ | Email du candidat à anonymiser |

---

## Gestion des tests

### Ajouter un test à un candidat

Assigne un test à un candidat existant.

**Endpoint:** `POST /api/usage`

**Paramètres:**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `act_id` | integer | ✅ | 2: refuser si test non terminé existe<br>9: créer même si test non terminé existe |
| `tst_frm_id` | integer | ✅ | Identifiant du test |
| `ema` | string | ✅ | Email du candidat |
| `nee_ful_scr` | integer | ❌ | 1: mode plein écran requis |
| `html` | integer | ❌ | 1: réponse en HTML (balise `<a>`) |
| `redirect` | integer | ❌ | 1: redirection HTTP 302 vers le test |
| `ses_id` | integer | ❌ | ID de session à associer |
| `add_pro` | integer | ❌ | 1: ajouter surveillance à distance (coût supplémentaire) |
| `max_num_tst` | integer | ❌ | Nombre maximum de tentatives |
| `max_num_tst_per` | integer | ❌ | Nombre maximum de tentatives par X jours |
| `rtn_pag` | string | ❌ | URL de redirection après le test |
| `cal_bac_pag` | string | ❌ | URL de callback (GET) à la fin du test |
| `cpf_id` | string | ❌ | ID du dossier CPF |
| `nb_test_by_credit` | integer | ❌ | Utiliser un pack de crédits (2,3,4) |

**Réponse réussie:**

```json
{
  "success": true,
  "tst_url": "https://recette.isograd.com/test/start/...",
  "pla_tst_id": 987654,
  "error_message": null
}
```

**Codes d'erreur:**

| Code | Message |
|------|---------|
| 106 | Ce `tst_frm_id` n'est pas autorisé |
| 107 | Ce candidat n'existe pas |
| 301 | Vous n'avez plus de crédits pour ce type de test |
| 402 | Le candidat a atteint `max_num_tst` pour ce test |

---

### Créer un candidat et assigner un test

Action combinée qui crée un candidat et lui assigne immédiatement un test.

**Endpoint:** `POST /api/usage`

**Paramètres:**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `act_id` | integer | ✅ | 8 |
| ... | ... | ... | Tous les paramètres de "Créer un candidat" et "Ajouter un test" |

**Réponse:** Identique à "Ajouter un test à un candidat"

---

### Ajouter/Retirer la surveillance en ligne

Modifie les options de surveillance pour un test existant.

**Endpoint:** `POST /api/usage`

**Paramètres:**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `act_id` | integer | ✅ | 16: ajouter surveillance<br>17: retirer surveillance |
| `pla_tst_id` | integer | ✅ | ID du test |

**Codes d'erreur:**

| Code | Message |
|------|---------|
| 303 | Vous n'avez plus de crédits de surveillance |

---

### Supprimer un test

Supprime un test non commencé.

**Endpoint:** `POST /api/usage`

**Paramètres:**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `act_id` | integer | ✅ | 23 |
| `pla_tst_id` | integer | ✅ | ID du test |

**Codes d'erreur:**

| Code | Message |
|------|---------|
| 304 | Le test est déjà commencé ou n'existe pas |

---

### Créer un test de confirmation

Crée un test de confirmation pour un test déjà terminé.

**Endpoint:** `POST /api/usage`

**Paramètres:**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `act_id` | integer | ✅ | 58 |
| `pla_tst_id` | integer | ✅ | ID du test complété |
| `redirect` | integer | ❌ | 1: redirection HTTP 302 |

**Réponse:**

```json
{
  "success": true,
  "tst_url": "https://recette.isograd.com/test/confirmation/...",
  "error_message": null
}
```

---

### Obtenir le statut d'un test

Récupère l'état actuel d'un test.

**Endpoint:** `POST /api/usage`

**Paramètres:**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `act_id` | integer | ✅ | 10 |
| `pla_tst_id` | integer | ✅ | ID du test |

**Réponse:**

```json
{
  "success": true,
  "sta_id": 3,
  "error_message": null
}
```

**Valeurs de `sta_id`:**
- `1`: Non commencé
- `2`: En cours
- `3`: Terminé
- `4`: Correction en attente

**Codes d'erreur:**

| Code | Message |
|------|---------|
| 501 | Ce `pla_tst_id` n'existe pas |

---

## Résultats

### Obtenir les résultats (JSON)

Récupère les résultats d'un test sous forme JSON.

**Endpoint:** `POST /api/usage`

**Paramètres:**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `act_id` | integer | ✅ | 4 |
| `tst_frm_id` | integer | ✅ | ID du type de test |
| `ema` | string | ✅ | Email du candidat |
| `pla_tst_id` | integer | ❌ | ID du test spécifique (sinon: dernier test) |

**Réponse:**

```json
{
  "success": true,
  "gra_typ_id": 2,
  "num_val": 750,
  "max_val": 1000,
  "txt_des": "750/1000",
  "no_sto_tim_spe": 3600,
  "error_message": null
}
```

**Types de notation (`gra_typ_id`):**

| ID | Description |
|----|-------------|
| 1 | Niveau sur échelle 1-5 |
| 2 | Score /1000 (IRT) |
| 3 | Score /1000 (moyenne) |
| 4 | Nombre de réponses correctes |
| 5 | Plage de score (1-350, 350-700, 700-1000) |
| 15 | Score /100 (moyenne) |

**Codes d'erreur:**

| Code | Message |
|------|---------|
| 106 | `tst_frm_id` non autorisé |
| 107 | Candidat inexistant |
| 501 | Candidat n'a pas passé ce test |
| 502 | Test non terminé |
| 601 | Le test doit être une évaluation |

---

### Obtenir les résultats détaillés (JSON)

Récupère les résultats détaillés par compétence.

**Endpoint:** `POST /api/usage`

**Paramètres:**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `act_id` | integer | ✅ | 5 |
| `tst_frm_id` | integer | ✅ | ID du type de test |
| `ema` | string | ✅ | Email du candidat |
| `pla_tst_id` | integer | ❌ | ID du test spécifique |
| `des_lan_id` | integer | ❌ | Code langue pour les descriptions |

**Réponse:**

```json
{
  "success": true,
  "des_lan_id": 2,
  "result_details": {
    "45": {
      "des": "Programming logic",
      "level": 3.5,
      "level_des": "Intermediate level",
      "scalemin": 1,
      "scalemax": 5
    },
    "46": {
      "des": "Data structures",
      "successrate": 0.85
    }
  },
  "error_message": null
}
```

---

### Obtenir le rapport PDF

Génère ou récupère l'URL du rapport PDF des résultats.

**Endpoint:** `POST /api/usage`

**Paramètres:**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `act_id` | integer | ✅ | 6 |
| `tst_frm_id` | integer | ✅ | ID du type de test |
| `ema` | string | ✅ | Email du candidat |
| `pla_tst_id` | integer | ❌ | ID du test spécifique |
| `lan_id` | integer | ❌ | Code langue du rapport |
| `redirect` | integer | ❌ | 1: redirection HTTP 302 vers le PDF |

**Réponse:**

```json
{
  "success": true,
  "pdf_url": "https://recette.isograd.com/reports/pdf/...",
  "error_message": null
}
```

---

## Emails

### Envoyer un email d'inscription

Envoie un email d'inscription au candidat.

**Endpoint:** `POST /api/usage`

**Paramètres:**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `act_id` | integer | ✅ | 42 |
| `mai_tem_id` | integer | ✅ | ID du modèle d'email |
| `can_id` | integer | ➡️ | ID du candidat |
| `pla_tst_ids` | string | ❌ | IDs des tests (séparés par virgule) |
| `sen_dat` | string | ❌ | Date d'envoi (YYYY-MM-DD) |
| `for_sen` | integer | ❌ | 1: envoyer immédiatement si `sen_dat` est passée |
| `ema_sen` | string | ❌ | Email expéditeur approuvé |

---

## Administration

### Créer une session

Crée une session d'examens avec dates et mot de passe.

**Endpoint:** `POST /api/usage`

**Paramètres:**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `act_id` | integer | ✅ | 11 |
| `des` | string | ✅ | Description de la session |
| `sta_dat` | string | ✅ | Date de début (YYYY-MM-DD HH:MM) |
| `end_dat` | string | ✅ | Date de fin (YYYY-MM-DD HH:MM) |
| `psw` | string | ❌ | Mot de passe de la session |

**Réponse:**

```json
{
  "success": true,
  "ses_id": 1234,
  "error_message": null
}
```

---

### Se connecter en tant qu'administrateur

Génère une URL de connexion automatique au back-office.

**Endpoint:** `POST /api/usage`

**Paramètres:**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `act_id` | integer | ✅ | 13 |
| `use_ema_for_adm` | integer | ❌ | 1: connexion en tant qu'admin spécifique |
| `ema` | string | ❌ | Email admin (si `use_ema_for_adm` = 1) |

⚠️ **Attention:** Ne jamais exposer cette action aux candidats. Utiliser `sub_com_id` si vous gérez des sous-comptes.

---

### Se connecter en tant que candidat

Génère une URL de connexion automatique à l'espace candidat.

**Endpoint:** `POST /api/usage`

**Paramètres:**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `act_id` | integer | ✅ | 38 |
| `ema` | string | ✅ | Email du candidat |

---

### Obtenir la liste des tests disponibles

Récupère tous les tests autorisés pour votre compte.

**Endpoint:** `POST /api/usage`

**Paramètres:**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `act_id` | integer | ✅ | 14 |
| `lan_id` | integer | ❌ | Code langue |

**Réponse:**

```json
{
  "success": true,
  "tests": [101, 102, 103, 201, 305],
  "error_message": null
}
```

---

### Obtenir les crédits disponibles

Consulte votre solde de crédits par type.

**Endpoint:** `POST /api/usage`

**Paramètres:**

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `act_id` | integer | ✅ | 15 |

**Réponse:**

```json
{
  "success": true,
  "credits": [
    {
      "des": "Tests Python",
      "amount": 150
    },
    {
      "des": "Tests Excel",
      "amount": 75
    }
  ],
  "error_message": null
}
```

---

## Annexes

### Codes langues

| Code | Langue |
|------|--------|
| 1 | Français |
| 2 | Anglais |
| 3 | Allemand |
| 4 | Néerlandais |
| 5 | Espagnol |
| 6 | Italien |
| 7 | Grec |
| 8 | Arabe |

---

### Comptes distributeurs

Les comptes distributeurs peuvent effectuer des actions sur leurs sous-comptes en ajoutant le paramètre `sub_com_id` à n'importe quelle requête.

**Exemple:**
```http
POST /api/usage
...
act_id=1
sub_com_id=456
ema=candidate@example.com
...
```

---

### Codes d'erreur généraux

| Code | Message |
|------|---------|
| 103 | Le paramètre `act_id` n'est pas valide |
| 105 | Le paramètre "{nom}" est requis |

---

## Exemple complet (Python)

```python
import requests

# 1. Authentification
credentials = {
    'client_id': 'votre_client_id',
    'client_secret': 'votre_client_secret',
}
auth_url = 'https://auth.isograd.com/oauth2/token?grant_type=client_credentials&scopes=usage'
response = requests.post(auth_url, data=credentials)
access_token = response.json()['access_token']

# 2. Configuration
headers = {'Authorization': f'Bearer {access_token}'}
api_url = 'https://recette.isograd.com/api/usage'

# 3. Créer un candidat
payload = {
    'act_id': 1,
    'gen_id': 3,
    'ema': 'john.doe@example.com',
    'fst_nam': 'John',
    'lst_nam': 'Doe',
}
response = requests.post(api_url, data=payload, headers=headers)
result = response.json()

if result['success']:
    can_id = result['can_id']
    print(f"Candidat créé avec l'ID: {can_id}")
    print(f"URL de connexion: {result['lgn_url']}")
    
    # 4. Assigner un test
    test_payload = {
        'act_id': 2,
        'tst_frm_id': 101,
        'ema': 'john.doe@example.com',
        'redirect': 1
    }
    test_response = requests.post(api_url, data=test_payload, headers=headers)
    test_result = test_response.json()
    
    if test_result['success']:
        print(f"Test assigné: {test_result['tst_url']}")
else:
    print(f"Erreur: {result['error_message']}")
```

---

## Support

Pour toute question ou demande d'activation de fonctionnalités, contactez le support Isograd.
