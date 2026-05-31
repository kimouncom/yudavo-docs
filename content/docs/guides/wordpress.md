---
title: Utiliser le plugin WordPress
weight: 20
---

# Utiliser le plugin WordPress Yudavo

Le plugin WordPress officiel integre le widget public Yudavo dans WordPress sans faire transiter les reservations par WordPress lui-meme.

En pratique, le plugin :

- rend le conteneur du widget ;
- charge le script public du widget une seule fois par page ;
- transmet des valeurs publiques via des attributs `data-*` ;
- laisse le widget appeler directement l'API Yudavo.

Le plugin ne stocke pas les reservations et ne doit jamais contenir de secrets backend.

## Compatibilite

- WordPress `6.6+`
- PHP `8.1+`

## Ce qu'il vous faut avant installation

- l'URL de base de votre API Yudavo, par exemple `https://api.example.com` ;
- le slug de votre organisation ;
- une publishable key en production ;
- un site WordPress dont le domaine est autorise cote Yudavo.

## Installation

1. Copiez le plugin dans `wp-content/plugins/`.
2. Activez-le dans `Extensions`.
3. Ouvrez `Reglages > Yudavo`.
4. Renseignez les valeurs publiques du widget.
5. Ajoutez le shortcode dans une page ou un article.

## Reglages WordPress

Le plugin expose des reglages publics uniquement. Les champs reels supportes par le code actuel sont :

- `API base URL`
- `Organisation slug`
- `Publishable key`
- `Locale`
- `Appointment title`
- `Appointment duration`
- `Timezone`
- `Accent color`
- `Theme`
- `Layout`
- `Density`
- `Variant`
- `Brand name`
- `Brand logo URL`
- `Intro text`
- `Show summary`

Minimum requis pour commencer :

- `API base URL`
- `Organisation slug`

En production, ajoutez aussi `Publishable key`.

Important : le plugin derive automatiquement l'URL du script widget a partir de `API base URL`. Si votre API est `https://api.example.com`, le plugin charge :

```text
https://api.example.com/randevou/randevou-widget.js
```

## Utilisation du shortcode

Shortcode minimal :

```text
[yudavo]
```

Exemple avec surcharges locales :

```text
[yudavo
  appointment_title="Rendez-vous de découverte"
  appointment_duration="45 min"
  accent_color="#0f6fff"
  layout="compact"
  variant="compact-steps"
  theme="light"
]
```

Le shortcode genere un conteneur HTML avec les attributs attendus par le widget, puis laisse le script public s'auto-monter.

## Attributs supportes par le shortcode

Le plugin accepte les attributs publics suivants :

- `api_base_url`
- `tenant_slug`
- `publishable_key`
- `locale`
- `appointment_title`
- `appointment_duration`
- `timezone`
- `accent_color`
- `theme`
- `layout`
- `density`
- `variant`
- `brand_name`
- `brand_logo`
- `intro_text`
- `show_summary`

Priorite des valeurs :

1. les reglages globaux du plugin servent de base ;
2. les attributs du shortcode surchargent uniquement l'instance concernee.

## Plusieurs widgets sur une meme page

Le plugin supporte plusieurs shortcodes `[yudavo]` sur une meme page.

Comportement actuel :

- les assets du widget sont charges une seule fois ;
- chaque shortcode rend son propre conteneur ;
- chaque instance reste isolee.

## Ce qu'il ne faut pas mettre dans WordPress

Ne saisissez jamais dans WordPress :

- des credentials Google Calendar ;
- des credentials CalDAV ou Zimbra ;
- des cles SMTP ou Brevo ;
- des tokens admin ;
- des tokens internes d'automatisation ;
- des secrets d'infrastructure.

Le plugin est volontairement limite aux valeurs publiques exposees au navigateur.

## Depannage rapide

### Le widget ne s'affiche pas

Verifiez :

- que le shortcode `[yudavo]` est present dans le contenu rendu ;
- que `API base URL` et `Organisation slug` sont bien renseignes ;
- que le script `.../randevou/randevou-widget.js` est accessible ;
- que le theme ou un constructeur de page ne supprime pas le shortcode cote rendu.

### Le widget s'affiche mais les disponibilites ne chargent pas

Verifiez :

- la validite de `publishable_key` en production ;
- le slug d'organisation ;
- l'autorisation du domaine WordPress cote Yudavo ;
- la configuration CORS du backend Yudavo.

### Message admin de configuration incomplete

Le plugin affiche un avertissement aux administrateurs si `API base URL` ou `Organisation slug` manque dans `Reglages > Yudavo`.

## Recommandation de mise en production

Pour un site WordPress en production :

1. creez une publishable key dediee au site ;
2. restreignez-la au domaine final si vous utilisez une allowlist ;
3. configurez `API base URL`, `Organisation slug` et `Publishable key` dans le plugin ;
4. testez la page publique depuis le domaine final, pas seulement depuis l'administration WordPress.

Si vous avez besoin du contrat complet des endpoints appeles par le widget, utilisez la [Reference OpenAPI](/reference/).
