#!/usr/bin/env python3.6
import boto3
from datetime import datetime
import time
import os
s3_resource = boto3.resource('s3')
mon_bucket = "monprojet6"

from argparse import ArgumentParser
parser = ArgumentParser()
parser.add_argument("-f", action="store_true",dest="chemin",help="chemin de location restauration")
args = parser.parse_args()

début = time.time()

# On determine le moment du début de l'execution de la sauvegarde
if args.chemin:

	f = open(os.path.join('/home/adminsys/fichier_aws.txt')).read().splitlines()

	for fichier in f:
		s3_resource.Bucket(mon_bucket).\
		upload_file(Filename = f'{fichier}',Key =os.path.basename(fichier))

# Methode de chargement d'un fichier dans le bucket par son nom
# On determine le moment de la fin de la sauvegarde
	fin = time.time()

# On affiche un message indiquant la fin de la sauvegarde
	print("-----------------------------------------")
	print("Sauvegarde multiple effectuée avec succès")
	print("-----------------------------------------")

#On affiche un message indiquant le temps de la sauvegarde
	print(f"Temps de chargement: {fin - début} secondes")
	print("-----------------------------------------")
else:
	fichier = input("Quel fichier voulez vous sauvegarder? : ")
	chemin = input("Veuillez entrer le chemin du fichier à sauvegarder :")
s3_resource.Bucket(mon_bucket).upload_file(Filename = f'{chemin}{fichier}',Key = fichier)
