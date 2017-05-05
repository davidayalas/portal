+++
date        = "2017-05-01"
title       = "Connectar amb una BBDD Mongo amb SSL a través de NodeJS"
description = "Connectar amb una BBDD Mongo amb SSL a través de NodeJS"
section     = "howtos"
categories  = ["canigo"]
key         = "MAIG2017"
+++

### A qui va dirigit

Aquest how-to va dirigit a tots aquells que vulguin connectar-se amb una base de dades Mongo que requereix autenticació SSL(Per exemple un Mongo creat a Compose) des d'una aplicació NodeJS.

### Introducció

Quan es crea una base de dades Mongo a Compose, la url de connexió que es genera conté el paràmetre **ssl=true**. Això provoca que per a connecar-se des d'una aplicació nodejs s'hagi d'afegir el certificat.

### Crear certificat

S'ha d'accedir a Compose i copiar el certificat proporcionat (SSL Certificate (Self-Signed)) i el desem a un fitxer, per exemple **mongodbcert.cert**, que quedarà de la següent manera:

	-----BEGIN CERTIFICATE-----
	Codi certificat
	-----END CERTIFICATE-----

### MongoClient

Si s'utilitza el package mongodb el codi de la nostra aplicació serà el següent:

	var MongoClient = require('mongodb').MongoClient
		, format = require('util').format;

	var fs = require('fs');
		
	var caFile = fs.readFileSync(__dirname + '/mongodbcert.cert');
		
	MongoClient.connect('mongodb://user:password@host:port/db', 
		{ 	mongos: {
				ssl: true,
				sslValidate: true,
				sslCA: caFile,
				poolSize: 1,
				reconnectTries: 1
			}
		}
		,function (err, db) {
			if (err) {
				throw err;
			} else {
				console.log("successfully connected to the database");
			}
		db.close();
	});
	
A la variable caFile es carrega el certificat i afegim un paràmetre a Mongoclient.connect amb les opcions de connexió a la propietat **sslCA**.

Per a configurar ssl, hem d'afegir les propietats **ssl** i **sslValidate** a true
	
### Mongoose

Si s'utilitza el package mongoose el codi de la nostra aplicació serà el següent:

	const mongoose = require('mongoose');
	const fs = require('fs');

	const mongoOpt = {
			  "server": {
				"auto_reconnect": true,
				"reconnectTries": 1,
				"socketOptions": {
				  "keepAlive": 1,
				  "connectTimeoutMS": 30000
				}
			  },
			  "mongos": {
				"ssl": true,
				"sslValidate": true,
				"sslCA": fs.readFileSync(__dirname + '/mongodbcert.cert')
			  }
			};

	mongoose.connect('mongodb://user:password@host:port/db', mongoOpt);

	var db = mongoose.connection;
	db.on('error', function(err) {
		console.error('Error in MongoDB connection: ' + err);
		mongoose.disconnect();
	});

	db.on('connected', function() {
		console.log('Connexió amb el servidor de BBDD');
		mongoose.disconnect();
	});

Quan es realitza la connexió, al paràmetre d'opcions (mongoOpt) s'han d'afegir les propietats **ssl** i **sslValidate** a true i **sslCA** amb el fitxer del certificat