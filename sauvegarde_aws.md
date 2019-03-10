#!/usr/bin/env python3.6

# Importation du module boto3
import boto3

# Importation du module sys
import sys

#Importation du module os
import os

# Importation du module botocore
import botocore

# Importation du module datetime afin d'enregistrer la date et l'heure d'exécution
from datetime import datetime
import time

# Importation du module ArgumentParser de la librairie argparse
from argparse import ArgumentParser

# On crée l'objet parser pour ajouter des options à l'analyseur de paramètres
parser = ArgumentParser()

# On ajoute les informations relatives à l'appel de l'analyseur de paramètres (commande d'appel,
# type etc..)
# On ajoute la valeur par defaut dans argparse qui represente le chemin du fichier au cas ou le paramatre -f n'est pas renseigné
parser.add_argument("-f","--chemin",help = "chemin du fichier contenant la liste des fichiers")
parser.add_argument("-b","--bucket",help="nom du bucket")
parser.add_argument("-c","--path",help="chemin du fichier à sauvegarder/restaurer")
parser.add_argument("-s","--sauvegarde",action="store_true",help="appel du module de sauvegarde")
parser.add_argument("-r","--restauration",action="store_true",help="appel du module de restauration")
parser.add_argument("-a","--affichage",action="store_true",help="appel du module d'affichage du contenu du bucket")

# On déclare la variable args qui représente la methode parse_args utilisée par Argumentparser
args = parser.parse_args()

# On affecte la variable mon_bucket afin d'entrer en paramètre le nom du bucket avec le parser
mon_bucket = args.bucket

# On affecte la variable path afin d'entrer en paramètre le chemin avec le parser
path = args.path

path = str(path)

chemin = args.chemin

s3 = boto3.resource('s3')

conn = boto3.client('s3')

# On affecte le chemin du fichier contenant la liste des fichiers par défaut
liste_defaut="/home/adminsys/fichier_aws.txt"

# sauvegarde():-----------------------------------------------------------------------------------------------------------------------

# On determine le moment du début de l'execution de la sauvegarde

# On cré la fonction sauvegarde
def sauvegardes(liste):

	début = time.time()

	# On affecte la variable erreur afin de l'incrémenter dans la boucle "for"
	erreur = 0
	erreur = int(erreur)
	# On verifie l'existance du fichier contenant la liste des fichiers
	try:
		assert os.path.exists(f'{liste}')
	except AssertionError:
		print('fichier contenant la liste introuvable')
		sys.exit(1)

	# Lecture ligne par ligne du fichier liste
	with open(os.path.join(f'{liste}')) as file:
		f = file.read().splitlines()

	# On ouvre le fichier de log
	l = open("logs_sauvegarde.txt","a")

	# Création d'un boucle for afin de créer una variable por chaque fichiers de la liste
	for fichier in f:

		# Methode de chargement dans le bucket combiné de conditions afin de véerifier la présence des fichiers en local 
		if os.path.exists(fichier):

			s3.Bucket(mon_bucket).\
			upload_file(Filename = f'{fichier}',Key =os.path.basename(fichier))

			# On inscrit la sauvegarde dans un fichier de log
			l.write(f"[SUCCESS] fichier '{os.path.basename(fichier)}' sauvegardé dans {mon_bucket} {datetime.now()}\n")

		else:
			# On incremente les erreurs dans la variable erreur
			erreur += 1

			# Création d'un fichier de log afin d'enregistrer les erreurs
			l.write(f"[ERROR] fichier '{os.path.basename(fichier)}' introuvable dans {os.path.dirname(fichier)}: {datetime.now()}\n")

	l.close()

	#On affiche un message indiquant le temps de la sauvegarde
	fin = time.time()

	print("------------------------------------------")
	print(f"Temps de traitement: {fin-début} secondes")

	# On affiche un message indiquant la fin de la sauvegarde et le code erreur
	if erreur >= 1:
		print("----------------------------------")
		print("** Erreur lors de la sauvegarde **")
		print("----------------------------------")
		sys.exit(1)
	else:

		print("---------------------------")
		print("** Tout s'est bien passé **")
		print("---------------------------")
		sys.exit(0)


def sauvegarde_manuelle():

	#On cré une boucle while afin de demander à l'utilisateur d'enter un nom de fichier ou quitter le programme
	while True:

		fichier = input("Quel fichier voulez vous sauvegarder? ('q' pour quitter): ")

			# Création d'un bloc try afin de sortir du programme en tapant 'q'
		try:
			assert fichier != 'q'

		except AssertionError:

			sys.exit(0)

		début = time.time()

			# Methode de chargement dans le bucket combiné de conditions afin de véerifier la présence des fichiers en local 
		if os.path.exists(f'{path}{fichier}'):
			s3.Bucket(mon_bucket).upload_file(Filename = f'{path}{fichier}',Key = fichier)

				# On affiche un message indiquant la fin de la sauvegarde
			print("---------------------------------------------------------------")
			print(f"** Sauvegarde de '{fichier}' effectuée avec succès dans {mon_bucket} **")
			print("---------------------------------------------------------------")
			break

		else:

			# On affiche un message indiquant le fichier en erreur
			print(f"[ERROR] fichier '{os.path.basename(fichier)}' introuvable dans {path}")
			continue

	fin = time.time()

	#On affiche un message indiquant le temps de la sauvegarde
	print(f"Temps de traitement: {fin-début} secondes")
	print("------------------------------------------")

	sys.exit(0)



# restauration():-----------------------------------------------------------------------------------------------------------------------

def restaurations(liste):

	début = time.time()
	# On affecte la variable erreur afin de l'incrémenter dans la boucle "for"
	erreur = 0
	erreur = int(erreur)

	# On verifie l'existance du fichier contenant la liste des fichiers
	try:
		assert os.path.exists(liste)
	except AssertionError:
		print('fichier contenant la liste introuvable')
		sys.exit(1)

	# Lecture ligne par ligne du fichier liste
	with open(os.path.join(f'{liste}')) as file:
		f = file.read().splitlines()

	# On ouvre le fichier de log
	l = open("logs_restauration.txt","a")

	# Création d'un boucle for afin de créer una variable por chaque fichiers de la liste
	for fichier in f:

		chemin = os.path.dirname(fichier)
		nom_du_fichier = os.path.basename(fichier)

		# Methode de téléchargement combiné d'un bloc try afin d'afficher une erreur si le fichier n'existe pas dans le bucket 
		try:
			s3.Object(mon_bucket,nom_du_fichier)\
			.download_file(f'{chemin}/{nom_du_fichier}')

			l.write(f"[SUCCES] fichier '{nom_du_fichier}' restauré dans {chemin} {datetime.now()}\n")

		except botocore.exceptions.ClientError as e:
			if e.response['Error']['Code'] == "404":

				# On affiche un message indiquant le fichier en erreur
				erreur += 1
				# Création d'un fichier de log afin d'enregistrer les erreurs

				l.write(f"[ERROR] fichier '{nom_du_fichier}' non existant dans le bucket {mon_bucket}: {datetime.now()}\n")
	l.close()
	fin = time.time()

	print("------------------------------------------")
	print(f"Temps de traitement: {fin-début} secondes")


	# On affiche un message indiquant la fin de la restauration et le code erreur
	if erreur >= 1:
		print("------------------------------------")
		print("** erreur lors de la restauration **")
		print("------------------------------------")
		sys.exit(1)


	# On affiche un message indiquant la fin de la restauration
	else:
		print("---------------------------")
		print("** Tout s'est bien passé **")
		print("---------------------------")
		sys.exit(0)


def restauration_manuelle():

	# Création d'une boucle while en cas d'erreur ou demande de sortie d programme
	while True:

		fichier = input("Quel fichier voulez vous restaurer? ('q' pour quitter): ")

		# Création d'un bloc try afin de sortir du programme en tapant 'q'
		try:

			assert fichier != 'q'

		except AssertionError:

			sys.exit(0)

		début = time.time()

		# Methode de téléchargement combiné d'un bloc try afin d'afficher un erreur si le fichier n'existe pas dans le bucket 
		try:

			s3.Object(mon_bucket,fichier).download_file(f'{path}{fichier}')
		except botocore.exceptions.ClientError as e:
			if e.response['Error']['Code'] == "404":

				# On affiche un message indiquant le fichier en erreur
				print(f'[ERROR] fichier {fichier} non existant dans le bucket')
				continue
		else:

			# On affiche un message indiquant la fin de la restauration
			print("-------------------------------------------------------------------")
			print(f"** Restauration de '{fichier}' effectuée avec succès dans {path} **")
			print("-------------------------------------------------------------------")
			break

	#On affiche un message indiquant le temps de la sauvegarde
	fin = time.time()
	print(f"Temps de traitement: {fin-début} secondes")
	print("------------------------------------------")

	sys.exit(0)

# affichage()--------------------------------------------------------------------------------------

def affichages():

	#On cré une boucle while afin de confirmé le nom du bucket auquel nous voulons affichier le contenu
	while True:

		#Capture de la reponse de l'utilisateur combiné d'un bloc try afin de s'assurer de la réponse
		contenu = input(f'Voulez vous afficher le contenu du bucket \"{mon_bucket}\" ? (o ou n) : ')

		try :

			assert contenu == 'o' or contenu == 'n'

		except AssertionError:
			print("Veuillez entrer o ou n : ")
			continue

		#Méthode d'affichage du bucket si la reponse est "o"
		if contenu == 'o':
			for key in conn.list_objects(Bucket = mon_bucket)['Contents']:
				print(key['Key'])
			break
		else:
			break


	sys.exit(0)

# Execution globale()-------------------------------------------------------------------------------

if args.sauvegarde:

	if args.path:
		sauvegarde_manuelle()
	elif args.chemin:
		liste = args.chemin
		sauvegardes(liste)
	else:
		liste = liste_defaut
		sauvegardes(liste)

elif args.restauration:

	if args.path:
		restauration_manuelle()
	elif args.chemin:
		liste = args.chemin
		restaurations(liste)
	else:
		liste = liste_defaut
		restaurations(liste)

elif args.affichage:

	affichages()

else:

	print("----------------------------------------------------------------------------------")
	print("-- Veuillez entrer une option : -s(sauvegarde) , -r(restauration) , -a(affichage)")
	print(", -b(bucket) , -c(chemin) , -b(bucket)")
	print("Exemple de sauvegarde en indiquant le fichier contenant la liste des fichiers à traiter :")
	print("./script_sauvegarde.py -s -f /chemin/vers/lefichier/ -b nom_du_bucket")
	print("----------------------------------------------------------------------------------")
