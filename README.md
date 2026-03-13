# UIWrapper — Documentation

> Wrapper PySide6 pensé pour construire des interfaces de bureau en Python de manière lisible, chainable, et sans répétition.

---
## Installation
```bash
pip install uiwrapper
```

## Table des matières

1. [Démarrage rapide](#démarrage-rapide)
2. [UIWrapper | La fenêtre principale](#uiwrapper)
3. [Section | Les zones de contenu](#section)
4. [Widgets de base](#widgets-de-base)
5. [Formulaires](#formulaires)
6. [Mise en page](#mise-en-page)
7. [Positionnement](#positionnement)
8. [Sidebars](#sidebars)
9. [Onglets](#onglets)
10. [Scroll](#scroll)
11. [Tables](#tables)
12. [Références nommées](#références-nommées)
13. [Styling et thèmes](#styling-et-thèmes)
14. [Notifications](#notifications)
15. [Fenêtres secondaires](#fenêtres-secondaires)
16. [Inspector](#inspector)
17. [Raccourcis et bonnes pratiques](#raccourcis-et-bonnes-pratiques)

---

## Démarrage rapide

```python
from ui.ui_tools import UIWrapper

def main():
    ui = UIWrapper(title="Mon App", width=800, height=600, padding=10)
    ui.load_style("style/style.qss")

    ui.sidebar("left", size=180, align="top") \
        .add_label("Navigation") \
        .add_spacing(10) \
        .add_button("Accueil", lambda: print("Accueil")) \
        .add_button("Paramètres", lambda: print("Paramètres"))

    ui.add_label("Bienvenue !") \
      .add_button("Cliquez ici", lambda: ui.notify("Bonjour !", "success"))

    ui.run()

if __name__ == "__main__":
    main()
```

---

## UIWrapper

`UIWrapper` est le point d'entrée de toute interface. Il crée la fenêtre, gère les sidebars, le style et le cycle de vie de l'application.

### Création

```python
ui = UIWrapper(title="Mon App", width=800, height=600, padding=10)
```

| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| `title` | `str` | `""` | Titre affiché dans la barre de fenêtre |
| `width` | `int` | `800` | Largeur initiale en pixels |
| `height` | `int` | `600` | Hauteur initiale en pixels |
| `padding` | `int` | `0` | Marge intérieure globale en pixels |

---

### Gestion de la fenêtre

#### `ui.run()`
Lance l'application et affiche la fenêtre. À appeler en dernier, après avoir construit toute l'interface.

```python
ui.run()
```

---

#### `ui.center()`
Centre la fenêtre sur l'écran. À appeler avant `run()`.

```python
ui.center()
ui.run()
```

---

#### `ui.set_resizable(resizable)`
Autorise ou bloque le redimensionnement de la fenêtre par l'utilisateur.

```python
ui.set_resizable(False)   # fenêtre figée
ui.set_resizable(True)    # redimensionnable
```

---

#### `ui.set_min_size(width, height)` / `ui.set_max_size(width, height)`
Définit les dimensions minimales ou maximales de la fenêtre.

```python
ui.set_min_size(400, 300)
ui.set_max_size(1920, 1080)
```

---

#### `ui.set_opacity(value)`
Modifie la transparence de la fenêtre. `1.0` = opaque, `0.0` = invisible.

```python
ui.set_opacity(0.95)
```

---

#### `ui.set_icon(path)`
Définit l'icône affichée dans la barre de titre et la barre des tâches.

```python
ui.set_icon("assets/icon.png")
```

---

#### `ui.set_on_close(callback)`
Déclenche une fonction quand l'utilisateur ferme la fenêtre.

```python
ui.set_on_close(lambda: print("Au revoir !"))
```

---

#### `ui.load_style(path)`
Charge un fichier `.qss` (CSS Qt) et l'applique à toute la fenêtre.

```python
ui.load_style("style/style.qss")
```

---

#### `ui.open_window(title, width, height, padding)`
Ouvre une fenêtre secondaire indépendante. Retourne un nouvel `UIWrapper` avec toute la même API disponible.

```python
def ouvrir_aide():
    win = ui.open_window("Aide", width=400, height=300, padding=10)
    win.add_label("Bienvenue dans l'aide.")
    win.add_button("Fermer", lambda: win.window.close())

ui.add_button("Aide", ouvrir_aide)
```

---

#### `ui.notify(text, kind, duration)`
Affiche une notification flottante en bas de la fenêtre, qui disparaît automatiquement.

| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| `text` | `str` | — | Texte affiché |
| `kind` | `str` | `"info"` | Style : `"success"`, `"error"`, `"warning"`, `"info"` |
| `duration` | `int` | `3000` | Durée en millisecondes avant disparition |

```python
ui.notify("Sauvegardé !", "success")
ui.notify("Connexion perdue", "error", duration=5000)
```

---

## Section

Une `Section` représente une zone de l'interface dans laquelle on ajoute des éléments. Toutes les méthodes retournent `self` ce qui permet le **chaînage**.

`UIWrapper` expose directement les méthodes de `Section` pour le contenu central — pas besoin d'y accéder explicitement.

---

## Widgets de base

### `add_label(text, stretch, name)`
Ajoute un texte non interactif.

```python
ui.add_label("Bienvenue !")
ui.add_label("Titre important", name="titre-principal")
```

---

### `add_button(text, callback, stretch, gap, name)`
Ajoute un bouton cliquable.

| Paramètre | Description |
|-----------|-------------|
| `text` | Texte du bouton |
| `callback` | Fonction appelée au clic |
| `stretch` | Proportion de l'espace occupé (voir Positionnement) |
| `gap` | Espace en pixels ajouté **après** le bouton |
| `name` | Nom pour y accéder plus tard via `get()` |

```python
ui.add_button("Valider", lambda: print("ok"))
ui.add_button("Supprimer", on_delete, gap=10, name="btn-supprimer")
```

---

### `add_separator()`
Ajoute une ligne de séparation visuelle. Horizontale dans une colonne, verticale dans une ligne.

```python
ui.add_label("Section A")
ui.add_separator()
ui.add_label("Section B")
```

---

## Formulaires

### `add_input(placeholder, callback, stretch, name)`
Ajoute un champ texte. Le `callback` est appelé à chaque frappe avec le texte courant.

```python
ui.add_input("Email", name="email")
ui.add_input("Rechercher...", callback=lambda texte: filtrer(texte))
```

Pour lire la valeur plus tard :
```python
valeur = ui.get("email").text()
```

---

### `add_dropdown(options, callback, stretch, name)`
Ajoute une liste déroulante. Le `callback` reçoit la valeur sélectionnée à chaque changement.

```python
ui.add_dropdown(["Français", "English", "Español"], name="langue")
ui.add_dropdown(["Option A", "Option B"], callback=lambda val: print(val))
```

---

### `add_checkbox(text, callback, checked, stretch, name)`
Ajoute une case à cocher. Le `callback` reçoit `True` ou `False`.

```python
ui.add_checkbox("Mode sombre", callback=lambda etat: toggle_theme(etat))
ui.add_checkbox("Accepter les CGU", checked=False, name="cgu")
```

---

## Mise en page

### `add_row(stretches, stretch, align, cross_align)`
Crée une **ligne** — les éléments ajoutés dedans s'alignent horizontalement.
Retourne la nouvelle `Section` enfant.

```python
row = ui.add_row()
row.add_button("Oui", lambda: print("oui"))
row.add_button("Non", lambda: print("non"))
```

Avec chaînage et `.end()` pour remonter au parent :
```python
ui.add_row() \
  .add_button("Oui", lambda: print("oui")) \
  .add_button("Non", lambda: print("non")) \
  .end() \
  .add_label("Suite du contenu principal")
```

---

### `add_column(stretches, stretch, align, cross_align)`
Crée une **colonne** — les éléments ajoutés dedans s'empilent verticalement.
Fonctionne exactement comme `add_row` mais dans l'axe vertical.

```python
ui.add_row([1, 2, 1]) \
  .add_column().add_label("Gauche").end() \
  .add_column().add_label("Centre (double)").end() \
  .add_column().add_label("Droite")
```

---

### `end()`
Remonte à la `Section` parente pour continuer le chaînage à un niveau supérieur.

```python
ui.add_column() \
  .add_label("Dans la colonne") \
  .add_button("Bouton", callback) \
  .end() \                          # ← remonte au contenu principal
  .add_label("Hors de la colonne")
```

---

### `clear()`
Supprime tous les éléments d'une section. Utile pour reconstruire dynamiquement une zone.

```python
section = ui.add_column(name="zone-dynamique")
# ... plus tard ...
section.clear()
section.add_label("Nouveau contenu")
```

---

## Positionnement

### Espacement fixe : `add_spacing(px)`
Ajoute un espace fixe **entre deux éléments**, en pixels. Ne consomme pas de stretch.

```python
ui.add_label("Titre")
ui.add_spacing(20)           # 20px d'espace fixe
ui.add_button("Bouton", cb)
```

---

### Espace élastique : `add_spacer(stretch)`
Ajoute un espace vide qui s'étire. Utile pour pousser des éléments vers un bord.

```python
# Bouton collé à droite
ui.add_row() \
  .add_spacer(1) \             # pousse tout vers la droite
  .add_button("OK", cb)

# Bouton centré
ui.add_row() \
  .add_spacer(1) \
  .add_button("Centré", cb) \
  .add_spacer(1)
```

---

### Proportions : `stretches=[...]`
Passe une liste de ratios à `add_row()` ou `add_column()`. Chaque valeur est consommée dans l'ordre par les éléments ajoutés.

```python
# 3 colonnes : 1/4 - 2/4 - 1/4
ui.add_row([1, 2, 1]) \
  .add_column().add_label("Gauche").end() \
  .add_column().add_label("Centre").end() \
  .add_column().add_label("Droite")
```

Le paramètre `stretch` par élément **prend la priorité** sur la liste si les deux sont fournis.

---

### Alignement : `align`
Positionne le groupe d'éléments sur l'axe principal de la section.

| Valeur | Effet sur une colonne | Effet sur une ligne |
|--------|----------------------|---------------------|
| `"top"` | Éléments en haut | — |
| `"bottom"` | Éléments en bas | — |
| `"left"` | — | Éléments à gauche |
| `"right"` | — | Éléments à droite |
| `"center"` | Éléments centrés | Éléments centrés |

```python
ui.add_column(align="center") \
  .add_button("Centré verticalement", cb)
```

---

### Alignement croisé : `cross_align`
Positionne les éléments sur l'axe **perpendiculaire**.

Sur une **colonne** (axe vertical) → `cross_align` gère l'horizontal : `"left"`, `"right"`, `"center"`

Sur une **ligne** (axe horizontal) → `cross_align` gère le vertical : `"top"`, `"bottom"`, `"center"`

```python
# Boutons centrés verticalement ET collés à gauche
ui.add_column(align="center", cross_align="left") \
  .add_button("A", cb) \
  .add_button("B", cb)
```

---

### Marges : `set_margins(top, right, bottom, left)`
Définit les marges intérieures d'une section en pixels.

```python
ui.add_column() \
  .set_margins(top=10, left=15, right=15, bottom=10) \
  .add_label("Avec marges")
```

---

### Espacement global : `set_spacing(px)`
Définit l'espacement entre **tous les éléments** d'une section.

```python
ui.sidebar("left") \
  .set_spacing(8) \            # 8px entre chaque élément
  .add_button("A", cb) \
  .add_button("B", cb)
```

---

## Sidebars

`ui.sidebar(side)` crée et retourne une `Section` positionnée sur un des quatre bords de la fenêtre. Elle n'est créée qu'au premier appel (lazy) — pas de coût si non utilisée.

```python
ui.sidebar(side, stretches, align, cross_align, size)
```

| Paramètre | Description |
|-----------|-------------|
| `side` | `"left"`, `"right"`, `"up"`, `"down"` |
| `stretches` | Liste de ratios pour les éléments |
| `align` | Alignement sur l'axe principal |
| `cross_align` | Alignement sur l'axe perpendiculaire |
| `size` | Largeur (left/right) ou hauteur (up/down) fixe en pixels |

```python
# Sidebar gauche avec boutons en haut
ui.sidebar("left", align="top", size=160) \
  .set_margins(top=10, left=8, right=8) \
  .set_spacing(6) \
  .add_label("Navigation") \
  .add_separator() \
  .add_button("Accueil", cb) \
  .add_button("Paramètres", cb)

# Barre du bas centrée horizontalement
ui.sidebar("down", align="center", cross_align="center", size=40) \
  .add_label("Version 1.0.0")
```

> **Note :** `cross_align` sur `"up"` / `"down"` accepte `"top"`, `"bottom"`, `"center"`.
> Sur `"left"` / `"right"` il accepte `"left"`, `"right"`, `"center"`.

---

## Onglets

### `add_tabs(tabs, stretch, name)`
Crée un widget à onglets. Retourne un dictionnaire `{nom_onglet: Section}`.

```python
pages = ui.add_tabs(["Accueil", "Paramètres", "À propos"])

pages["Accueil"] \
  .add_label("Bienvenue !") \
  .add_button("Action", cb)

pages["Paramètres"] \
  .add_input("Nom", name="nom") \
  .add_checkbox("Mode sombre")

pages["À propos"] \
  .add_label("Version 1.0.0")
```

Le paramètre `name` permet de cibler le widget en QSS via `#nom-des-onglets`.

---

## Scroll

### `add_scroll(stretch, horizontal)`
Crée une zone scrollable. Retourne la `Section` intérieure dans laquelle on ajoute du contenu normalement.

| Paramètre | Défaut | Description |
|-----------|--------|-------------|
| `horizontal` | `False` | Active le scroll horizontal si `True` |

```python
scroll = ui.add_scroll()
for i in range(50):
    scroll.add_label(f"Ligne {i}")
```

---

## Tables

### `add_table(columns, rows, stretch)`
Crée un tableau avec des colonnes nommées. Retourne le `QTableWidget` pour pouvoir y ajouter des lignes.

```python
table = ui.add_table(["Nom", "Age", "Ville"])
```

---

### `add_table_row(table, values)`
Ajoute une ligne de données à un tableau existant.

```python
table = ui.add_table(["Nom", "Age", "Ville"])
ui.add_table_row(table, ["Alice", "30", "Paris"])
ui.add_table_row(table, ["Bob",   "25", "Lyon"])
```

---

## Références nommées

### `name=` sur les widgets
La plupart des méthodes `add_*` acceptent un paramètre `name`. Cela enregistre le widget sous ce nom ET lui assigne un `objectName` Qt pour le ciblage QSS.

```python
ui.add_input("Email", name="email")
ui.add_button("Supprimer", cb, name="btn-suppr")
```

---

### `section.get(name)`
Récupère un widget enregistré par son nom pour lire ou modifier sa valeur dynamiquement.

```python
# Lire la valeur d'un input
valeur = ui.get("email").text()

# Modifier un label
ui.get("titre").setText("Nouveau titre")

# Changer l'état d'une checkbox
ui.get("mode-sombre").setChecked(True)
```

---

## Styling et thèmes

### Fichier QSS : `ui.load_style(path)`
La méthode recommandée. Charge un fichier `.qss` (syntaxe CSS) et l'applique globalement.

```python
ui.load_style("style/style.qss")
```

Dans le fichier QSS, on peut cibler :
- **Par type** : `QPushButton`, `QLabel`, `QLineEdit`...
- **Par objectName** : `#sidebar-left`, `#email`, `#btn-suppr`
- **Par hiérarchie** : `QWidget#sidebar-left QPushButton`
- **Par état** : `:hover`, `:pressed`, `:focus`, `:checked`, `:disabled`

```css
/* Tous les boutons */
QPushButton {
    background-color: #3a3a3a;
    border-radius: 6px;
    padding: 5px 12px;
}

/* Un bouton spécifique */
#btn-suppr {
    background-color: #8b0000;
    color: white;
}

/* La sidebar gauche */
QWidget#sidebar-left {
    background-color: #161616;
    border-right: 1px solid #333;
}
```

---

### Style local : `section.set_style(css)`
Applique du CSS Qt uniquement sur le widget conteneur d'une section donnée.

```python
ui.sidebar("left") \
  .set_style("background-color: #161616; border-right: 1px solid #333;")
```

---

## Notifications

### `ui.notify(text, kind, duration)`
Affiche un bandeau coloré flottant en bas de la fenêtre, qui disparaît automatiquement.

| `kind` | Couleur |
|--------|---------|
| `"info"` | Bleu foncé |
| `"success"` | Vert |
| `"warning"` | Orange foncé |
| `"error"` | Rouge foncé |

```python
ui.notify("Fichier sauvegardé", "success")
ui.notify("Champ requis manquant", "warning", duration=4000)
ui.notify("Connexion perdue", "error", duration=0)   # ne disparaît pas
```

> `notify` s'appelle toujours sur `ui`, pas sur une section.

---

## Fenêtres secondaires

### `ui.open_window(title, width, height, padding)`
Ouvre une nouvelle fenêtre indépendante. Retourne un `UIWrapper` complet avec toute la même API.

```python
def ouvrir_parametres():
    win = ui.open_window("Paramètres", width=350, height=250, padding=10)
    win.add_label("Préférences") \
       .add_checkbox("Notifications") \
       .add_button("Fermer", lambda: win.window.close())

ui.add_button("Paramètres", ouvrir_parametres)
```

---

## Inspector

L'inspector analyse l'arbre de widgets **sans rien modifier**. Il fonctionne sur n'importe quel `QWidget` Qt.

```python
from ui.ui_inspector import print_tree, save_tree, generate_qss
```

---

### `print_tree(root)`
Affiche l'arborescence complète de l'interface dans le terminal.

```python
print_tree(ui.window)
```

Exemple de sortie :
```
QMainWindow
└─ QWidget  [central]
   └─ QWidget  [middle]
      ├─ QWidget  [sidebar-left]
      │  ├─ QLabel  'Navigation'
      │  └─ QPushButton  'Accueil'  [#btn-accueil]
      └─ QWidget  [content]
         └─ QLabel  'Bienvenue !'
```

---

### `save_tree(root, path)`
Sauvegarde la même arborescence dans un fichier texte.

```python
save_tree(ui.window, "ui_tree.txt")
```

---

### `generate_qss(root, path)`
**Génère automatiquement un fichier `.qss` pré-rempli** à partir de l'arbre. Tous les types de widgets trouvés et tous les `objectName` sont listés avec leurs propriétés commentées — il n'y a plus qu'à décommenter et remplir les valeurs.

```python
generate_qss(ui.window, "style/style.qss")
```

Le fichier généré ressemble à :

```css
/* ================================================================ */
/* QSS généré automatiquement par ui_inspector                      */
/* ================================================================ */

/* ── Styles globaux par type ── */

QPushButton {
    /* background-color: ;
    color: ;
    border-radius: px; */
}

/* ── Styles spécifiques par nom ── */

#sidebar-left {
    /* background-color: ; */
}

#btn-accueil {
    /* background-color: ; */
    /* color: ; */
}
```

---

## Raccourcis et bonnes pratiques

**Construire l'interface avant `run()`**
Tous les `add_*`, `sidebar()`, `load_style()` doivent être appelés avant `ui.run()`.

**Utiliser `name=` dès le départ**
Même si vous ne lisez pas encore la valeur, nommer les widgets dès la création permet de les cibler en QSS sans toucher au Python.

**Générer le QSS avec l'inspector**
Construire l'interface d'abord, appeler `generate_qss()`, puis customiser le fichier généré. C'est bien plus rapide que de deviner les sélecteurs.

**Ordre de configuration recommandé**
```python
ui = UIWrapper(...)      # 1. Créer
ui.load_style(...)       # 2. Style
ui.set_icon(...)         # 3. Icône
ui.set_resizable(...)    # 4. Contraintes
ui.center()              # 5. Position

# 6. Construire l'interface
ui.sidebar("left") ...
ui.add_label(...) ...

# 7. (optionnel) Inspector
generate_qss(ui.window)

ui.run()                 # 8. Lancer
```
