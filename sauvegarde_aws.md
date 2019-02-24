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
parser.add_argument("-f",action = "store_true",dest="chemin",help="chemin de location restauration")
parser.add_argument("-b","--bucket",help="nom du bucket")
parser.add_argument("-c","--path",help="chemin de location restauration")
parser.add_argument("-s","--sauvegarde",action="store_true",help="appel du module de sauvegarde")
parser.add_argument("-r","--restauration",action="store_true",help="appel du module de restauration")

# On déclare la variable args qui représente la methode parse_args utilisée par Argumentparser
args = parser.parse_args()

# On affecte la variable mon_bucket afin d'entrer en paramètre le nom du bucket avec le parser
mon_bucket = args.bucket

# On affecte la variable path afin d'entrer en paramètre le chemin avec le parser
path = args.path

s3 = boto3.resource('s3')

# sauvegarde():-----------------------------------------------------------------------------------------------------------------------
if args.sauvegarde:

	# On determine le moment du début de l'execution de la sauvegarde
	début = time.time()
	if args.chemin:

		f = open(os.path.join('/home/adminsys/fichier_aws.txt')).read().splitlines()

		# Création d'un boucle for afin de créer una variable por chaque fichiers de la liste
		for fichier in f:

			# Methode de chargement dans le bucket combiné de conditions afin de véerifier la présence des fichiers en local 
			if os.path.exists(fichier):

				s3.Bucket(mon_bucket).\
				upload_file(Filename = f'{fichier}',Key =os.path.basename(fichier))

				# On affiche un message indiquant la fin de la sauvegarde
				print("-----------------------------------------------")
				print("** Sauvegarde multiple effectuée avec succès **")
				print("-----------------------------------------------")

			else:

				# On affiche un message indiquant le fichier en erreur
				print(f'fichier {os.path.basename(fichier)} introuvable dans {os.path.dirname(fichier)}')

				# Création d'un fichier de log afin d'enregistrer les erreurs
				f = open("Fichiers_en erreur.txt","a")
				f.write(f'fichier {os.path.basename(fichier)} introuvable dans {os.path.dirname(fichier)}: {datetime.now()}\n')
				f.close()
		# On determine le moment de la fin de la sauvegarde
		fin = time.time()

	else:
		while True:

			fichier = input("Quel fichier voulez vous sauvegarder? ('q' pour quitter): ")

			# Création d'un bloc try afin de sortir du programme en tapant 'q'
			try:
				assert fichier != 'q'

			except AssertionError:

				sys.exit()

			# Methode de chargement dans le bucket combiné de conditions afin de véerifier la présence des fichiers en local 
			if os.path.exists(f'{path}{fichier}'):
				s3.Bucket(mon_bucket).upload_file(Filename = f'{path}{fichier}',Key = fichier)

				# On affiche un message indiquant la fin de la sauvegarde
				print("---------------------------------------------------------------")
				print(f"** Sauvegarde de {fichier} effectuée avec succès dans {mon_bucket} **")
				print("---------------------------------------------------------------")
				break

			else:

				# On affiche un message indiquant le fichier en erreur
				print(f'fichier {os.path.basename(fichier)} introuvable dans {path}')
				continue
	fin = time.time()

	#On affiche un message indiquant le temps de la sauvegarde
	print("----------------------------------------------")
	print(f"Temps de téléchargement: {fin-début} secondes")
	print("----------------------------------------------")

# restauration():-----------------------------------------------------------------------------------------------------------------------
elif args.restauration:

	début = time.time()

	if args.chemin:

		f = open(os.path.join('/home/adminsys/fichier_aws.txt')).read().splitlines()

		# Création d'un boucle for afin de créer una variable por chaque fichiers de la liste
		for fichier in f:

			chemin = os.path.dirname(fichier)
			nom_du_fichier = os.path.basename(fichier)

			# Methode de téléchargement combiné d'un bloc try afin d'afficher une erreur si le fichier n'existe pas dans le bucket 
			try:
				s3.Object(mon_bucket,nom_du_fichier)\
				.download_file(f'{chemin}/{nom_du_fichier}')
			except botocore.exceptions.ClientError as e:
				if e.response['Error']['Code'] == "404":

					# On affiche un message indiquant le fichier en erreur
					print(f"Fchier {nom_du_fichier} non existant dans le bucket")					f = open ("fichiers_en_erreur.txt","a")

					# Création d'un fichier de log afin d'enregistrer les erreurs
					f = open ("fichiers en erreur.txt","a")
					f.write(f'fichier {nom_du_fichier} non existant dans le bucket {mon_bucket}: {datetime.now()}\n')
					f.close()
			else:

				# On affiche un message indiquant la fin de la sauvegarde
				print("----------------------------------------------")
				print("** Restauration multiple effectuée avec succès **")
				print("----------------------------------------------")
	else:

		# Création d'une boucle while en cas d'erreur ou demande de sortie d programme
		while True:

			fichier = input("Quel fichier voulez vous restaurer? ('q' pour quitter): ")

			# Création d'un bloc try afin de sortir du programme en tapant 'q'
			try:

				assert fichier != 'q'

			except AssertionError:

				sys.exit()

			# Methode de téléchargement combiné d'un bloc try afin d'afficher un erreur si le fichier n'existe pas dans le bucket 
			try:

				s3.Object(mon_bucket,fichier).download_file(f'{path}{fichier}')
			except botocore.exceptions.ClientError as e:
				if e.response['Error']['Code'] == "404":
	
					# On affiche un message indiquant le fichier en erreur
					print(f'Fichier {fichier} non existant dans le bucket')
					continue
			else:
				break

	# On affiche un message indiquant la fin de la restauration
	print("-------------------------------------------------------------------")
	print(f"** Restauration de {fichier} effectuée avec succès dans {path} **")
	print("-------------------------------------------------------------------")

	# On determine la fin de la restauration
	fin = time.time()

	#On affiche un message indiquant le temps de la sauvegarde
	print("----------------------------------------------")
	print(f"Temps de téléchargement: {fin-début} secondes")
	print("----------------------------------------------")


# On demande à l'utilisateur de choisir une option si celui ci n'entre aucun paramètre
else:
	print("-- Veuillez entrer une option -s(sauvegarde) ou -r(restauration) --")

