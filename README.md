# OZK Blogs

Drupal 7 module die alle logica en UX rond het content type **Article** (de
"blog" op onvergetelijk.nl) op één plek verzamelt, in plaats van verspreid over
losse Drupal Rules en losse veld-/thema-instellingen.

Onderdeel van de website van **Stichting Onvergetelijke Zomerkampen (OZK)**.
Doel: hoofdleiding/groepsleiding snel en zonder ruis een kampblog laten plaatsen.

## Wat doet de module (PHP-hooks)

| Hook / functie | Doel |
|---|---|
| `ozk_blogs_node_presave` → `ozk_blogs_datum_corrigeren` | Vult `field_datum` met de aanmaakdatum als het veld leeg is. |
| `ozk_blogs_node_presave` → `ozk_blogs_titel_samenstellen` | Stelt de (verborgen) node-titel samen: `d/m kampcode: blognaam` uit `field_datum` + `field_kamp_categorie` (description = korte kampcode) + `field_blogtitel`. Alleen als Blognaam gevuld is. |
| `ozk_blogs_node_presave` → `ozk_blogs_tag_afleiden` | Zet automatisch de kamptype-tag (kinderkamp/…/topkamp) o.b.v. de bovenste term van `field_kamp_categorie`; behoudt handmatig gekozen extra tags. |
| `ozk_blogs_node_insert` → `ozk_blogs_activiteit_loggen` | Logt bij een gepubliceerde nieuwe blog een CiviCRM-activiteit (activity_type_id 149) op het contact van de auteur. |
| `ozk_blogs_node_view` | Rendert de YouTube-vlog uit `field_vlog_url` als responsieve embed (`.embed-container` in een omkaderd blok, welkomvideo-stijl) boven de blogtekst; alleen op de volledige pagina. `ozk_blogs_youtube_id()` haalt het ID uit watch?v= / youtu.be / embed / shorts / kale ID. |
| `ozk_blogs_form_article_node_form_alter` | Verbergt titel- en tags-veld; koppelt de compacte-formulier-CSS; hangt de #after_builds op (datum, body, fotovelden); verbergt dubbele Documenten-titel. |
| `ozk_blogs_datum_after_build` | Wist de "E.g., <datum>"-format-hint van de date_popup-widget. |
| `ozk_blogs_body_after_build` | Verbergt de filter-guidelines/help; forceert het tekstformaat op **Filtered HTML** (kiezer verborgen via CSS, blijft in DOM voor de editor). |
| `ozk_blogs_field_widget_form_alter` | Vervangt het generieke media-label "Attach media" per fotoveld: "upload meer banner foto's" / "upload meer foto's". Draait ook bij AJAX-herbouw. |
| `ozk_blogs_photo_field_after_build` → `ozk_blogs_mark_photo_buttons` | Maakt van de "Verwijderen"-knop een rood **×** (rechtsboven) en van de "Bewerken"-modallink een blauw **✎** (linksboven). Werking blijft (AJAX op #name, ctools-modal-class blijft). |
| `ozk_blogs_admin_paths_alter` | Haalt `node/add/article` en article-edit uit de admin-paden zodat het formulier in het front-end-thema **Porto_sub** rendert i.p.v. Seven. |

## CSS (`ozk_blogs.form.css`)

Gescoped op `.ozk-blog-form` (klasse die de form_alter op de `<form>` zet).
Verzorgt: compacte veldafstanden, label-/legend-formaat, ruim Body-veld
(min-height), verbergen formaat-UI, foto's als **grid van kaartjes**, ronde
**×**/**✎**-iconen, upload-secties als kader met whitesmoke-achtergrond en
"Browse" als knop (OZK-magenta), grijze "Bijsnijden"-knop.

## Veld-/site-instellingen (in de DB, NIET in deze module-code)

Deze zijn bewust config i.p.v. code (zo beheert de site z'n veldconfiguratie):

- `field_blogtitel` — **verplicht**; voedt de auto-titel. Label "Blognaam".
- `body` — label **"Blogtekst"**, widget-weight 4 (direct onder Kamp), tekstformaat vast op Filtered HTML.
- `field_kamp_categorie` — voedt titel, URL én kamptype-tag.
- `field_tags` — niet verplicht, verborgen op het formulier (automatisch gevuld).
- `field_vlog_url` — YouTube-link (aangemaakt voor deze module); weergave via `ozk_blogs_node_view`.
- `field_documenten` — label "Documenten".
- `field_image` (Banner) / `field_extraimages` (Meer fotos) — manualcrop aan, `crop_info` uit, `thumbnail_facebook` uit de stijllijst; Banner cropt `ozk_header_style_3840`, Meer fotos `ozk_fullsize_3840_2160`.
- **Pathauto** article-patroon: `blog/[node:field-datum:custom:Ymd]/[node:field-kamp-categorie:description]/[node:title]`.
- Taxonomieterm 572 (jeugdkamp2) description: `jk2` (was `jk2\n` — kapotte URL/titel).

## Vervangt (uitgeschakeld, niet verwijderd — als referentie)

- Rules `rules_corrigeer_datum_blog`, `rules_blog_aanpassen_titel`, `rules_nieuwe_blog_online`.
- De tijdelijke `porto_sub_form_article_node_form_alter()` in het Porto_sub-theme.

## Valkuilen / gotchas

- **Thema-switch**: `hook_custom_theme()` werkt NIET voor node-formulieren omdat
  `node_admin_theme` het admin-thema forceert via admin-pad-detectie (die wint).
  Gebruik `hook_admin_paths_alter()`.
- **CSS-cache-lag**: op deze omgeving komen CSS-wijzigingen vaak één versie te
  laat aan (browser/cache). Server serveert wél direct het juiste bestand
  (geen aggregatie: `preprocess_css` uit). Test cache-vrij in incognito; forceer
  desnoods `_drupal_flush_css_js()`. Kritieke wijzigingen liever server-side
  (#access/after_build) dan puur CSS.
- **Media-bewerkpopup is context-loos**: de crop-stijlen daar komen uit de
  globale file-entity-instelling, niet per veld — daar is de crop-lijst dus niet
  per veld te beperken.
- **Alleen-gecropt tonen kan niet betrouwbaar**: `.manualcrop-preview-cropped`
  wordt pas door JS gevuld; daarom tonen we het (kleine) originele voorbeeld.

## Afhankelijkheden

- CiviCRM (voor het activiteitenlog)
- `wachthond()` uit `nl.onvergetelijk.base` voor debug-logging

## Maintainer

Richard van Oosterhout — webteam@onvergetelijk.nl
