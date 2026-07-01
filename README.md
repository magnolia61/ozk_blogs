# OZK Blogs

Drupal 7 module die alle logica rond het content type **Article** (de "blog"
op onvergetelijk.nl) op één plek verzamelt, in plaats van verspreid over losse
Drupal Rules en een form-alter in het theme.

Onderdeel van de website van **Stichting Onvergetelijke Zomerkampen (OZK)**.

## Wat doet de module

| Hook | Functie | Gedrag |
|------|---------|--------|
| `hook_node_presave` | `ozk_blogs_datum_corrigeren()` | Vult `field_datum` met de aanmaakdatum van de node als het veld leeg is. |
| `hook_node_presave` | `ozk_blogs_titel_samenstellen()` | Stelt de (verborgen) node-titel samen als `"d/m kampcode: blognaam"` uit `field_datum` + `field_kamp_categorie` (description = korte kampcode) + `field_blogtitel`. Alleen als Blognaam is ingevuld. |
| `hook_node_insert` | `ozk_blogs_activiteit_loggen()` | Logt bij een gepubliceerde nieuwe blog een CiviCRM-activiteit (activity_type_id 149) op het contact van de auteur. |
| `hook_form_alter` | `ozk_blogs_form_article_node_form_alter()` | Verbergt het verplichte core titelveld op zowel het aanmaak- als bewerkformulier. |

## Achtergrond

Deze module vervangt drie Drupal Rules (na migratie **uitgeschakeld**, niet
verwijderd, als referentie/fallback):

- `rules_corrigeer_datum_blog` — "Corrigeer Datum blog"
- `rules_blog_aanpassen_titel` — "BLOG aanpassen titel"
- `rules_nieuwe_blog_online` — "Nieuwe Blog online"

En de eerdere tijdelijke `porto_sub_form_article_node_form_alter()` uit het
Porto_sub-theme.

De contrib-module `exclude_node_title` verbergt de titel wél bij bewerken
(`node/%/edit`) maar bewust niet bij aanmaken (`node/add/article`); deze module
dekt dat gat.

## URL-patroon

De blog-URL wordt door **Pathauto** opgebouwd (config, geen code):

```
blog/[node:field-datum:custom:Ymd]/[node:field-kamp-categorie:description]/[node:title]
```

## Afhankelijkheden

- CiviCRM (voor het activiteitenlog)
- `wachthond()` uit `nl.onvergetelijk.base` voor debug-logging

## Maintainer

Richard van Oosterhout — webteam@onvergetelijk.nl
