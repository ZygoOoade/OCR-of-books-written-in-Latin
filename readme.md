## Processus

(étape 1) l'utilisateur upload un PDF dont les pages sont au format image.
<br>
(étape 2) l'utilisateur sélectionne une zone rectangulaire d'intérêt contenant seulement le texte d'intérêt sur toutes les pages du PDF uploadé.
<br>
(étape 3) le PDF uploadé est décomposé en autant de fichiers image (`png` ou `jpg`) qu'il contient de pages.
<br>
(étape 4) toutes ces images sont rognées avec les proportions de la zone rectangulaire d'intérêt définie par l'utilisateur à l'étape 2.
<br>
(étape 5) les images passent dans un OCR les unes après les autres.

**Remarques :**
* La sélection d'une zone rectangulaire d'intérêt sert à éliminer les éléments qu'il pourrait y avoir autour (comme un titre dans l'en tête ou un numéro de page), qui seraient captés par l'OCR et ensuite inséré avec le texte lui-même. Nous pensons surtout au texte dans les marges : l'OCR ne détecte pas qu'il est dans la marge et le met donc au milieu d'une phrase du corps du texte, ce qui ferait dysfonctionner notre processus.
* Il faut que la zone rectangulaire minimale contenant le texte d'intérêt ne bouge pas d'une page à l'autre du PDF. Autrement dit, que la taille du pied de page, des marges et de l'en tête restent fixe. Cela n'est souvent pas le cas, par exemple, avec des textes récents où il y a des notes de bas de page. Il nous semble que dans ce cas, il faudrait selectionner la zone rectangulaire pour chaque page du PDF.
* Nous ne savons pas ce que des OCR comme `pytesseract` produiraient avec tous les éléments de mise en forme du texte, avec tous les éléments non textuels éventuellement présents dans le PDF, et avec les caractères non latin. Pour ce qui est de la mise en forme (texte en gras, italique, alinéas, etc.) nous pensons que celle-ci sera perdue après l'OCR. Nous partirons du principe, en général, que les PDF uploadé ne contiennnent que du texte, et donc pas d'image, pas de tableaux, de schémas ou autre.

**Problèmes :**
* Il serait intéressant de savoir si l'OCR fonctionne mieux avec des images de petit format qu'avec des images de grand format. Si tel est le cas, il nous semble qu'il faudrait implémenter - en plus - un découpage des images extraites du PDF.
* Il serait intéressant de savoir si l'OCR a de meilleures perfomances dans des conditions spécifiques. Il est tout à fait possible, en effet, d'employer sur les fichiers `png` des opérations spécifiques comme le fait d'enlever l'arrière plan, d'augmenter la résolution ou bien de faire varier la luminosité et/ou le contraste. Pour ce qui est de la résolution de l'image, il semble probable que cela augmente les performances de l'OCR et augmente aussi son coût/son temps de fonctionnement.

**Pistes**
* Il serait très intéressant de tester et comparer la performance de plusieurs OCR. Pour avoir de meilleures performances, le processus suivant pourrait être suivi : une même image est donnée en input à plusieurs OCR. Un LLM compare les différents outputs des différents OCR. Si les différents OCR débouchent sur les mêmes phrases, la qualité de l'OCR est considérée comme fiable. Si en revanche le LLM détecte des différences entre les différents outputs, l'utilisateur intervient manuellement. Il est clair qu'un tel procédé est plus coûteux computationnellement en temps et en énergie, mais également qu'ils présupposent que les différents OCR mobilisés soient à la fois performants mais diffèrent au niveau du réseau de neurone qui les constitue. Il semble en effet possible, dans le cas où les différents OCR sont similaires, qu'ils fassent les mêmes erreurs.
* Contrairement aux LLMs multimodaux comme Claude 3.5 ou GPT4o, `pytesseract` donne en output le texte de manière brute. Il semble intéressant de distribuer ses résultats à différents LLMs ensuite pour reconstituer le texte original, et d'employer la méthode précédente d'un système de vote où les données reconstituées par un maximum de modèle sont considérées comme fiables.
