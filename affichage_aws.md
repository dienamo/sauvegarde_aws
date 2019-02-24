#!/usr/bin/env python3.6

from boto3 import client

conn = client('s3')

from argparse import ArgumentParser

parser = ArgumentParser()

parser.add_argument("-b","--bucket",help = "nom du bucket")

args = parser.parse_args()

mon_bucket = args.bucket


while True:

	contenu = input(f'Voulez vous afficher le contenu du bucket {mon_bucket}? (o ou n) : ')

	try :

		assert contenu == 'o' or contenu == 'n'

	except AssertionError:
		print("Veuillez entrer o ou n : ")
		continue

	if contenu == 'o':
		for key in conn.list_objects(Bucket = mon_bucket)['Contents']:
			print(key['Key'])
		break
	else:
		break


