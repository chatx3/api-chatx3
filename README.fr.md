# API ChatX3

> 🇬🇧 English version: [README.md](README.md)

Guide de démarrage rapide pour l'**API ChatX3**.

L'API ChatX3 vous permet d'envoyer une question à l'assistant ChatX3 et de recevoir une réponse formatée en Markdown, spécialisée pour l'**écosystème Sage X3** (support, développement L4G, recherche documentaire).

Ce guide couvre l'authentification, le format des requêtes et des réponses, les limites d'usage et les restrictions actuelles.

---

## Endpoint

```
POST https://akfcgzazfvqipbjvdemn.supabase.co/functions/v1/api-ask
```

Toutes les requêtes utilisent la méthode `POST` avec un corps JSON.

---

## Authentification

Chaque requête doit inclure votre clé API personnelle dans l'en-tête `x-api-key`.

```
x-api-key: <VOTRE_CLE_API>
```

Vous pouvez trouver et copier votre clé API dans **Gestion des utilisateurs → Clé API** (utilisez le bouton *Copier la clé API* à côté de votre utilisateur). Gardez cette clé privée : elle vous identifie et est liée à vos limites d'usage.

Les requêtes sans clé valide sont rejetées avec une réponse **401**.

---

## Requête

### Champs du corps

| Champ             | Requis | Type   | Description                                          |
| ----------------- | ------ | ------ | ---------------------------------------------------- |
| `message_content` | Oui    | string | La question ou le message à envoyer à l'assistant.   |

### Exemple minimal

```json
{
  "message_content": "Comment créer un utilisateur Syracuse ?"
}
```

### Exemple — curl

```bash
curl -X POST "https://akfcgzazfvqipbjvdemn.supabase.co/functions/v1/api-ask" \
  -H "x-api-key: <VOTRE_CLE_API>" \
  -H "Content-Type: application/json" \
  -d '{"message_content": "Comment créer un utilisateur Syracuse ?"}'
```

### Exemple — JavaScript (fetch)

```javascript
const res = await fetch(
  "https://akfcgzazfvqipbjvdemn.supabase.co/functions/v1/api-ask",
  {
    method: "POST",
    headers: {
      "x-api-key": "<VOTRE_CLE_API>",
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      message_content: "Comment créer un utilisateur Syracuse ?"
    })
  }
);

const data = await res.json();
console.log(data.message);
```

### Exemple — Python (requests)

```python
import requests

url = "https://akfcgzazfvqipbjvdemn.supabase.co/functions/v1/api-ask"
headers = {
    "x-api-key": "<VOTRE_CLE_API>",
    "Content-Type": "application/json",
}
payload = {"message_content": "Comment créer un utilisateur Syracuse ?"}

# Les réponses peuvent prendre jusqu'à 120 s, prévoyez un timeout généreux.
response = requests.post(url, json=payload, headers=headers, timeout=130)
response.raise_for_status()

data = response.json()
if data.get("success"):
    print(data["message"])
    print("conversation_id:", data["conversation_id"])
else:
    print("Erreur :", data.get("error"))
```

### Exemple — n8n (nœud HTTP Request)

Configurez un nœud HTTP Request comme suit :

| Paramètre        | Valeur                                                                             |
| ---------------- | ---------------------------------------------------------------------------------- |
| Méthode          | `POST`                                                                             |
| URL              | `https://akfcgzazfvqipbjvdemn.supabase.co/functions/v1/api-ask`                    |
| Authentification | Aucune (utilisez un en-tête, voir ci-dessous)                                      |
| Send Headers     | Activé — ajoutez `x-api-key = <VOTRE_CLE_API>` et `Content-Type = application/json`|
| Send Body        | Activé — Body Content Type : `JSON`                                                |
| Body (JSON)      | `{ "message_content": "Comment créer un utilisateur Syracuse ?" }`                 |
| Timeout          | Réglez à au moins `130000` ms (Options → Timeout)                                  |

La réponse de l'assistant est disponible en aval via `{{ $json.message }}`, accompagnée de `{{ $json.conversation_id }}` et `{{ $json.message_id }}`.

---

## Réponse

### Succès (200 OK)

```json
{
  "success": true,
  "message": "# Création d'un utilisateur Syracuse dans Sage X3 V11\n\nLes utilisateurs Syracuse sont...",
  "message_id": "63debdac-e0c9-42ae-b51d-d0f904603376",
  "conversation_id": "api_20260624_400330"
}
```

| Champ             | Description                                                                                     |
| ----------------- | ----------------------------------------------------------------------------------------------- |
| `success`         | `true` quand la requête a réussi.                                                               |
| `message`         | La réponse de l'assistant, formatée en **Markdown**. Affichez-la en tant que Markdown pour une meilleure lisibilité. |
| `message_id`      | Identifiant unique de la réponse de l'assistant.                                                |
| `conversation_id` | Identifiant de la conversation. Retourné même si vous n'en avez pas fourni un.                  |

La réponse est en **Markdown** (titres, gras, listes, blocs de code). Affichez-la avec un visualiseur Markdown plutôt qu'en texte brut.

---

## Limites d'usage

Pour maintenir le service réactif et le protéger contre les abus, les requêtes sont limitées par utilisateur sur trois fenêtres glissantes :

| Fenêtre  | Limite        |
| -------- | ------------- |
| 4 heures | 50 messages   |
| 7 jours  | 250 messages  |
| 30 jours | 800 messages  |

Lorsqu'une limite est atteinte, l'API retourne une réponse **429** au lieu d'une réponse :

```json
{
  "success": false,
  "rate_limited": true,
  "error": "Vous avez atteint votre limite de 50 messages par 4 heures. Réessayez dans 2 h.",
  "limit": 50,
  "window": "4 hours",
  "retry_at": "2026-06-24T18:30:00.000Z",
  "retry_after_seconds": 7200
}
```

La réponse inclut également un en-tête standard `Retry-After` (en secondes). Votre intégration doit détecter le `429`, lire `retry_after_seconds` (ou `retry_at`) et attendre avant de réessayer plutôt que d'envoyer des requêtes répétées.

---

## Limitations actuelles

L'API est en développement actif. À ce jour, veuillez noter les restrictions suivantes :

- **Pas de gestion de documents.** Vous ne pouvez pas joindre ou téléverser de fichiers (PDF, Word, images, etc.). L'assistant répond uniquement à partir du texte `message_content` et de sa propre base de connaissances. L'entrée par fichier est prévue mais pas encore disponible.
- **Pas de mémoire de conversation.** Chaque requête est traitée indépendamment. Même si un `conversation_id` est retourné, l'assistant n'utilise pas encore l'historique des messages précédents comme contexte. Une question de suivi qui s'appuie sur un échange précédent ne fonctionnera pas comme attendu : reformulez le contexte complet dans chaque `message_content`.
- **Traitement synchrone avec délai.** Les réponses sont générées à la demande et prennent généralement **20 à 40 secondes**, avec un maximum strict de **120 secondes**. Configurez le timeout de votre client à au moins 120 secondes. Si le traitement dépasse cette limite, l'API retourne une erreur et vous devez réessayer.

---

## Réponses d'erreur

| Statut | Signification        | Cause typique                                              |
| ------ | -------------------- | ---------------------------------------------------------- |
| 400    | Requête incorrecte   | `message_content` manquant.                                |
| 401    | Non autorisé         | Clé API manquante ou invalide.                             |
| 429    | Trop de requêtes     | Une limite d'usage a été atteinte (voir ci-dessus).        |
| 502    | Erreur en amont      | Le service IA a retourné une erreur. Réessayez.            |
| 500    | Erreur serveur       | Erreur inattendue, y compris un timeout de traitement. Réessayez. |

Toutes les réponses d'erreur partagent la même structure :

```json
{
  "success": false,
  "error": "Description de ce qui s'est mal passé."
}
```

Vérifiez toujours le champ `success` (et le statut HTTP) avant de lire `message`.

---

## Comportement client recommandé

1. Envoyez toujours l'en-tête `x-api-key`.
2. Utilisez un timeout de requête d'au moins **120 secondes**.
3. Traitez `success === false` et tout statut non-200 comme un échec, et présentez `error` à l'utilisateur.
4. Sur `429`, attendez en utilisant `retry_after_seconds` avant de réessayer.
5. Affichez `message` en **Markdown**.
6. Comme il n'y a pas encore de mémoire de conversation, incluez tout le contexte pertinent directement dans `message_content`.

