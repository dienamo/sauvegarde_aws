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
	
	début = time.time()
	
	f = open(os.path.join('/home/adminsys/fichier_aws.txt')).read().splitlines()

	for nom_du_fichier in f:
		s3_resource.Bucket(mon_bucket).\
		upload_file(Filename = f'{nom_du_fichier}',Key =os.path.basename(nom_du_fichier)
			    
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
	début = time.time()
	s3_resource.Bucket(mon_bucket).upload_file(Filename = f'{path}{fichier}',Key = fichier)
	fin = time.time()
# On affiche un message indiquant la fin de la sauvegarde			
	print("-------------------------------------------------------------------")		
	print(f"Sauvegarde de {nom_du_fichier} effectuée avec succès dans {path}")
	print("-------------------------------------------------------------------")

#On affiche un message indiquant le temps de la sauvegarde
	print(f"Temps de chargement: {fin - début} secondes")
	print("-----------------------------------------")
			  
#def_restauration():-----------------------------------------------------------------------------------------------------------------------
			   
if args.chemin:
	début = time.time()
	f = open(os.path.join('/home/adminsys/fichier_aws.txt')).read().splitlines()

	for fichier in f:
		chemin = os.path.dirname(fichier)
		nom_du_fichier = os.path.basename(fichier)
		s3.Object(mon_bucket,nom_du_fichier)\
		.download_file(f'{chemin}/{nom_du_fichier}')
	fin = time.time()
	print("restauration multiple effectuée avec succès")
#On affiche un message indiquant le temps de la sauvegarde
	print(f"Temps de chargement: {fin - début} secondes")
	print("-----------------------------------------")

else:
	nom_du_fichier = input('Quel fichier voulez vous restaurer? :')
	début = time.time()
	try:

		s3.Object(mon_bucket,nom_du_fichier).download_file(f'{path}{nom_du_fichier}')
	except botocore.exceptions.ClientError as e:
		if e.response['Error']['Code'] == "404":
			print("Fichier non existant dans le bucket")
			   
	fin = time.time()
# On affiche un message indiquant la fin de la restauration			
	print("-------------------------------------------------------------------")		
	print(f"restauration de {nom_du_fichier} effectuée avec succès dans {path}")
	print("-------------------------------------------------------------------")
#On affiche un message indiquant le temps de la sauvegarde
	print(f"Temps de chargement: {fin - début} secondes")
	print("-----------------------------------------")
			    
fin_traitement = time.time()
			    
# On écrit la date et l'heure d'exécution du script dans le fichier date_exécution
f = open("date_execution.txt","a")
f.write(f"{nom_du_fichier} restauré le :{datetime.now()} en {fin_traitement-début_traitement} secondes\n")
f.close()
