# Feedbacks

Une observation est consituée d'une description et optionnellement d'une ou plusieurs photos. Elle est créée par un utilisateur à un endroit donné et dans une catégorie (et donc dans une organisation) donnée.

## Création d'une observation

```
POST /feedbacks/issues
```

```json
{
    "geo":
        {
            "elevation":1,
            "point":
                {
                    "latitude":44.851343361295214,
                    "longitude":-0.5763262510299683
                }
        },
    "category":"b0d007d5-e6ad-4113-b2b5-d8a1858a2fb1",
    "description":"Mon feedback 5",
    "reporter":"6dbbd601-267f-46ea-be90-8c9742f7180b",
    "image":"data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mP8/5+hHgAHggJ/PchI7wAAAABJRU5ErkJggg=="
}
```

Ce endpoint se présente sous la forme /feedbacks/issue et non pas simplement /feedbacks, car à terme, il sera possible de créer différents types d'observation. Actuellement, seul le type "issue" est disponible.

Coordonnées géographiques du point : LIEN

image : LIEN

L'utilisateur peut ensuite ajouter d'autres images à son observation :

```
POST /feedbacks/ba45ab2c-1578-4be7-bb27-8ab7914f3785/images
```

paramètres :
```
    image : l'image à ajouter
```

## Modération et cycle de vie d'une observation

Après qu'un utilisateur a créé une nouvelle observation, celle ci possède le statut PENDING_REVIEW : en attente de modération. Elle devra être validée par un administrateur du domaine applicatif.

LIEN : statuts et changements de statuts

Un administrateur valide une observation avec le endpoint :

```
POST /feedbacks/{feedback}/state
```

```json
[{"op":"replace","path":"transition","value":"accept"}]
```

L'observation prend alors le statut ACCEPTED et un rapport est créé sur cette observation.

LIEN : rapports

Pour refuser une observation :

```json
[{"op":"replace","path":"transition","value":"refuse"}]
```

L'observation prend alors le statut REFUSED.

**Acceptation automatique d'une observation**

Un utilisateur qui est membre d'une organisation peut créer une nouvelle observation qui sera automatiquement acceptée sans passer par l'étape de modération. À condition que cette nouvelle observation soit effectuée sur une catégorie appartenant à l'organisation dont l'utilisateur est un membre.

Supposons que la requête suivante est exécutée par un utilisateur membre de l'organisation 84d36093-b8bc-47ad-bc8a-a043b3e301a9 et que la catégorie b0d007d5-e6ad-4113-b2b5-d8a1858a2fb1 appartient à cette organisation :


```
POST /feedbacks/issues
```

```json
{
    "geo":
        {
            "elevation":1,
            "point":
                {
                    "latitude":44.851343361295214,
                    "longitude":-0.5763262510299683
                }
        },
    "category":"b0d007d5-e6ad-4113-b2b5-d8a1858a2fb1",
    "description":"Mon feedback 5",
    "reporter":"6dbbd601-267f-46ea-be90-8c9742f7180b",
    "image":"data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mP8/5+hHgAHggJ/PchI7wAAAABJRU5ErkJggg==",
    "organization":"84d36093-b8bc-47ad-bc8a-a043b3e301a9"
}
```

créera une observation qui aura automatiquement le statut ACCEPTED.

Si le paramère "organization" n'avait pas été passé, alors cette observation aurait suivi le cycle normal et aurait reçu le statut PENDING_REVIEW.

**Résumé du cycle de vie d'une observation**

![Cycle de vie d'une observation](images/feedback_workflow.png "Cycle de vie d'une observation")

## Récupération des observations

Pour récupérer toutes les observations :

```
GET /feedbacks
```

Plusieurs critères permettent de filter cette liste.

**Par statut : paramètre state**

Par exemple, pour filtrer les observations en attente de validation, un administrateur effectuera la requête :

```
GET /feedbacks?state=PENDING_REVIEW
```

**Autour d'un point : paramètre geo_near**

Exemple :

```
GET /feedbacks?geo_near[radius]=1000&geo_near[geo_coordinates]=+44.8-0.5
```

retournera les observations situées dans un rayon de 1000 mètres autour du point de latitude +44.8 et de longitude 0.5.

**Dans un geo hash : paramètre geo_hash**

```
GET /feedbacks?geo_hash[]=ezz
```

retournera les observations comprises dans le geo hash ezz et dans les geo hash voisins de ezz.

LIEN : geo_hash

**Sur une période donnée : paramètres before et after**

Exemple :

```
GET /feedbacks?after=2017-01-10T00:00:00+05:00&before=2017-02-22T23:59:59+05:00
```

retournera les observations effectuées entre le 10/01/2017 et le 22/02/2017.

Les dates sont écrites au format [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html)

## Commentaires

Les utilisateurs de la communauté peuvent commenter une observation :

```
POST /feedbacks/{feedback}/comments
```

Paramètres :
```
    text : contenu du commentaire
```

Pour récupérer les commentaires d'une observation :

```
GET /feedbacks/{feedback}/comments
```

## Soutiens

Un utilisateur peut soutenir une contribution en effectuant la requête suivante, sans paramètres :

```
POST /feedbacks/{feedback}/contributions
```

Pour récupérer tous les soutiens effectués sur une observation :

```
GET /feedbacks/{feedback}/contributions
```