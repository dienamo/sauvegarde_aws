#!/usr/bin/env python3.6

from boto3 import client

conn = client('s3')

While True:
	contenu = input(f'Voulez vous afficher le contenu du bucket {mon_bucket}? (o ou n) : ')
	try : 
		assert contenu == 'o' or contenu == 'n'
	except AssertionError
		print("Veuillez entrer o ou n : ")
		continue

	if contenu == 'o':
		for key in conn.list.objects(Bucket = mon_bucket)['Contents']:
			print(key['Key']
