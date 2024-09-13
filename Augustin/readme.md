Nous avons choisi d'utiliser [Google Cloud vision](https://cloud.google.com/vision?hl=fr) car ce modèle permet d'exporter un fichier `.json` donnant des informations sur la **mise en forme** du fichier qu'on fait passer dans l'OCR.

Le code python prend le fichier .json exporté par Cloud Vision OCR et le transforme en fichier PDF.
En l'occurence, il s'agit des 15 premières pages du livre "Le vocabulaire de Saint Augustin".
