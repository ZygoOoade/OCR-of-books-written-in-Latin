## Problème

Le présent projet se propose de mettre en œuvre un processus automatisé de conversion de fichiers PDF dont le contenu est constitué d'images en fichiers PDF dont le contenu est constitué de texte exploitable.
<br>
En effet, les PDFs constitués par scans présentent plusieurs inconvénients :
* Le texte ne peut pas être sélectionné car chacune de ses pages est considérée au format image.
* Le fichier PDF pèse lourd, où son poids varie notamment en fonction de la résolution des images choisie lors du scan.

## Bounding Box Cropping

En premier lieu nous proposons une solution de redimensionnement par zone d'intérêt (bounding box croping) qui est très performante notamment pour enlever un watermark présent sur un PDF.

Après avoir installé les packages :

```
!pip install Pillow pdf2image
!apt-get install -y poppler-utils
```

Nous utilisons une boucle `for` pour transformer chaque page du PDF en une image au format `jpg`. Toutes les images obtenues dans un dossier sont stockées dans un dossier.

```
for pdf_file in os.listdir(pdf_folder):
    if pdf_file.endswith(".pdf"):
        pdf_path = os.path.join(pdf_folder, pdf_file)

        try:
            # Convertir le PDF en images
            pages = convert_from_path(pdf_path)

            # Sauvegarder chaque page comme un fichier JPG
            for i, page in enumerate(pages):
                jpg_file = f"{os.path.splitext(pdf_file)[0]}_page_{i+1}.jpg"
                jpg_path = os.path.join(output_folder, jpg_file)
                page.save(jpg_path, "JPEG")

            print(f"Conversion terminée pour {pdf_file}")
        except PDFPageCountError:
            print(f"Erreur lors de la conversion de {pdf_file}: PDF non valide ou corrompu")
        except Exception as e:
            print(f"Erreur inattendue lors de la conversion de {pdf_file}: {str(e)}")
```

Comme Google Colab ne prend pas en charge de manière native les applications basées sur une interface graphique, nous utilisons le package suivant :

```
!pip install jupyter-dash dash plotly pillow
```


L'utilisateur sélectionne une zone rectangulaire d'intérêt (bounding box) contenant seulement le texte d'intérêt sur toutes les pages du PDF uploadé.
<br>
Puis toutes les images sont rognées avec les proportions de la zone rectangulaire d'intérêt définie par l'utilisateur à l'étape précédente.



Les données sous forme d'image ne sont pas dans les dossiers github mais disponibles aux liens suivants :

https://drive.google.com/drive/folders/1-0ba_-gKzAtB29r7HHTKCUR-F__8BtmZ?usp=sharing
https://www.e-rara.ch/zut/content/titleinfo/4837753?query=clavius
https://www.e-rara.ch/zut/content/titleinfo/1182315


**Remarques :**
* La sélection d'une zone rectangulaire d'intérêt sert à éliminer les éléments qu'il pourrait y avoir autour (comme un titre dans l'en tête ou un numéro de page), qui seraient captés par l'OCR et ensuite inséré avec le texte lui-même. Nous pensons surtout au texte dans les marges : l'OCR ne détecte pas qu'il est dans la marge et le met donc au milieu d'une phrase du corps du texte, ce qui ferait dysfonctionner notre processus.
* Il faut que la zone rectangulaire minimale contenant le texte d'intérêt ne bouge pas d'une page à l'autre du PDF. Autrement dit, que la taille du pied de page, des marges et de l'en tête restent fixe. Cela n'est souvent pas le cas, par exemple, avec des textes récents où il y a des notes de bas de page. Il nous semble que dans ce cas, il faudrait selectionner la zone rectangulaire pour chaque page du PDF.
* Nous partirons du principe, en général, que les PDF uploadé ne contiennnent que du texte, et donc pas d'image, pas de tableaux, de schémas ou autre.

**Problèmes :**
* Il serait intéressant de savoir si l'OCR fonctionne mieux avec des images de petit format qu'avec des images de grand format. Si tel est le cas, il nous semble qu'il faudrait implémenter - en plus - un découpage des images extraites du PDF.
* Il serait intéressant de savoir si l'OCR a de meilleures perfomances dans des conditions spécifiques. Il est tout à fait possible, en effet, d'employer sur les fichiers `png` des opérations spécifiques comme le fait d'enlever l'arrière plan, d'augmenter la résolution ou bien de faire varier la luminosité et/ou le contraste. Pour ce qui est de la résolution de l'image, il semble probable que cela augmente les performances de l'OCR et augmente aussi son coût/son temps de fonctionnement.

**Pistes**
* Il serait très intéressant de tester et comparer la performance de plusieurs OCR. Pour avoir de meilleures performances, le processus suivant pourrait être suivi : une même image est donnée en input à plusieurs OCR. Un LLM compare les différents outputs des différents OCR. Si les différents OCR débouchent sur les mêmes phrases, la qualité de l'OCR est considérée comme fiable. Si en revanche le LLM détecte des différences entre les différents outputs, l'utilisateur intervient manuellement. Il est clair qu'un tel procédé est plus coûteux computationnellement en temps et en énergie, mais également qu'ils présupposent que les différents OCR mobilisés soient à la fois performants mais diffèrent au niveau du réseau de neurone qui les constitue. Il semble en effet possible, dans le cas où les différents OCR sont similaires, qu'ils fassent les mêmes erreurs.
