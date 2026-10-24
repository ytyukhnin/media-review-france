---
name: "media-review-france"
description: "Revue média France des 48 dernières heures (franceinfo, Le Monde, Le Figaro, BFM TV + autres) livrée en story Instagram avec camembert des thématiques."
---

# Revue média France – story Instagram

Objectif : analyser les titres de la presse française sur les 48 dernières heures, calculer la part de chaque thématique et livrer une image au format story Instagram. Répondre en français.

## 1. Période
Fenêtre = 48 h avant la demande (ex. sam. 12 h → lun. 12 h). Obtenir l'heure exacte avec l'outil d'heure courante. Si l'utilisateur précise une autre période, la suivre.

## 2. Collecte
Les sites des médias (franceinfo, Le Monde, Le Figaro, BFM TV) ne sont généralement pas accessibles directement : passer par Titrespresse.
- Pages par média : `titrespresse.com/source/39-francetv.fr` (franceinfo), `source/15-lemonde.fr`, `source/16-lefigaro.fr`, `source/66-bfmtv.com`, ainsi que les variantes `news/15-lemonde.fr`, `news/16-lefigaro.fr`, `news/66-bfmtv.com` (souvent plus fraîches) et la pagination `?pageindex=N`.
- Relevés tous médias : `alaune/recents?pageindex=N`, `top?h=24&pageindex=N`, `top?h=2`, et le bloc « Les derniers articles publiés » des pages article.
- N'utiliser que des URL apparues dans des résultats de recherche ou des pages déjà récupérées (ne pas en fabriquer). Si une page source n'est pas autorisée, la trouver d'abord via une recherche Titrespresse ou depuis une page Titrespresse déjà ouverte.
- Dans le prompt WebFetch, demander pour chaque article : l'identifiant numérique de l'URL Titrespresse | média | description neutre (12 mots max) | ancienneté affichée, ainsi que les URL de pagination.
- **Dater les pages** : beaucoup de pages profondes sont servies depuis un vieux cache (parfois plusieurs semaines). Les identifiants Titrespresse sont séquentiels (≈ 300/h la nuit, 500–650/h le jour) : caler 2–3 points de référence (métadonnée `published_time` d'une page article) et écarter toute page ou tout article hors fenêtre. « il y a 1 jour / 2 jours » suit le jour calendaire.
- Viser tous les titres des 4 médias sur la période, plus 5 à 8 relevés tous médias répartis dans le temps. Noter honnêtement la couverture réelle de chaque média.

## 3. Classement
- Paraphraser chaque titre (ne pas recopier les titres mot pour mot).
- 1 titre = 1 thème principal ; même média + même sujet = une seule ligne.
- Exclure : actualité étrangère sans angle français, programmes TV et podcasts, agendas et sorties, conseils pratiques, horoscopes, contenus sponsorisés, vie hyperlocale (associations, commerces, animations, clubs locaux), doublons.
- 8 thèmes (codes et couleurs fixes) :
  - VIO Violences urbaines et ordre public `#eb6834` (interdictions, couvre-feux, dispositifs de sécurité)
  - POL Réponse politique et débat public `#1baf7a`
  - REV Revendications `#008300` (mobilisations, syndicats, lycéens, pouvoir d'achat, carburants)
  - JUS Justice et faits divers `#d55181` (enquêtes, procès, attentats, accidents)
  - SOL Solidarité et rassemblements citoyens `#4a3aa7` (marches blanches, cagnottes)
  - ECO Économie et commerces `#e66767` (commerçants, entreprises, notation, finances)
  - VIG Vigilance météo et risques `#2a78d6`
  - AUT Autres sujets `#c3c2b7` (culture, sport, sciences, société)
- Ajouter un indicateur 0/1 : titre lié à l'événement dominant de la période.

## 4. Calcul
Écrire un script Python (`corpus.py`) contenant les lignes `(relevé, jour, média, sujet, thème, lien)`. Il vérifie l'absence de doublon (média, sujet), calcule les parts par la méthode du plus fort reste (total = 100 %) et produit `corpus.csv` et `stats.json` (total, nombre de médias, titres des 4 médias, part liée à l'événement, thèmes triés). Contrôle de robustesse : recalculer sur les 4 médias seuls.

## 5. Image (lire d'abord la skill dataviz)
Matplotlib, police Inter, PNG 1080×1920 converti en RGB, fond `#fcfcfb`.
- Zones de sécurité : rien au-dessus de 250 px ni sous 1670 px ; marges latérales de 72 px.
- En-tête : « JJ – JJ mois AAAA · 48 dernières heures » (30 px), « Revue média France » (78 px semibold), « Part de chaque thématique dans N titres de M médias » (31 px).
- Camembert (centre 540/≈730, rayon 280) partant de midi dans le sens horaire, « Autres » en dernier, bords blancs.
- Paires de couleurs à ne jamais placer côte à côte : VIO–JUS, VIO–REV, VIO–ECO, POL–JUS, POL–ECO, JUS–ECO. Chercher l'ordre valide le plus proche du tri par taille (force brute), puis valider avec `node validate_palette.js "<hex dans l'ordre>" --mode light` (seuls échecs acceptés : le gris de « Autres »).
- Libellés : % + nom court pour les parts ≥ 12 %, % seul pour 5–12 %, étiquette extérieure avec trait pour moins de 5 %. Encre noire ou blanche selon le contraste.
- Tableau sous le camembert : pastille couleur, thème, nombre de titres, part. Tri par taille, « Autres » en dernier.
- Une phrase de lecture : « X titres sur N (P %) portent sur [événement dominant]. »
- Pied de page : « Sources : franceinfo, Le Monde, Le Figaro, BFM TV » / « K titres de ces 4 médias + n autres, via Titrespresse » / « 1 titre = 1 thème · hors étranger, agendas et vie locale ». « Généré par l'IA » (32 px semibold) aligné à droite.
- Ne pas comparer deux thèmes entre eux.
- Vérifier dans le script : textes dans les marges, aucun chevauchement. Regarder l'image puis un aperçu en 390 px de large.

## 6. Livraison
Retourner le PNG sous le nom **`revue_media_story.png`** dans le répertoire courant, et l'afficher dans la réponse. La réponse contient :
- 3–4 lignes de résultats (thèmes de tête et leurs moteurs, part liée à l'événement, contrôle de robustesse) ;
- les limites de couverture par média ;
- une liste « Sources : » de liens Markdown vers les pages Titrespresse utilisées.