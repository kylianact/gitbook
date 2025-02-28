---
description: >-
  Ici, nous aurons toutes les petites commandes qui simplifierons nos
  quotidiens.
---

# Commandes utiles

**Explication des commandes**

<mark style="color:green;">`sed '/^\s*[#;]/d' fichier.conf > fichier_clean.conf`</mark>

La commande  utilise <mark style="color:green;">`sed`</mark> pour traiter et nettoyer un fichier de configuration. Voici une explication détaillée :

* <mark style="color:green;">**`sed`**</mark> : Stream Editor qui permet de filtrer et transformer du texte.
* <mark style="color:green;">**`/^\s*[#;]/d`**</mark> : Cette expression régulière recherche les lignes qui commencent par des espaces optionnels (<mark style="color:green;">`\s*`</mark>) suivis d'un caractère de commentaire (<mark style="color:green;">`#`</mark> ou <mark style="color:green;">`;`</mark>). Le `d` à la fin indique à <mark style="color:green;">`sed`</mark> de supprimer ces lignes.
* <mark style="color:green;">**`fichier.conf`**</mark> : Le fichier d'entrée contenant les configurations, potentiellement avec des commentaires.
* <mark style="color:green;">**`>`**</mark> : Redirige la sortie de la commande vers un fichier.
* <mark style="color:green;">**`fichier_clean.conf`**</mark> : Le fichier de sortie qui contiendra les configurations sans les lignes de commentaires.

Utiliser cette commande permet de simplifier le fichier de configuration en supprimant toutes les lignes de commentaires, rendant la lecture et l'analyse du fichier plus efficaces.
