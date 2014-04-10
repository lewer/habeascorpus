# habeascorpus

Outil d'exploration de corpus. Organisé en trois composants :
- /scripts : permet d'appliquer des algorithmes qui déterminent des topics dans un corpus en utilisant la bibliothèque Gensim (http://radimrehurek.com/gensim/).
- /api : API RESTful (http://www.django-rest-framework.org/) sous forme de serveur qui répond aux requêtes GET (dans un navigateur web) pour renvoyer les données du corpus calculées avec Gensim.
- /browser : un navigateur web Angularjs (http://angularjs.org/) qui consomme l'API pour permettre la visualisation des données, en utilisant notamment d3.js (http://d3js.org/)

## Préparation des données

Le corpus se présente sous la forme d'un fichier TSV (un article par ligne)

Pour SPIP on peut exporter sa base comme suit :

```
echo "SELECT a.id_article,a.titre, a.chapo,a.texte,a.lang, GROUP_CONCAT(DISTINCT u.nom SEPARATOR ', ') AS auteurs, GROUP_CONCAT(DISTINCT m.titre SEPARATOR ', ') AS mots, SUBSTRING(a.date_redac,1,7) AS date FROM spip_articles a LEFT JOIN spip_auteurs_articles au ON a.id_article=au.id_article LEFT JOIN spip_auteurs u ON au.id_auteur=u.id_auteur LEFT JOIN spip_mots_articles am ON a.id_article=am.id_article LEFT JOIN spip_mots m ON am.id_mot=m.id_mot WHERE a.statut IN ('publie') GROUP BY a.id_article;" | mysql $BASE -B > $BASE.tsv
```

(ici avec un test sur le statut 'publié') ; ce format permet d'extraire facilement avec `grep` le nombre d'articles écrits par Untel (ou parlant de "truc"), ou avec `wc` de mesurer le nombre de mots et de signes correspondants (voir http://seenthis.net/messages/224616).

Une fois le fichier `corpus.tsv` obtenu, on le place dans un dossier `data` qui va contenir l'ensemble des fichiers générés par `habeascorpus`. On calcule la représentation bag-of-words du corpus en lançant depuis le dossier `data` :

```
python $habeascorpus/scripts/corpus_to_matrix.py corpus.tsv -v
```

où `$habeascorpus` est le chemin de `habeascorpus` sur le disque dur.

On obtient ainsi dans le dossier `data` le fichier dictionnaire `corpus_wordids.txt` qui associe un id à chaque mot du corpus, et le fichier `corpus_bow.mm`, représentation bag-of-words du corpus.

## Algorithmes

On peut ensuite appliquer l'algorithme LDA qui détermine les topics du corpus (ici on demande 100 topics):

```
python $habeascorpus/scripts/lda.py 100 -v
```

Cette commande produit le fichier `corpus_lda.mm`, qui indique pour chaque document les topics qui lui sont reliés. Elle produit également le fichier `corpus_topics.txt`, qui liste les topics du corpus.

## API

L'API RESTful permet de récupérer les données calculées précédemment en effectuant des requêtes GET.

### Initialisation

Il faut commencer par créer une base de données contenant les données précedemment calculées. Depuis le dossier `data` :

```
python $habeascorpus/manage.py generate_database
```

### Utilisation

Pour démarrer l'API, lancer depuis le dossier `data` la commande :

```
python $habeascorpus/api/manage.py runserver
```

On accède à l'API en entrant l'url http://127.0.0.1:8000/api/ dans un navigateur web.

### Requêtes

- `http://127.0.0.1:8000/api/documents/` : liste des documents du corpus
- `http://127.0.0.1:8000/api/documents/id/` : détails d'un document

- `http://127.0.0.1:8000/api/topics/` : liste des topics
- `http://127.0.0.1:8000/api/topics/id/` : détails d'un topic
- `http://127.0.0.1:8000/api/topics/id/history/` : historique d'un topic

## Navigateur 

Pour visualiser les données dans un navigateur web, démarrer l'API et accéder à l'URL `http://127.0.0.1:8000/static/index.html`




