---
title: Projet Unix
author: Pierre Poulain
license: Creative Commons Attribution - Partage dans les Mêmes Conditions 4.0
---

MEG M1 option Bioinformatique Génomique

# Objectifs et contexte

L'objectif de ce projet est de vous donner une première expérience sur la manipulation et l'analyse de données obtenues par des techniques de séquençage haut débit (RNA-Seq ici).

Vous allez vous intéresser au Complexe Majeur d'Histocompatibilité (CMH), localisé sur le bras court du chromosome 6. Le CMH est une des régions du génome humain la plus polymorphe.

Plus précisément, vous allez étudier l'impact de la séquence de référence du génome dans la cartographie des données issues de séquençage. Pour cela, vous comparerez deux jeux de séquences (*reads*) obtenues par séquençage haut débit de deux lignées cellulaires humaines (PGF et COX). La séquence individuelle de chaque lignée est connue.

Après une étape de nettoyage, vous cartographierez les *reads* sur la séquence du génome de référence pour le CMH (haplotype PGF) et sur la séquence du génome de COX. Enfin, vous analyserez et visualiserez vos résultats.

Vous aurez à votre disposition des articles scientifiques sur le CMH.


## Données

Pour réaliser ce projet, vous utiliserez l'archive (`project_data.tgz`) qui contient :

- Dans le répertoire `chr6p_samples`, deux jeux de séquences, obtenues par RNA-Seq, au format fastq compressé (.fastq.gz). Le jeu de données GUP-1 correspond à l'haplotype PGF. Le jeu de données GUP-3 correspond à l'haplotype COX. Pour chaque échantillon, vous aurez à votre disposition deux fichiers car les données sont pairées (les *reads* R1 dans un sens et les *reads* R2 dans l'autre sens).
- Dans le répertoire `chr6p_genomes`, les séquences complètes, au format fasta, des deux génomes de références : PGF (référence pour le CMH) et COX.
- Dans le répertoire `chr6p_annotations`, les annotations des deux génomes de référence, au format gtf.

**Remarque** : le jeu de données initial était trop gros pour être manipulé dans la cadre de ce projet. Pour chaque échantillon, la taille des fichiers `.fastq.gz` était d'environ 900 Mo chacun. Il y a deux fichiers `.fastq.gz` par échantillon. Pour que vous puissiez néanmoins manipuler ces données, seules les séquences correspondantes au bras court du chromosome 6 ont été conservées. Chaque fichier `.fastq.gz` fait de l'ordre de 40 Mo. De même, les séquences des deux génomes de référence (PGF et COX) ont été limitées au bras court du chromosome 6. Les fichiers correspondants font 60 Mo chacun.


## Environnement informatique et script d'automatisation

Vous aurez également à votre disposition :

- Un serveur Linux `serv-bioinfo` pour lancer les analyses nécessaires. Pour rappel, le serveur `serv-bioinfo` est accessible par ssh. L'adresse complète du serveur ainsi que votre identifiant vous ont été communiqués par votre enseignant.

- Les logiciels nécessaires pour votre analyse ont été installées dans un environnement conda. Ce type d'environnement tend à devenir une référence en bioinformatique aujourd'hui. Si vous souhaitez en apprendre plus sur conda, consultez [cet article](https://bioinfo-fr.net/comment-fixer-les-problemes-de-deploiement-et-de-durabilite-des-outils-en-bioinformatique) et [celui-ci](https://bioinfo-fr.net/conda-le-meilleur-ami-du-bioinformaticien).

- Un script (`workflow.sh`) qui contient *uniquement* le processus d'analyse pour l'échantillon GUP-1 (PGF) cartographié sur les génomes de référence de PGF et COX.


## Analyses des données de séquençage

Le processus d'analyse des données est constituée des étapes :

1. Nettoyage des *reads* : suppression des adaptateurs et des *reads* de mauvaise qualité. Outil : `trimmomatic` (java).

2. Indexation des génomes qui vont être utilisés pour la cartographie. Sans indexation, la cartographie des *reads* sur le génome serait extrêmement lente. Outil : `hisat2` (Perl et Python).

3. Cartographie des *reads* sur le génome puis indexation des *reads* cartographiés. Outils : `hisat2` (Perl et Python) puis `samtools` (C).

4. Analyse de la cartographie. Outil : `samtools` (C).

5. Visualisation des données. Outil : `IGV` (java).


Les outils `trimmomatic`, `hisat2` et `samtools` sont disponibles sur le serveur `serv-bioinfo`. L'outil de visualisation `IGV` n'est pas installé sur le serveur mais il sera à installer sur votre machine d'analyse (au Script ou à la maison). IGV est téléchargeable sur le site du [Broad Institute](https://software.broadinstitute.org/software/igv/download).


# Travail demandé

0. Test du fonctionnement du script `workflow.sh` sur le jeu de données du projet.

    - Connectez-vous à la machine *serv-bioinfo* et créez le répertoire `project` dans votre répertoire utilisateur. Déplacez-vous dans ce répertoire.

    - Déclarez les variables d'environnements `http_proxy` et `https_proxy` qui définissent le proxy web :
        ```
        $ export http_proxy=http://www-cache.script.univ-paris-diderot.fr:3128/
        $ export https_proxy=http://www-cache.script.univ-paris-diderot.fr:3128/
        ```

    - Activez conda puis activer l'environnement `unix-project` qui contient les logiciels dont vous aurez besoin :
        ```
        $ source /opt/bioinfo/soft/miniconda3/etc/profile.d/conda.sh
        $ conda activate unix-project
        ```
        L'expression `(unix-project)` devrait alors apparaître à gauche de votre invite de commande.

    - Avec la commande `wget`, téléchargez les fichiers `project_data.tgz` et `workflow.sh` depuis les adresses suivantes :
        ```
        http://cupnet.net/docs/MEG_projet/project_data.tgz
        https://raw.githubusercontent.com/pierrepo/meg-unix-project/master/workflow.sh
        ```

    - Vérifiez les empreintes md5 des fichiers que vous avez téléchargés. Elles doivent être :
        ```
        9d7c5ff96e72848eb09d18cb9abbb4fd  project_data.tgz
        977663a5d8759927000088ad295179a5  workflow.sh
        ```

    - Décompressez l'archive `data_project.tgz` avec la commande :
        ```
        $ tar zxvf project_data.tgz
        ```
        Vérifiez que vous avez bien 3 répertoires : `chr6p_samples`, `chr6p_genomes` et `chr6p_annotations`.

    - Rendez exécutable le script que vous avez téléchargé avec la commande :
        ```
        $ chmod +x workflow.sh
        ```

    - Enfin, lancez le script d'analyse :
        ```
        $ ./workflow.sh
        ```

    - L'exécution du script va prendre entre 5 à 15 min, suivant la charge de la machine *serv-bioinfo*.

    Le fichier `workflow-XXX-YYY.log` (avec `XXX` la date et `YYY` l'heure à laquelle le script a été lancé) contient les temps d'exécution des différentes étapes ainsi que les résultats de quelques analyses. Si le processus d'analyse s'est exécuté correctement, le message `Well done!` doit apparaître à la fin du fichier.

1. Modification du script (`workflow.sh`) pour faire l'analyse complète des données.

    En partant de  `workflow.sh`, vous nommerez votre script `workflow2.sh`. Votre script devra effectuer toutes les opérations suivantes :

    - Nettoyage des *reads* des échantillons GUP-1 et GUP-3.

    - Indexation des génomes de références PGF et COX.

    - Cartographie puis indexation des *reads* des échantillons GUP-1 puis GUP-3 sur PGF et GUP-1 puis GUP-3 sur COX.

    - Analyse du nombre de *reads* cartographiés, avec ou sans mismatch, pour toutes les cartographies possibles (GUP-1 sur PGF, GUP-1 sur COX, GUP-3 sur PGF et GUP-3 sur COX).

    L'utilisation de boucles sera ici pertinente.

    **Remarque :** Pour modifier votre script `workflow2.sh`, vous pouvez travailler directement sur le serveur *serv-bioinfo*. Vous pourrez alors utiliser un éditeur comme `nano` :

    ```
    $ nano workflow2.sh
    ```

    Utilisez les combinaisons de touches *ctrl + o* puis *Enter* pour enregistrer et *ctrl + x* pour quitter l'éditeur.

    Vous pouvez également travailler depuis votre machine au Script ou à la maison, mais il faudra penser à copier votre script depuis votre machine vers *serv-bioinfo*.


2. Lancement de votre script `workflow2.sh` sur la machine *serv-bioinfo*.

    - Vérifiez bien que le script `workflow2.sh` est bien exécutable, que vous êtes dans le répertoire `project` et que vous avez activé conda et l'environnement `unix-project`.

    - Lancez-le... et croisez les doigts :

        ```
        $ ./workflow2.sh
        ```

    - Si cela ne fonctionne pas, corrigez votre script.

3. Analyse de l'effet du nettoyage des *reads* sur le nombre et la taille moyenne des *reads*.

    - Expliquez rapidement le format de fichier `.fastq`.

    - Calculez le nombre de *reads* dans les 4 fichiers *.fastq.gz* fournis (2 pour GUP-1 et 2 pour GUP-3), avec la commande suivante (à lancer depuis le répertoire `chr6p_samples`) :

        ```
        $ zgrep -c '^+$' *.fastq.gz
        ```

        **Remarque :** La commande `zcat` est l'équivalente de la commande `cat`, mais pour les fichiers de type texte, compressés par gzip.

    - Calculez le nombre de *reads* dans les 4 fichiers `.fastq.gz` après nettoyage (2 pour GUP-1 et 2 pour GUP-3, uniquement pour les *reads* *appariés*, fichiers `*P.fastq.gz`), avec la commande suivante (à lancer depuis le répertoire `chr6p_trim`) :

        ```
        $ zgrep -c '^+$' *P.fastq.gz
        ```

    - Expliquez le fonctionnement des commandes Unix précédentes. Déterminez pour chaque échantillon, le pourcentage de *reads* conservés par le nettoyage. Quel a été l'effet du nettoyage sur le nombre de *reads* ?

    - Calculez la taille moyenne des *reads* pour les 4 fichiers `.fastq.gz` fournis (2 pour GUP-1 et 2 pour GUP-3), avec la commande suivante (à lancer depuis le répertoire `chr6p_samples`) :

        ```
        $ for name in *.fastq.gz; do echo ${name}; zcat ${name} | awk 'NR%2==0' | awk 'NR%2==1' | awk '{l+=length($0); c+=1} END {print "mean length:", l/c}'; done
        ```

    - Et de manière similaire, calculez la taille moyenne des *reads* après nettoyage (2 pour GUP-1 et 2 pour GUP-3, uniquement pour les *reads* *appariés*, fichiers `*P.fastq.gz`), avec la commande suivante (à lancer depuis le répertoire `chr6p_trim`) :

        ```
        $ for name in *P.fastq.gz; do echo ${name}; zcat ${name} | awk 'NR%2==0' | awk 'NR%2==1' | awk '{l+=length($0); c+=1} END {print "mean length:", l/c}'; done
        ```

    - Expliquez en **détail** le fonctionnement des commandes Unix précédentes. Comparez et discutez les valeurs obtenues. Quel a été l'effet du nettoyage sur la taille moyenne des *reads* ?


4. Analyse du nombre de *reads* cartographiés suivant le génome utilisé.

    - Récupérez les données dans le fichier `workflow2-XXX-YYY.log`. Comparez les nombres totaux de *reads* cartographiés et les ratios nombre de *reads* cartographiés sans mismatch / nombre total de *reads* cartographiés.

    - Concluez sur le rôle du génome de référence sur la cartographie des *reads*.

5. Rapatriement des données.

    Une fois que vous avez réussi à faire vos analyses, déconnectez-vous de *serv-bioinfo*.

    Rapatriez, sur votre machine (au Script ou à la maison), les données du projet, en lançant la commande (**depuis votre machine**) :

    ```
    $ scp -r login@adresse-serv-bioinfo:~/project/chr6p*  ./
    ```

    Attention cependant à la gestion de votre espace disque (voir plus bas).

6. Visualisation avec le logiciel IGV.

    - Installez IGV sur une machine Linux ou FreeBSD (à tester pour Mac) :

        ```
        $ wget http://data.broadinstitute.org/igv/projects/downloads/2.4/IGV_2.4.16.zip
        $ unzip IGV_2.4.16.zip
        $ cd IGV_2.4.16
        ```

        Et lancez IGV :

        ```
        $ ./igv.sh
        ```

        depuis le répertoire d'IGV (`IGV_2.4.16`).

    - Une fenêtre graphique devrait s'afficher sur votre écran.

        - Dans le menu *Genomes*, *Load Genome from File*, sélectionnez un génome dans le répertoire `chr6p_index`. Par exemple, `chr6p-pgf.fa`. L'information correspondante devrait s'afficher en haut de la fenêtre.

        - Dans le menu *File*, *Load from File*, sélectionnez une cartographie de *reads* dans le répertoire `chr6p_map` qui corresponde au génome de référence. Par exemple, les *reads* de GUP-1 aligés sur le génome PGF, `GUP-1_6p_on_chr6p-pgf.bam`. L'information correspondante devrait s'afficher au milieu de la fenêtre.

        - Enfin, dans le menu *File*, *Load from File*, sélectionnez dans le répertoire `chr6p_annotations` l'annotation correspondante au génome déjà chargé. Par exemple, pour les annotations du génome PFG `chr6-pgf.gtf`. L'information correspondante devrait s'afficher en bas de la fenêtre.

        - Zoomez sur le chromosome 6 en cliquant sur le bouton de zoom **+** en haut à droite de la fenêtre.

    - Dans la fenêtre à gauche du bouton *Go* en haut, tapez *HLA-B* puis cliquez sur *Go*.

    - Comparez l'expression du gène *HLA-B* chez PGF (avec GUP-1) et chez COX (avec GUP-3).

    - Observez également l'épissage alternatif et la différence d'expression chez PGF (avec GUP-1) et COX (avec GUP-3) du gène *HLA-DPB1*.

    - Comparez et discutez vos résultats avec la littérature scientifique.



# Documents demandés et critères d'évaluation

Tous les documents demandés seront organisés dans un répertoire dont le nom aura la forme `projet_unix_NOM_PRENOM` avec, bien sur, `NOM` et `PRENOM` à adapter. Ce répertoire sera ensuite archivé et compressé dans un fichier au format `.tgz` sous le nom `projet_unix_NOM_PRENOM.tgz`. Cette archive sera déposée sur le site du cours sur Moodle avant le :

**26 janvier 2019 20h00**

Contenu du répertoire `projet_unix_NOM_PRENOM` :

1. Votre script `workflow2.sh`.

2. Le fichier de sortie `workflow2-XXX-YYY.log`.

3. Un rapport au format PDF (avec pour nom `rapport_NOM_PRENOM.pdf`) qui contiendra les éléments suivants :

    - Une introduction sur le contexte biologique (CMH). Utilisez et citez la bibliographie mise à votre disposition.

    - Une présentation des données qui vous ont été fournies (origine, taille, particularité). Pour chaque type de données, vous décrirez en quelques lignes le format utilisé.

    - Une rapide présentation des différentes manipulations que vous avez réalisées. Vous préciserez en particulier la version des logiciels utilisés. Vous produirez également un schéma représentant le processus d'analyse avec les différentes étapes, les outils et les données.

    - Les explications et les analyses demandées dans les questions 3, 4 et 6 de la section précédente. N'hésitez pas à ajouter quelques images pertinentes pour la question 6.

    - Une conclusion sur l'influence du génome de référence pour la cartographie des *reads* et sur ce que vous avez appris. Utilisez la bibliographie mise à votre disposition. Indiquez également, les difficultés que vous avez rencontrées lors de ce projet et comment vous les avez surmontées.

    L'orthographe, la grammaire, la syntaxe, l'organisation et la présentation générale du rapport seront prises en compte lors de l'évaluation. Notre nom, prénom et numéro d'étudiant devront figurer sur la première page de votre rapport. Je vous conseille la lecture du document [Rédiger un rapport scientifique](http://cupnet.net/rapport-scientifique/).

    Votre rapport fera entre 5 et 10 pages (strict), avec une taille de police et une mise en page adéquate pour que sa lecture en soit agréable.

Par ailleurs, vous devez impérativement respecter les noms et les formats de fichiers demandés. Soyez en particulier très attentifs à la casse.


# Coup de pouce

Si vous en avez le besoin, le fichier `project_results_help.tgz` vous sera communiqué ultérieurement. Ce fichier contient les données d'une analyse complète réalisée sur ma machine. Vous aurez également à votre disposition le fichier de sortie `workflow2-XXX-YYY.log` correspondant. Pour ces deux fichiers, vous aurez les empreintes MD5 correspondantes.

Au cas où vous n'arriveriez pas à créer le script `workflow2.sh` et donc faire l'analyse complète vous-même, vous pourrez utiliser ces données pour réaliser les analyses demandées dans les questions 3, 4 et 6. Si tel est le cas, **indiquez le clairement** dans votre rapport.


# Gestion de l'espace disque

Les données que vous allez manipuler et générer pour ce projet ont une taille importante. Vous disposez d'un quota (espace maximum autorisé) de 1 Go sur les machines du Script et de 4 Go sur la machine *serv-bioinfo*. Faites très attention quand vous copiez des données depuis *serv-bioinfo* vers les machines du Script car vous pouvez atteindre votre quota très facilement et bloquer votre compte.

Pour vous aider dans cette gestion, la commande `du -ch` vous sera utile. Par exemple,

```
$ du -ch *
16M     chr6p_annotations
115M    chr6p_genomes
297M    chr6p_index
257M    chr6p_map
143M    chr6p_samples
135M    chr6p_trim
175M    project_data.tgz
4,0K    workflow_20171211-003932.log
4,0K    workflow2_20171211-010343.log
8,0K    workflow2.sh
8,0K    workflow.sh
1,2G    total
```

Si vous souhaitez réaliser l'étape de visualisation avec IGV sur votre machine au Script, vous pouvez vous contenter de ne rapatrier que les fichiers nécessaires à la visualisation avec la commande :

```
$ scp -r login@adresse-serv-bioinfo:~/project/chr6p_{index,map,annotations}  ./
```

# Mise en garde

En bioinformatique, il est facile et parfois très tentant de prendre tel quel le script d'un copain pour le sien. Néanmoins, il est aussi très facile de mesurer la similitude entre deux fichiers.

Je vous encourage à travailler à plusieurs et à discuter ensemble, pour comprendre vos erreurs, améliorer votre script, analyser vos résultats. Mais, **votre script et vos analyses devront être originaux et individuels**. Une comparaison sera systématiquement réalisée entre vos différents scripts et résultats (c'est simple et rapide sous Unix).

Par ailleurs, la correction de votre projet sera en partie automatisée et réalisée par un programme. Vous devrez impérativement respecter les instructions qui vous ont été communiquées concernant la nomenclature et l'organisation des fichiers et des répertoires. Vous serez lourdement pénalisés si vous ne respectez pas ces consignes.
