---
name: yaml-http-client
description: >
  Générer, configurer et utiliser YamlHttpClient (NuGet: YamlHttpClient) en C#/.NET.
  Déclencher dès que la conversation implique des appels HTTP configurés en YAML, la librairie
  YamlHttpClient, YamlHttpClientFactory, YamlHttpOrchestrator, IYamlHttpClientAccessor,
  ou des pipelines HTTP multi-étapes avec retry/cache/mock/chaos dans un projet .NET.
  Utiliser aussi si l'utilisateur mentionne « yaml http », « orchestration HTTP YAML »,
  « AutoCallAsync », « ExecuteSetAsync », ou demande d'intégrer des appels HTTP via fichier YAML.
---

# YamlHttpClient — Skill

Librairie .NET pour configurer des appels HTTP en YAML avec retry, cache, chaos engineering, mock et orchestration multi-étapes.

**NuGet** : `dotnet add package YamlHttpClient`  
**Source** : https://github.com/anisite/YamlHttpClient

---

## Utilisation de base

```csharp
// Charger depuis un fichier YAML, cibler une clé nommée
YamlHttpClientFactory httpClient = new YamlHttpClientFactory(
    new YamlHttpClientConfigBuilder().LoadFromFile("config.yml", "monAppel"));

// Appel court (recommandé) — intègre automatiquement retry + cache
var response = await httpClient.AutoCallAsync(monObjetInput);
await httpClient.CheckResponseAsync(response); // valide selon check_response dans le YAML

var contenu = await response.Content.ReadAsStringAsync();
```

> Toujours préférer `AutoCallAsync` à `BuildRequestMessage + SendAsync` — seul `AutoCallAsync` active correctement retry et cache ensemble.

### Loaders disponibles

```csharp
new YamlHttpClientConfigBuilder().LoadFromFile("config.yml",    "cle");
new YamlHttpClientConfigBuilder().LoadFromString(yamlString,    "cle");
new YamlHttpClientConfigBuilder().LoadFromBytes(yamlBytes,      "cle");
```

---

## Structure YAML complète

```yaml
http_client:
  monAppel:
    method: POST                                  # GET, POST, PUT, DELETE, PATCH...
    url: https://api.exemple.com/{{place}}/data   # Handlebars supporté

    use_default_credentials: true                 # NTLM (identité app pool)
    # auth_basic: 'username:password'             # Authentification Basic

    headers:
      CodeNT: '{{System.CodeNT}}'
      Accept: 'application/json'

    content:
      json_content: |                             # Corps JSON avec Handlebars
        {
          "val": "{{val1}}",
          "obj": {{{Json .}}}
        }
      # string_content: 'texte brut {{val}}'
      # form_content:
      #   champ1: valeur1
      #   champ2: '{{val}}'

    check_response:
      throw_exception_if_body_contains_any:
        - erreur
      throw_exception_if_body_not_contains_all:
        - success

    retry:
      max_retries: 3
      delay_milliseconds: 500
      retry_on_status_codes: [500, 502, 503, 504]

    cache:
      enabled: true
      ttl_seconds: 600                            # Clé = Method + URL + hash body; 2xx seulement

    chaos:                                        # Pour tests de résilience (staging/dev)
      enabled: false
      injection_rate_percentage: 30
      delay_milliseconds: 300                     # Toujours appliqué si > 0
      simulate_status_code: 503
      # simulate_network_exception: true          # Lève HttpRequestException

    mock:                                         # Court-circuite le réseau entièrement
      enabled: false
      status_code: 200
      headers:
        Content-Type: 'application/json'
      body: '{ "result": "mocked" }'
```

---

## Injection de dépendances

```csharp
// Program.cs / Startup.cs
services.AddYamlHttpClientAccessor();

// Dans un service
public class MonService
{
    private readonly IYamlHttpClientAccessor _client;

    public MonService(IYamlHttpClientAccessor client)
    {
        _client = client;
        _client.HttpClientSettings = new YamlHttpClientConfigBuilder()
            .LoadFromFile("config.yml", "monAppel");
        _client.HandlebarsProvider = YamlHttpClientFactory.CreateDefaultHandleBars();
    }

    public async Task<string> AppelerAsync(object data)
    {
        var response = await _client.AutoCallAsync(data);
        await _client.CheckResponseAsync(response);
        return await response.Content.ReadAsStringAsync();
    }
}
```

---

## Orchestrateur multi-étapes (`YamlHttpOrchestrator`)

Exécute des pipelines séquentiels définis dans `http_client_set`. Chaque étape alimente les suivantes via Handlebars. Requiert .NET 6+.

### YAML — pipeline

```yaml
http_client_set:
  pipeline_utilisateurs:
    sequence:
      - http_client: get_token
      - http_client: get_users
        as: get_users             # alias optionnel; défaut = nom du client
    data_adapter:
      template: |
        {
          "total": {{get_users.body.total}},
          "items": {{{Json get_users.body.records}}}
        }

http_client:
  get_token:
    method: POST
    url: https://auth.exemple.com/token
    content:
      json_content: |
        { "client_id": "{{input.clientId}}", "client_secret": "{{input.secret}}" }

  get_users:
    method: GET
    url: https://api.exemple.com/users
    headers:
      Authorization: 'Bearer {{get_token.body.access_token}}'
    retry:
      max_retries: 2
      delay_milliseconds: 500
      retry_on_status_codes: [500, 502, 503]
```

> Si `data_adapter.template` est absent, `ExecuteSetAsync` retourne l'objet agrégé complet en JSON brut.

### Modèle de données dans les templates

```
{
  input:      { ...inputData... },
  get_token:  { body: {...}, headers: {...}, url: "https://..." },
  get_users:  { body: {...}, headers: {...}, url: "https://..." }
}
```

### C# — `ExecuteSetAsync` (entrée principale)

```csharp
var yaml   = File.ReadAllText("pipeline.yml");
var config = new DeserializerBuilder()
    .WithNamingConvention(NullNamingConvention.Instance)
    .IgnoreUnmatchedProperties()
    .Build()
    .Deserialize<YamlHttpClientConfigBuilder>(yaml);

var orchestrator = new YamlHttpOrchestrator(YamlHttpClientFactory.CreateDefaultHandleBars());

var result = await orchestrator.ExecuteSetAsync(
    setName:            "pipeline_utilisateurs",
    config:             config,
    inputData:          new { clientId = "mon-app", secret = "s3cr3t" },
    defaultHttpTimeout: TimeSpan.FromSeconds(30),
    ct:                 CancellationToken.None
);

// URLs effectivement appelées
foreach (var url in orchestrator.LastCalledUrls)
    Console.WriteLine($"Appelé : {url}");
```

### C# — `ExecuteSequenceAsync` (séquence définie en C#)

```csharp
var steps = new List<dynamic>
{
    new { HttpClient = "get_token",  As = "get_token"  },
    new { HttpClient = "get_users",  As = "get_users"  }
};

var result = await orchestrator.ExecuteSequenceAsync(
    inputData:           new { clientId = "mon-app", secret = "s3cr3t" },
    sequenceAppels:      steps,
    dictClientsConfig:   config.HttpClient,
    dataAdapterTemplate: "{{get_users.body.records}}",
    defaultHttpTimeout:  TimeSpan.FromSeconds(30),
    ct:                  CancellationToken.None
);
```

### Options par défaut de l'orchestrateur

| Option | Défaut |
|---|---|
| Cache activé | `true` |
| Cache TTL | `1200` s (20 min) |
| Max retries | `3` |
| Retry sur codes | `500, 501, 502, 503, 504` |

Passer `null` pour `options` pour utiliser ces valeurs.

---

## Helpers Handlebars

Disponibles dans URL, headers, body et `data_adapter`.

| Helper | Exemple | Résultat |
|---|---|---|
| `{{{Json obj}}}` | `{{{Json obj}}}` | `{"test":1}` |
| `{{{Json obj ">forcestring"}}}` | — | `"{\"test\":1}"` |
| `{{{Json . ">flatten;.;[{0}]"}}}` | — | `{"obj[0].test":1}` |
| `{{{Json . ">flatten;_;_{0}"}}}` | — | `{"obj_0_test":1}` |
| `{{{Base64 monObjet}}}` | — | encodage Base64 |
| `{{#ifCond A '=' B}}...{{/ifCond}}` | — | bloc conditionnel |

**Opérateurs `ifCond`** : `=`, `==`, `!=`, `<>`, `<`, `>`, `contains`, `in`

**Cache de templates** : compilés une seule fois via `CompileWithCache`.  
Appeler `HandleBarsExtensions.ClearTemplateCache()` pour forcer un rechargement (hot reload de config YAML).

---

## Modes chaos — tableau de référence

| Option YAML | Effet | Déclenché par le taux ? |
|---|---|---|
| `delay_milliseconds` | Délai fixe sur chaque appel | Non — toujours appliqué |
| `simulate_status_code` | Retourne une fausse réponse HTTP | Oui (`injection_rate_percentage`) |
| `simulate_network_exception: true` | Lève `HttpRequestException` | Oui |

Combiner chaos + retry pour valider que la politique de retry récupère bien les pannes.

---

## Règles et pièges à éviter

- **Ne pas utiliser** `BuildRequestMessage + SendAsync` séparément si retry ou cache sont activés — utiliser `AutoCallAsync`.
- Le cache est partagé entre toutes les instances pour la même URL. Clé = `Method + URL + hash(body)`.
- `mock.enabled: true` court-circuite entièrement le réseau — compatible avec `check_response` et l'orchestrateur.
- `use_default_credentials: true` = NTLM automatique via l'identité du pool applicatif (contexte IIS on-prem).
- NTLM avec utilisateur/mot de passe explicite : **pas encore supporté** (roadmap).
- Authentification par certificat client : **pas encore supportée** (roadmap).
