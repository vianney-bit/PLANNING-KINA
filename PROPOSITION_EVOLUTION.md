# Proposition d'évolution — Planning KINA

## Objectif
Garder la logique actuelle (projets, besoins, affectation) en améliorant :
- la **saisie** (plus guidée, plus rapide),
- le **visuel** (structure plus claire),
- la **lecture planning** (filtres et vues métier).

Cette proposition respecte la hiérarchie métier existante d'affichage :
1. Véhicules
2. Internes
3. RG
4. Adjoint
5. ARA
6. Déco

---

## 1) Structure fonctionnelle cible (sans changer le cœur)

### A. Bouton “Ajouter un régisseur”
Créer un flux dédié pour enrichir la base moyens humains :
- champs : `nom`, `prenom`, `email`, `telephones[]`, `commentaire`
- enregistrement dans `means` avec :
  - `type = "human"`
  - `group = "HUMANS_RG"` (par défaut pour régisseur)
  - `active = true`

> Variante utile : permettre de choisir le groupe (RG / ADJOINT / ARA / DECO / INTERNES) via une liste paramétrable.

### B. Bouton “Créer un projet”
Conserver les champs actuels :
- nom client
- nom projet
- email contact client

Puis ouvrir une **fenêtre/modale “Ajouter les besoins du projet”** :
- type besoin : véhicule / parking / humain
- sous-type besoin (issu d'un catalogue paramétrable)
- dates :
  - continues (début/fin)
  - discontinues (sélection multiple)
- quantité
- brut/jour (si humain)
- commentaire optionnel

Les besoins créés restent `open` (non affectés) et remontent en haut dans “Besoins à affecter”.

### C. Affectation des besoins
Conserver le principe actuel :
- clic sur un besoin non affecté/partiel
- ouverture modal
- choix des dates
- choix d'une ressource compatible
- transaction : update coverage + statut + création assignment

---

## 2) Modèle de données recommandé (compatible avec l'existant)

## Collection `means`
Conserver les documents existants et ajouter/normaliser ces champs :
- `type`: `human | vehicle | parking`
- `group`: ex. `HUMANS_RG`, `HUMANS_ADJOINT`, `VEHICLES_INTERNAL`, etc.
- `label`: nom affiché
- `firstName`, `lastName` (pour humains)
- `email`
- `phones`: tableau de numéros
- `comment`
- `active`: bool
- `availabilityMode` (optionnel): `all | current_month | current_plus_future`
- timestamps

## Collection `needs`
Déjà bonne base. Ajouter en option :
- `priority` (low/normal/high)
- `projectStartDate`, `projectEndDate` (dénormalisé pour filtres rapides)

## Collection `projects`
Conserver + enrichir optionnel :
- `status`: `active | archived`

## Collection `settings` (nouvelle)
Pour paramétrer “en amont” les types/intitulés besoins :
- `needCatalog`:
  - humains: RG, ADJOINT, ARA, DECO, INTERNES
  - véhicules: VAN_9, VUL, etc.
  - parking: PARKING
- ordre d'affichage planning

---

## 3) UX/UI proposée

## A. Barre d'actions en haut
- `+ Ajouter un moyen humain`
- `+ Créer un projet`
- `+ Ajouter un besoin` (optionnel, accès direct)

## B. Saisie guidée en 2 étapes
1. **Projet**
2. **Besoins du projet** (wizard / modal)

Bénéfice : moins d'erreurs, logique “je crée un projet puis ses besoins”.

## C. Carte “Besoins à affecter”
Conserver la zone actuelle, mais ajouter :
- filtres rapides : type, sous-type, période
- badge de priorité
- compteur de couverture (`x/y dates couvertes`)

## D. Vue planning par lignes métier
Conserver l'ordre demandé :
- Véhicules
- Internes
- RG
- Adjoint
- ARA
- Déco

Chaque ligne affiche uniquement les ressources selon le filtre de période sélectionné.

---

## 4) Filtres demandés pour ne pas surcharger les moyens humains

Ajouter un sélecteur global de visibilité :
- `Tous les moyens humains`
- `Mois en cours`
- `Mois en cours + projets à venir`

Règle de calcul suggérée :
- un moyen humain est visible s'il possède au moins une affectation dans la fenêtre choisie,
- sauf mode `Tous` qui montre l'ensemble des actifs.

Implémentation Firestore possible :
1. récupérer les `assignments` dans la période,
2. extraire les `resourceId` uniques,
3. charger les `means` correspondants,
4. afficher selon l'ordre des groupes métier.

---

## 5) Plan d'implémentation en étapes (itératif)

### Sprint 1 — fondations saisie
- modal `Ajouter un moyen humain`
- création humains dans `means`
- validation des champs (email/téléphone)

### Sprint 2 — création projet + besoins enchaînés
- modal `Créer projet`
- ouverture automatique modal `Ajouter besoins`
- utilisation d'un catalogue paramétré pour les sous-types

### Sprint 3 — planning filtrable
- ajout filtres de visibilité (Tous / Mois / Mois+Futur)
- regroupement et tri des humains par groupe métier

### Sprint 4 — qualité visuelle
- extraire CSS dans un fichier dédié
- composants réutilisables (cards, modals, boutons)
- états vides/chargement cohérents

---

## 6) Ce qui est déjà bien dans l'existant (à garder)
- logique `open/partial/filled`
- calcul de couverture par date
- affectation transactionnelle
- séparation projets / besoins / affectations

---

## 7) Décisions produit à valider
1. “Ajouter un régisseur” = uniquement RG, ou “Ajouter un humain” multi-groupes ?
2. Téléphone unique ou multiple par personne ?
3. Filtres planning : persistants (mémorisés) ou non ?
4. Besoins : autoriser la création sans dates (brouillon) ou non ?

---

## 8) Résultat attendu
Un outil qui reste simple comme aujourd'hui, mais :
- plus rapide en saisie,
- plus propre visuellement,
- mieux adapté au pilotage quotidien via filtres et hiérarchie métier.
