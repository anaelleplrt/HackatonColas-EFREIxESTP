# Démarche technique du volet images

[Retour à la présentation de DataBeton](README.md)

```text
Photos et annotations
        ↓
Relecture des labels et retrait des images trop ambiguës
        ↓
Séparation entraînement / validation / test interne
        ↓
Fine-tuning d’EfficientNet-B0 avec PyTorch
        ↓
Classe prédite pour chaque photo et export CSV
```

## Préparer les données

Le jeu de départ contenait très peu de photos annotées comme « camion vide ». L’équipe a relu les images et corrigé des labels. Pour le challenge, la catégorie « camion vide » couvrait aussi certains cas où le contenu était invisible, par exemple une benne bâchée. Les images inutilisables ou trop ambiguës ont été retirées. Le découpage aléatoire est stratifié par classe.

Les images sont converties en RGB, redimensionnées à **224 × 224** et normalisées avec les statistiques d’ImageNet. Le jeu d’entraînement reçoit aussi des retournements horizontaux et des variations de luminosité et de contraste.

## Adapter et évaluer le modèle

**EfficientNet-B0** est initialisé avec des poids préentraînés sur ImageNet. L’équipe remplace la dernière couche par une couche à **sept sorties**, puis affine le réseau avec **AdamW** et une entropie croisée avec *label smoothing*. Les poids sont sélectionnés sur la validation, puis évalués sur le test interne.

L’accuracy mesure la proportion d’images bien classées. La précision et le rappel par classe éclairent les erreurs sur les camions vides et les matériaux visuellement proches. Pour le jeu témoin, l’inférence choisit la classe au score le plus élevé. L’export contient une ligne par photo, avec le nom du fichier et la classe prédite.

## Examiner le biais de site

Certaines classes étaient concentrées sur un même site. Un découpage aléatoire par image peut laisser les mêmes conditions de prise de vue dans l’entraînement et le test interne. Pour évaluer un nouvel usage, il faudrait mettre un site à l’écart et documenter les combinaisons matériau/site trop rares pour être évaluées.

[Notebook d’origine](notebooks/part_DataScience_DataBeton_markdown.ipynb) · [Prédictions exportées](predictions/predictions_final.csv)
