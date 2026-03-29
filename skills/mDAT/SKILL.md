---
name: mdat
description: >
  Générer, écrire et maintenir des tests unitaires et d'intégration avec mdAT (NuGet: MDAT),
  la librairie de tests automatiques basée sur des fichiers Markdown pour MSTestV2 .NET.
  Déclencher dès que la conversation implique [MarkdownTest], Verify.Assert, des fichiers .md
  de tests, ObjectOrException, ReturnsOrThrows/ReturnsOrThrowsAsync, la génération automatique
  de fichiers Markdown de cas de test, ou toute référence à « mdAT », « MDAT », « markdown test »,
  « cas de test YAML », ou des tests pilotés par des fichiers Markdown dans un projet .NET MSTest.
  Utiliser aussi si l'utilisateur demande d'écrire des tests paramétrés sans [DataRow] / [InlineData].
---

# mdAT — Skill

Librairie MSTestV2 .NET pour piloter les tests unitaires et d'intégration depuis des fichiers Markdown contenant des blocs YAML. Les cas de test sont co-localisés dans un seul fichier `.md` lisible par les développeurs ET les analystes.

**NuGet** : `dotnet add package MDAT`  
**Source** : https://github.com/MTESSDev/mdAT  
**Cibles** : net6.0, net7.0, net8.0, net9.0

---

## Workflow en 5 étapes

1. Créer un test MSTestV2 normal (`[TestClass]`, `[TestMethod]`)
2. Ajouter les paramètres souhaités à la méthode de test
3. Décorer avec `[MarkdownTest("~/Tests/{method}.md")]`
4. Entourer l'appel testé avec `Verify.Assert()`
5. Lancer une fois → le fichier `.md` est **auto-généré**; modifier les cas et relancer

---

## Structure d'un test mdAT

```csharp
[TestClass]
public class MonService
{
    [TestMethod]
    [MarkdownTest("~/Tests/{method}.md")]   // {method} → kebab-case du nom de méthode
    public async Task Calculer(int val1, int val2, string expected)
    {
        _ = await Verify.Assert(
            () => Task.FromResult(MonUtil.Calculer(val1, val2)),
            expected);
    }
}
```

Fichier `Tests/calculer.md` généré automatiquement à la première exécution :

```md
# Calculer

## Case 1

Description

``````yaml
val1: 0
val2: 0
expected:
  verify:
   - type: match
     data: null
``````
```

---

## Syntaxe YAML des cas de test

### Paramètre simple (scalaire)

```yaml
val1: 42
val2: 100
expected: 142
```

> Quand `expected` est un scalaire (string, int…), `Verify.Assert` accepte directement la valeur comme raccourci de `verify[0].data`.

### Paramètre `Expected` complet

```yaml
expected:
  verify:
    - type: match          # seul type supporté actuellement
      jsonPath: $          # JSONPath sur le résultat sérialisé; défaut = "$"
      allowAdditionalProperties: false
      data: 42             # valeur JSON attendue (scalaire, objet YAML, ou JSON inline)
```

### Plusieurs assertions sur le même résultat

```yaml
expected:
  verify:
    - type: match
      jsonPath: $.items.length()   # compte les éléments d'un tableau
      data: 3
    - type: match
      jsonPath: $.items[*]         # wildcard — retourne toute la liste
      allowAdditionalProperties: true
      data:
        - val1
        - val2
        - val3
```

### Valeur attendue multi-lignes (JSON inline ou YAML)

```yaml
expected:
  verify:
    - type: match
      data: |
        {"id":1,"nom":"Alice"}
```

```yaml
expected:
  verify:
    - type: match
      data:
        id: 1
        nom: Alice
```

### Tester une exception attendue

```yaml
expected:
  verify:
    - type: match
      allowAdditionalProperties: true
      data:
        ClassName: System.ArgumentOutOfRangeException
        Message: valeur hors plage
```

> `Verify.Assert` attrape l'exception, la sérialise, et la compare au `data` attendu. `allowAdditionalProperties: true` ignore les champs supplémentaires (StackTrace, etc.).

### Exceptions supportées par `ReturnsOrThrows`

```yaml
# Dans un paramètre ObjectOrException<T>
monObjet:
  Exception:
    ClassName: System.InvalidOperationException
    Message: État invalide
    ParamName: null
    ObjectName: null
```

Exceptions supportées (liste partielle — voir `ReturnsOrThrows.cs`) :
`System.Exception`, `System.ArgumentNullException`, `System.ArgumentOutOfRangeException`,
`System.InvalidOperationException`, `System.NotImplementedException`,
`System.IO.FileNotFoundException`, `System.Data.DataException`, et ~60 autres types BCL/Data/IO.

---

## Fichier externe avec `!include`

Inclure un fichier YAML, JSON, ou binaire directement depuis le fichier `.md` :

```yaml
# Inclure un objet depuis un fichier YAML
form: !include ext_form.yml

# Inclure un fichier binaire → désérialisé en byte[] (base64)
fichier: !include HelloWorld.pdf

# Inclure dans expected (compare les bytes)
expected:
  verify:
    - type: match
      data: !include HelloWorld.pdf
```

| Extension | Comportement |
|---|---|
| `.yml` / `.yaml` | Désérialisé vers le type C# du paramètre |
| `.json` | Retourné comme `string` |
| Autre (`.pdf`, `.docx`…) | Converti en base64 (`byte[]`) |

---

## `ObjectOrException<T>` avec Moq

Permet de piloter un mock depuis le fichier Markdown : soit retourner une valeur, soit lever une exception.

```csharp
[TestMethod]
[MarkdownTest("~/Tests/{method}.md")]
public async Task TestMock(
    ObjectOrException<MonDto> dto,
    Expected expected)
{
    var mock = new Mock<IMonService>();
    mock.Setup(s => s.GetAsync()).ReturnsOrThrowsAsync(dto);

    _ = await Verify.Assert(() => Task.FromResult(mock.Object.ReturnVal()), expected);
}
```

```yaml
# Cas normal — retourne une valeur
dto:
  Value:
    Id: 1
    Nom: Alice

# Cas exception — lève System.Exception
dto:
  Exception:
    ClassName: System.Exception
    Message: Erreur simulée
    ParamName: null
    ObjectName: null
```

**Extensions Moq disponibles :**

| Méthode | Signature setup cible |
|---|---|
| `ReturnsOrThrows` | `ISetup<TMock, TResult>` (synchrone) |
| `ReturnsOrThrowsAsync` | `IReturnsThrows<TMock, Task<TResult>>` (async) |

---

## Contrôle de l'exécution des cas

| Marqueur dans l'en-tête du bloc YAML | Effet |
|---|---|
| _(aucun)_ | Cas exécuté normalement |
| `selected` | **Seul** ce cas (ou ces cas) est exécuté — utile pour déboguer |
| `skipped` | Cas ignoré (toute variante : `(skipped)`, `[skipped]`, `.skipped.`, etc.) |

```md
## Cas normal
``````yaml
val1: 1
val2: 2
expected: 3
``````

## Cas focus (seul ce cas sera exécuté)
``````yaml (selected)
val1: 99
val2: 1
expected: 100
``````

## Cas ignoré
``````yaml skipped
val1: 0
val2: 0
expected: -1
``````
```

---

## `generateExpectedData` — capturer le résultat réel

Utile pour bootstrapper l'`expected` d'un test complexe sans le saisir manuellement.

```yaml
expected:
  generateExpectedData: ./Tests/Generated/mon-output.yml   # chemin relatif au projet
  verify:
    - type: match
      data: null   # premier run : échec attendu, le fichier sera généré
```

Au run suivant, `mon-output.yml` contiendra le résultat sérialisé. Copier son contenu dans `data:` puis supprimer `generateExpectedData`.

---

## `jsonPath` — expressions supportées

```yaml
jsonPath: $                      # racine (défaut)
jsonPath: $.monChamp             # propriété simple
jsonPath: $.items[0].id          # accès par index
jsonPath: $.items[*]             # wildcard — retourne un tableau
jsonPath: $.items.length()       # compte les éléments
jsonPath: $.items.length()       # fonctionne aussi sur les strings (longueur)
```

---

## Nommage du fichier `.md`

| Syntaxe | Résolution |
|---|---|
| `~/Tests/{method}.md` | `{method}` → kebab-case du nom de méthode |
| `~/Tests/mon-fichier.md` | Chemin fixe relatif à la racine du projet test |
| `~\Tests\{method}.md` | Séparateur Windows également accepté |

`~` se résout vers `../../..` depuis `bin/Debug/netX.0/`, soit la racine du projet de test.

**Exemples de conversion kebab-case :**

| Nom de méthode | Fichier généré |
|---|---|
| `Calculer` | `calculer.md` |
| `Multi_level` | `multi-level.md` |
| `TestObjectOrExceptionAsync` | `test-object-or-exception-async.md` |

---

## Types C# supportés en paramètre

Tout type désérialisable par YamlDotNet est supporté. Types testés et validés :

- Scalaires : `int`, `string`, `bool`, `Guid`, `long?`, `DateOnly`
- Collections : `List<T>`, `IEnumerable<T>`, `Dictionary<string, T>`, `Dictionary<object, object>`
- POCO multi-niveaux avec récursion (protection à 10 niveaux de profondeur)
- `byte[]` et `byte[]?` (null supporté)
- `KeyValuePair<K,V>` (via `key:` / `value:` dans le YAML)
- `StringValues` (`Microsoft.Extensions.Primitives`) — converti depuis liste YAML
- `JsonDocument`, `JsonElement`, `JsonNode`, `JsonArray`, `JsonValue` (`System.Text.Json`)
- `Stream` → sérialisé en base64 dans `Verify.Assert`
- `ObjectOrException<T>` — piloter Moq depuis le YAML

### Ajouter un convertisseur YAML personnalisé

```csharp
// Dans [ClassInitialize] ou avant la première exécution des tests
MdatConfig.AddYamlTypeConverter(new MonTypeConverter());

// Avec positionnement explicite
MdatConfig.AddYamlTypeConverter(new MonTypeConverter(), e => e.OnTop());
```

---

## Nommage des tests dans Visual Studio / Azure DevOps

Le nom affiché suit le pattern :

```
NomMéthode@~/Tests/mon-fichier.md_Heading1_Heading2_Heading3
```

Les titres Markdown (`##`, `###`, etc.) alimentent le nom du test. Les renseigner clairement améliore la lisibilité dans les rapports Azure DevOps et Test Explorer.

---

## Configuration `.csproj` requise

```xml
<!-- Les fichiers .md doivent être copiés dans le répertoire de sortie -->
<ItemGroup>
  <Page Include="Tests\mon-test.md">
    <CopyToOutputDirectory>Always</CopyToOutputDirectory>
  </Page>
</ItemGroup>

<!-- Dossier pour les fichiers générés -->
<ItemGroup>
  <Folder Include="Tests\Generated\" />
</ItemGroup>

<!-- Documentation XML pour la génération automatique des commentaires dans le .md -->
<PropertyGroup>
  <GenerateDocumentationFile>True</GenerateDocumentationFile>
</PropertyGroup>
```

Les commentaires XML `/// <summary>` et `/// <param name="...">` sont utilisés pour enrichir le fichier `.md` auto-généré.

---

## Règles et pièges à éviter

- **Blocs YAML à 6 backticks** : mdAT parse uniquement les blocs de fences à exactement 6 backticks. Les blocs à 3 backticks sont ignorés.
- **`{method}` kebab-case** : `MaMethodeTest` → `ma-methode-test.md`. Un mauvais nom de fichier = aucun cas chargé silencieusement.
- **Fichier vide au départ** : ne pas créer le fichier `.md` manuellement; laisser mdAT le générer à la première exécution.
- **`allowAdditionalProperties: true`** : à utiliser quand le résultat JSON contient des champs supplémentaires non pertinents (ex. : objets exception avec `StackTrace`).
- **`[DoNotParallelize]`** : obligatoire si le test modifie un état global (`MdatConfig.AddYamlTypeConverter` dans `[ClassInitialize]`).
- **Couverture de code** : dans `.runsettings`, utiliser `<Include>MDAT</Include>` pour cibler uniquement la librairie et exclure le projet de test.
- **Valeur null** : écrire `data: null` (pas `data:` vide) pour comparer explicitement à null.
- **Résultat sérialisé JSON** : `Verify.Assert` sérialise toujours le résultat avec `Newtonsoft.Json`. Les noms de propriétés sont sensibles à la casse dans `jsonPath`.
