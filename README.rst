script de sauvegarde/restauration AWS

script de sauvegarde de données sur Amazon S3 grâce à la librairie BOTO3
installation

L'installation de la dernière version de BOTO3 se fait avec PIP:

pip install boto3

configuration

avant de pouvoir utiliser ce module il faut d'abord parametrer son systeme avec les informations d'authentification afin de pouvoir connecter son compte AWS avec la commande : "aws configure". cette commande vous nous permettre de renseigner les clés d'accès a notre compte AWS ainsi que les paramètres régionaux.

AWS Access key ID [None]: AWS Secret Access key [None]: Default region name [None]:

toutes ces information sont disponible dans les paramètres de compte AWS.
utilisation

afin d'utilise boto3 il faut importer le module "import boto3"

et indiquer le service que nous voulons utiliser,afin d'accéder à notre bucket nous allons utiliser la méthode boto3.resources('s3')

"s3 = boto3.ressource('s3')".

on peut ensuite interagir avec notre bucket en y sauvegardant des fichiers s3_resource.Bucket(mon_bucket).upload_file(Filename="/chemin/nom_du_fichier,key=nom_du_fichier).

afin de lever une exception botocore.exceptions.ClientError qui est une exception du module botocore créée en cas de non présence du fichier dans notre bucket

méthode de téléchargement d'un fichier avec les méthodes s3.Objects().download_files()

afin de restaurer un fichier en local à partir de notre bucket sur s3

création d'un bloc try

afin de lever une exception botocore.exceptions.ClientError qui est une exception du module botocore créée en cas de non présence du fichier dans notre bucket

    méthode de téléchargement d'un fichier avec les méthodes s3.Objects().download_files()

la restauration se fait en téléchargeant un objet à partir de s3 comme ceci:

s3.objects(bucket,fichier.download_file(chemin,fichier)).download_file

afin de lister le contenu du bcket utiliser la methode conn.list_object for fichiers in conn.list_objects(Bucket=mon_bucket)['Contents']

paramètrer le crontab afin de planifier une sauvegarde automatique
explication

    importation de la librairie boto3

pour l'utilisation de la librairie dans notre script

    importation des modules datetime et time

afin de capturer la date et le temps d'exécution du script

    declaration du service sur AWS auquel nous avons besoin (s3)

maintenant que nous avons déclaré une ressource nous pouvons effectuer des requêtes et obtenir des réponses de ce service

    affectation de la variable mon_bucket contenant le nom de mon bucket
    affactation de la variable début

afin de determiner le moment du début du chargement

    méthode de chargement du fichier de boto3 avec les fonctions s3.resource.Bucket().upload_file()

afin de faire la sauvegarde du fichier dans le bucket sur s3

    affectation de la variable fin

afin de determiner le momenbt de la fin du chargement

    on affiche un message indiquant la fin de la sauvegarde
    on affiche un message indiquant la durée de la sauvagerde en soustrayant le variable fin à la

variable début

    affectation de la variable conn

afin de se connecter au bucket et appeler la fonction list_object()

    utilisation d'une boucle while

afin de demander à l'utilisateur si il veut afficher ou non le contenu du bucket après la sauvegarde

    affectation de la variable affichage

afin de capturer la réponse de l'utilisateur

    création d'un bloc try contenant des conditions if,elif,else

afin de lever une erreur d'assertion au cas ou l'utilisateur ne tape pas "o" ou "n"

    méthode de connexion et listage du contenu bucket avec la fonction conn.list_objects()

combinée d'ne boucle for. afin d'afficher chaque élèments parcourus

    inscription de la date d'execution du script et du temps de chargement dans un fichier avec la

méthode write() afin d'avoir un historique de la date exécution et vérifier le temps de chargement
