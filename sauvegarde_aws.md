#!/usr/bin/env python3.6

# Importation du module boto3
import boto3

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

# On déclare la variable args qui représente la methode parse_args utilisée par Argumentparser
args = parser.parse_args()

mon_bucket = args.bucket

path = args.path

s3 = boto3.resource('s3')

# On determine le moment du début de l'execution de la sauvegarde

début_traitement = time.time()

#def_sauvegarde():-----------------------------------------------------------------------------------------------------------------------

if args.chemin:

	f = open(os.path.join('/home/adminsys/fichier_aws.txt')).read().splitlines()

	for fichier in f:
		if os.path.exists(fichier):
			s3_resource.Bucket(mon_bucket).\
			upload_file(Filename = f'{fichier}',Key =os.path.basename(fichier))
			print("-----------------------------------------")
			print("Sauvegarde multiple effectuée avec succès")
			print("-----------------------------------------")

		else:
			print(f'fichier {os.path.basename(fichier)} introuvable dans {os.path.dirname(fichier)}')
			f = open("Fichiers_en erreur.txt","a")
			f.write(f'fichier {os.path.basename(fichier)} introuvable dans {os.path.dirname(fichier)}\n')
			f.close()
# Methode de chargement d'un fichier dans le bucket par son nom
# On determine le moment de la fin de la sauvegarde
	fin = time.time()

# On affiche un message indiquant la fin de la sauvegarde

#On affiche un message indiquant le temps de la sauvegarde
else:
	while True:

		fichier = input("Quel fichier voulez vous sauvegarder? ('q' pour quitter): ")

		try:
			assert fichier != 'q'

		except AssertionError:

			sys.exit()

		chemin = input("Veuillez entrer le chemin du fichier à sauvegarder :")


		if os.path.exists(fichier):
			s3_resource.Bucket(mon_bucket).upload_file(Filename = f'{chemin}{fichier}',Key = fichier)
			print("---------------------------------------")
			print("Sauvegarde unique effectuée avec succès")
			print("---------------------------------------")
			break
		else:
			print(f'fichier {os.path.basename(fichier)} introuvable dans {chemin}')
			continue
			  
#def_restauration():-----------------------------------------------------------------------------------------------------------------------
			   
if args.chemin:

	f = open(os.path.join('/home/adminsys/fichier_aws.txt')).read().splitlines()

	for fichier in f:

		chemin = os.path.dirname(fichier)
		nom_du_fichier = os.path.basename(fichier)
		try:
			s3.Object(mon_bucket,nom_du_fichier)\
			.download_file(f'{chemin}/{nom_du_fichier}')
		except botocore.exceptions.ClientError as e:
			if e.response['Error']['Code'] == "404":
				print(f"Fchier {nom_du_fichier} introuvable dans le bucket")
				f = open ("fichiers_en_erreur.txt","a")
				f.write(f'fichier {nom_du_fichier} non existant dans le bucket {mon_bucket}: {datetime.now()}\n')
				f.close()
		else:
			print("Restauration multiple effectuée")
else:

	while True:

		nom_du_fichier = input("Quel fichier voulez vous restaurer? ('q' pour quitter): ")

		try:

			assert nom_du_fichier != 'q'

		except AssertionError:

			sys.exit()

		try:

			s3.Object(mon_bucket,nom_du_fichier).download_file(f'{path}{nom_du_fichier}')
		except botocore.exceptions.ClientError as e:
			if e.response['Error']['Code'] == "404":
				print(f'Fichier {nom_du_fichier} non existant dans le bucket')
				continue
		else:
			break

# On affiche un message indiquant la fin de la restauration
		print("-------------------------------------------------------------------")
		print(f"restauration de {nom_du_fichier} effectuée avec succès dans {path}")
		print("-------------------------------------------------------------------")

fin = time.time()

# On affiche le temps de téléchargement
print(f"Temps de téléchargement: {fin-début} secondes")
print("----------------------------------------------")

# On écrit la date et l'heure d'exécution du script dans le fichier date_exécution
f = open("date_execution.txt","a")
f.write(f"{nom_du_fichier} restauré le :{datetime.now()} en {fin-début} secondes\n")
f.close()
