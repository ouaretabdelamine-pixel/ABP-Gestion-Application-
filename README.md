# ABP SCHOOL — Logiciel de gestion

Logiciel de bureau pour la gestion quotidienne de l'école de formation
**ABP SCHOOL** : étudiants, formations, paiements, présences, rapports et reçus.

## Technologies

| Domaine              | Choix                                    |
|-----------------------|-------------------------------------------|
| Langage               | Java 21                                   |
| Interface graphique   | JavaFX 21                                 |
| Base de données       | MySQL 8 (compatible MariaDB 10.4+)        |
| Build                 | Maven                                     |
| Accès base de données | JDBC (PreparedStatement exclusivement)    |
| Rapports Word         | Apache POI                                |
| Reçus / rapports PDF  | Apache PDFBox                             |

## Prérequis

- JDK 21
- MySQL Server 8 (ou MariaDB 10.4+) installé et démarré
- Maven 3.9+
- phpMyAdmin (facultatif — pratique pour consulter la base manuellement,
  mais **pas indispensable** : l'application crée et initialise la base
  de données toute seule au premier démarrage)

**Guide d'installation détaillé (liens, étapes, configuration) : voir la
section "Installation des outils de développement" plus bas.**

## Démarrage rapide

```bash
mvn clean compile          # vérifie que le projet compile
mvn javafx:run             # lance l'application (interface disponible à partir du Module 2)
```

Au tout premier lancement :

1. Un fichier de configuration est créé automatiquement à l'emplacement
   `<dossier utilisateur>/.abpschool/config.properties`. Par défaut, il pointe
   vers `localhost:3306`, utilisateur `root`, sans mot de passe. Modifiez ce
   fichier si votre configuration MySQL est différente — **aucune
   recompilation n'est nécessaire**.
2. La base de données `abp_school` et toutes ses tables sont créées et
   pré-remplies automatiquement (formations, paramètres par défaut).
3. Les journaux applicatifs (utiles en cas de problème) sont écrits dans
   `<dossier utilisateur>/.abpschool/logs/`.

## Structure du projet

```
abp-school/
├── pom.xml
├── README.md
├── src/main/java/com/abpschool/
│   ├── App.java                 # point d'entrée
│   ├── config/                  # configuration, connexion base de données, logs
│   ├── model/                   # entités métier (Etudiant, Formation, Paiement...)
│   │   └── enums/
│   ├── dao/                     # accès aux données (DAO / Repository)
│   │   └── impl/
│   ├── exception/                # exceptions applicatives
│   └── util/                    # validations et utilitaires communs
└── src/main/resources/
    ├── database/schema.sql      # script SQL complet (exécuté automatiquement)
    └── images/                  # logo et icône de l'application
```

## État d'avancement

- [x] **Module 1** — Fondations techniques (projet Maven, base de données,
      configuration, journalisation, modèles, DAO Étudiants/Formations)
- [x] **Module 2** — Interface principale et tableau de bord (navigation,
      thème visuel, statistiques dynamiques, graphiques, bouton actualiser)
- [x] **Module 3** — Gestion complète des étudiants et des formations
      (ajout, modification, suppression, recherche instantanée)
- [x] **Module 4** — Paiements (historique, écran dédié) et reçus PDF
      (génération automatique, proposition d'impression, réimpression)
- [x] **Module 5** — Rapport Word automatique (Apache POI) : un fichier par
      jour, créé ou complété à chaque inscription, sans intervention manuelle
- [ ] Module 6 — Présences et recherche avancée
- [ ] Module 7 — Rapports, paramètres, sauvegarde/restauration
- [ ] Module 8 — Finitions, packaging exécutable, tests

## Décisions techniques notables

- **Pas de pool de connexions (HikariCP)** : application mono-utilisateur,
  une connexion dédiée par opération est plus simple et tout aussi efficace.
- **Montant payé / restant jamais stocké** : toujours recalculé à partir de
  l'historique réel des paiements (vue SQL `v_etudiants_solde`), pour
  qu'aucune désynchronisation ne soit possible.
- **Suppression logique** des étudiants (champ `supprime`) : préserve
  l'historique financier même après une "suppression".
- **PDFBox plutôt qu'iText** pour les PDF : licence Apache 2.0, sans
  contrainte pour un usage commercial.
- **`java.util.logging` plutôt que SLF4J/Logback** : journalisation complète
  avec rotation de fichiers, sans dépendance externe supplémentaire.
- **Configuration externalisée** dans le dossier utilisateur : les
  identifiants MySQL sont modifiables sans recompiler l'application.
- **Initialisation automatique de la base de données** : aucune manipulation
  dans phpMyAdmin n'est nécessaire pour démarrer.
- **Icônes vectorielles maison** (formes JavaFX, pas de police d'icônes ni
  d'images externes) : rendu net à toute taille, colorable par CSS, zéro
  dépendance supplémentaire.
- **Navigation par écrans "à venir"** plutôt que boutons désactivés : chaque
  section du menu est cliquable dès maintenant, avec un message clair sur le
  module qui l'implémentera, pour une navigation agréable dès le Module 2.
- **Couche Service** (`FormationService`, `EtudiantService`, `DashboardService`) :
  le contrôleur JavaFX n'exécute jamais de SQL directement. Chaîne complète :
  MySQL → DAO → Service → Controller → JavaFX. `DashboardService` synchronise
  automatiquement le statut Actif/Terminé (selon la date de fin) avant de
  calculer les statistiques.
- **Aucune valeur codée en dur** : toutes les statistiques du tableau de bord
  proviennent de requêtes SQL réelles, recalculées à chaque ouverture de
  l'écran et via le bouton "Actualiser".
- **Format de date français forcé** (`SelecteurDateUtils`) : JavaFX utilise
  par défaut le format court de la locale système (souvent MM/jj/aaaa), ce
  qui pouvait faire lire "8 octobre" comme "10 août". Corrigé pour jj/mm/aaaa
  partout, saisie comme affichage.
- **Formations jamais supprimées, seulement désactivées** : préserve
  l'historique des étudiants déjà inscrits à une formation retirée du
  catalogue actif.
- **Boîtes de dialogue traduites explicitement** : JavaFX affiche "Cancel"
  par défaut selon la locale système ; les boutons de confirmation sont
  désormais toujours en français.
- **Espace normale dans les montants** (`FormatUtils`, utilisé partout) :
  Java utilise par défaut une espace fine insécable comme séparateur de
  milliers en locale française, invisible à l'écran (JavaFX) mais rendue
  "?" dans les PDF (police standard non-Unicode). Centralisé pour un rendu
  identique et correct partout.
- **Chemins de fichiers sans accent** (`Recus`, pas `Reçus`) : contrairement
  au contenu texte, l'encodage des chemins dépend du système d'exploitation
  et peut faire échouer la création du dossier sur certaines configurations.
- **Paiements immuables** : ni modifiables ni supprimables une fois
  enregistrés (historique financier fiable) — conforme au cahier des charges.
- **Inscription + paiement initial atomiques** : si un montant est saisi à
  l'ajout d'un étudiant, l'étudiant et son premier paiement sont enregistrés
  dans une seule transaction SQL (l'un ne peut pas réussir sans l'autre).
- **Rapport Word non bloquant** (`catch (Throwable ...)` volontaire) : un
  incident lors de l'écriture du rapport journalier ne peut jamais faire
  échouer une inscription déjà valide en base.

## Note de transparence — Module 5

Contrairement aux modules précédents, je n'ai pas pu exécuter réellement le
générateur de rapport Word dans mon environnement de vérification : le seul
paquet Apache POI disponible hors ligne (`libapache-poi-java`, empaqueté par
Ubuntu) est livré avec des schémas OOXML incomplets et échoue même sur la
création d'un document basique, indépendamment de mon code. Le code a donc
été écrit et relu avec un soin particulier en s'appuyant sur l'API XWPF
d'Apache POI (`XWPFDocument`, `XWPFTable`, `XWPFRun`...), stable et inchangée
depuis de nombreuses versions, mais n'a pas pu être testé de bout en bout
comme le reste du projet. Merci de vérifier le premier rapport généré et de
me signaler tout ajustement nécessaire — je corrigerai immédiatement.

## Installation des outils de développement

Cette section liste tout ce qu'il faut installer sur le poste de
développement avant de pouvoir compiler et lancer l'application (le poste
final de l'école, une fois le logiciel packagé en exécutable, n'aura pas
besoin de tout ceci — voir Module 8).

### 1. JDK 21

- Téléchargement : https://adoptium.net/temurin/releases/?version=21
  (choisir "JDK", version 21 - LTS, votre système)
- Windows : lancer le `.msi`, laisser cochées les options "Set JAVA_HOME
  variable" et "Add to PATH"
- macOS : lancer le `.pkg`
- Linux (Debian/Ubuntu) : `sudo apt install openjdk-21-jdk`
- Vérification : `java -version` et `javac -version` doivent afficher `21.x`

### 2. Apache Maven

- Téléchargement : https://maven.apache.org/download.cgi (archive
  "Binary zip archive")
- Windows : extraire le zip (ex : `C:\Program Files\Apache\maven`), puis
  ajouter `<dossier>\bin` à la variable d'environnement `PATH`
  (Panneau de configuration → Variables d'environnement)
- macOS : `brew install maven` (ou extraction manuelle + `PATH`)
- Linux : `sudo apt install maven`
- Vérification : `mvn -version` (doit détecter le JDK 21 installé à l'étape 1)
- JavaFX n'a **pas** besoin d'être installé séparément : Maven le télécharge
  automatiquement comme dépendance du projet.

### 3. MySQL Server 8 (ou MariaDB, testée et compatible)

Option recommandée pour un poste de développement : **XAMPP**, qui installe
MariaDB et phpMyAdmin ensemble en une seule fois.
- Téléchargement : https://www.apachefriends.org/download.html
- Installer, cocher au minimum le composant "MySQL/MariaDB" (Tomcat, Perl,
  FileZilla, Mercury Mail sont inutiles pour ce projet, à décocher si besoin)
- Démarrer via le "XAMPP Control Panel" → bouton *Start* sur la ligne MySQL
- Par défaut, le mot de passe root est vide : aucune modification du fichier
  de configuration n'est nécessaire.

Alternative : MySQL Server autonome via l'installeur officiel
https://dev.mysql.com/downloads/installer/ (choisir "Custom" → MySQL Server
uniquement). Un mot de passe root est demandé pendant l'installation :
notez-le, il faudra le reporter dans `config.properties` (voir ci-dessous).

### 4. phpMyAdmin (facultatif)

Déjà inclus si XAMPP a été choisi à l'étape 3 (accessible sur
`http://localhost/phpmyadmin` une fois Apache et MySQL démarrés dans le
Control Panel). Non nécessaire au fonctionnement de l'application, qui gère
seule la création de ses tables.

### Configurer la connexion à la base de données

1. Lancer l'application une première fois (`mvn javafx:run`) — une erreur de
   connexion est normale à ce stade si les identifiants ne correspondent pas
   encore, cela crée simplement le fichier de configuration.
2. Éditer le fichier créé :
   - Windows : `C:\Users\<VotreNom>\.abpschool\config.properties`
   - macOS/Linux : `~/.abpschool/config.properties`
3. Ajuster si nécessaire :
   ```properties
   db.host=localhost
   db.port=3306
   db.name=abp_school
   db.user=root
   db.password=votre_mot_de_passe
   ```
4. Relancer `mvn javafx:run`.
