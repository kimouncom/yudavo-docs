---
title: Integrer le widget
weight: 10
---

# Integrer le widget Yudavo

Le widget Yudavo permet d'ajouter la reservation directement sur un site existant, sans exposer les secrets calendrier ou email dans le front-end.

Le code source utilise encore certains identifiants historiques `Randevou`, mais le contrat public du widget reste stable.

## Prerequis

Avant d'integrer le widget, il faut disposer de :

- l'URL de base de votre instance API Yudavo, par exemple `https://api.example.com` ;
- le `tenantSlug` de votre organisation, par exemple `cabinet-martin` ;
- une `publishableKey` en production ;
- un domaine autorise cote Yudavo si vous utilisez une allowlist d'origine ou de domaine.

Le widget appelle directement l'API publique :

- `GET /api/{slug}/slots`
- `POST /api/{slug}/book`

Il n'embarque pas de credentials Google Calendar, SMTP, Brevo, token admin ou token interne.

## Integration recommandee

Le mode recommande est l'integration declarative avec un conteneur HTML et le script public du widget.

```html
<div
  data-yudavo-widget
  data-api-base-url="https://api.example.com"
  data-tenant-slug="cabinet-martin"
  data-publishable-key="rdv_key_live_example"
  data-locale="fr"
  data-appointment-title="Rendez-vous de decouverte"
  data-appointment-duration="30 min"
  data-timezone="Europe/Paris"
  data-accent-color="#125c9d"
  data-theme="light"
  data-layout="split"
  data-density="comfortable"
  data-variant="classic"
  data-brand-name="Cabinet Martin"
  data-intro-text="Choisissez le moment qui vous convient."
  data-show-summary="true"
></div>
<script src="https://api.example.com/randevou/randevou-widget.js" defer></script>
```

Au chargement de la page, le script recherche automatiquement :

- les elements portant `data-yudavo-widget` ;
- l'element `#yudavo-booking`.

Le plugin WordPress officiel genere lui-meme ce markup et laisse le widget s'auto-monter.

## Parametres utiles

### Obligatoires

- `data-api-base-url`
- `data-tenant-slug`

### Fortement recommande en production

- `data-publishable-key`

En production, les routes publiques du widget exigent une publishable key tenant-scoped. Sans elle, l'API peut repondre avec `MISSING_PUBLISHABLE_KEY` ou `INVALID_PUBLISHABLE_KEY`.

### Options de presentation

- `data-locale`
- `data-appointment-title`
- `data-appointment-duration`
- `data-timezone`
- `data-accent-color`
- `data-theme`: `light`, `dark`, `auto`
- `data-layout`: `split`, `stacked`, `compact`
- `data-density`: `comfortable`, `compact`
- `data-variant`: `classic`, `compact-steps`
- `data-brand-name`
- `data-brand-logo`
- `data-intro-text`
- `data-show-summary`

Valeurs par defaut du runtime widget :

- `locale`: `fr`
- `mode`: `inline`
- `appointmentTitle`: `Rendez-vous`
- `appointmentDuration`: `30 min`
- `timezone`: `Europe/Paris`
- `accentColor`: `#125c9d`
- `theme`: `light`
- `layout`: `split`
- `variant`: `classic`
- `density`: `comfortable`
- `showSummary`: `true`

## Initialisation imperative

Si vous voulez controler explicitement le montage via JavaScript, le widget expose `window.Randevou.init()` et `window.Randevou.mount()`.

```html
<div id="yudavo-booking"></div>
<script src="https://api.example.com/randevou/randevou-widget.js" defer></script>
<script>
  window.addEventListener("DOMContentLoaded", function () {
    window.Randevou.init({
      target: "#yudavo-booking",
      apiBaseUrl: "https://api.example.com",
      tenantSlug: "cabinet-martin",
      publishableKey: "rdv_key_live_example",
      locale: "fr",
      layout: "split",
      variant: "compact-steps",
      brandName: "Cabinet Martin"
    });
  });
</script>
```

Ce mode est utile si le conteneur est injecte dynamiquement ou si vous devez piloter le rendu apres un evenement applicatif.

## Integration par iframe

Le widget peut aussi etre integre par iframe :

```html
<iframe
  src="https://static.example.com/randevou/widget.html?apiBaseUrl=https%3A%2F%2Fapi.example.com&tenantSlug=cabinet-martin&publishableKey=rdv_key_live_example&locale=fr&theme=light&layout=split&density=comfortable&brandName=Cabinet%20Martin"
  title="Prendre rendez-vous"
  loading="lazy"
></iframe>
```

Le mode `script` reste preferable pour une meilleure integration visuelle et une gestion plus simple du responsive.

## Recuperer la publishable key

La cle publique du widget se gere via l'API d'administration :

- `GET /api/admin/tenants/{slug}/publishable-keys`
- `POST /api/admin/tenants/{slug}/publishable-keys`
- `DELETE /api/admin/tenants/{slug}/publishable-keys/{keyId}`

La creation retourne la cle brute une seule fois. Conservez-la cote integrateur.

Exemple :

```bash
curl -X POST "https://api.example.com/api/admin/tenants/cabinet-martin/publishable-keys" \
  -H "Authorization: Bearer $ADMIN_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "domainAllowlist": ["https://www.example.com"]
  }'
```

## Points d'attention

### Origines autorisees

Le site qui embarque le widget doit etre autorise cote Yudavo. Selon votre configuration, un mauvais domaine peut produire une erreur `403` indiquant que la publishable key ne correspond pas au tenant ou a l'origine.

### CORS

Le backend Yudavo doit accepter l'origine finale du site qui integre le widget. C'est particulierement important pour :

- un site marketing sur un domaine principal ;
- un sous-domaine WordPress ;
- un environnement de preproduction distinct.

### Slots et reservation

Le widget ne calcule pas les disponibilites localement. Toute la logique metier passe par l'API Yudavo. Si un creneau devient indisponible entre l'affichage et la validation, l'API peut renvoyer `409 SLOT_UNAVAILABLE`.

## Depannage rapide

- `401 MISSING_PUBLISHABLE_KEY` : ajoutez `data-publishable-key` en production.
- `401 INVALID_PUBLISHABLE_KEY` : la cle ne correspond pas a une cle active.
- `403` sur `slots` ou `book` : verifiez le tenant, le domaine autorise et l'origine.
- Aucun rendu : verifiez que le script public charge bien `.../randevou/randevou-widget.js` et que le conteneur possede `data-api-base-url`.

Pour le contrat complet des endpoints et schemas, utilisez la [Reference OpenAPI](/reference/).
