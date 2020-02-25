# splunk-etalab-dvf
Splunk App for Etalab DVF data

Splunk with Docker:
https://hub.docker.com/r/splunk/splunk/
https://github.com/splunk/docker-splunk/
https://github.com/splunk/splunk-ansible/
https://www.splunk.com/en_us/blog/cloud/announcing-splunk-on-docker.html
https://www.splunk.com/en_us/blog/tips-and-tricks/hands-on-lab-sandboxing-with-splunk-with-docker.html


docker run -d -p 8000:8000 -e SPLUNK_START_ARGS=--accept-license -e SPLUNK_PASSWORD=torototo -e SPLUNK_APPS_URL=https://api.github.com/repos/sghaskell/maps-plus/tarball/master,https://api.github.com/repos/utano/splunk-etalab-dvf/tarball/master,https://api.github.com/repos/utano/splunk-etalab-dvf-data/tarball/master --name splunk splunk/splunk:7.3
docker exec -it splunk bash

ansible@971c580e3076:/opt/splunk$ sudo su - splunk
$ /bin/bash --init-file ${SPLUNK_HOME}/bin/setSplunkEnv
splunk@971c580e3076:~$ 


Download CSV data from etalab:
All CSV files are available here: https://cadastre.data.gouv.fr/data/etalab-dvf/latest/csv/

Import data into Splunk:
-> Add Data
-> Upload
-> Select File: 2019-31.csv.gz
-> Next
Create a source type: csv-etalab
	Timestamp:
		Extraction: Advanced
		Timestamp format: %Y-%m-%d
		Timestamp fields: date_mutation
	Advanced:
		MAX_DAYS_AGO: 10000
-> Next
-> Index: csv-etalab
-> Review
-> Submit

Install Maps+:
Download Maps+ from https://github.com/sghaskell/maps-plus
Install Maps+ on Splunk
-> Apps
-> Install app from file
-> Submit


Search with Maps+:

index="csv-etalab-dvf" sourcetype="csv-etalab-dvf" longitude="*" latitude="*" code_postal IN (31700 31840 31820 31770 31880 31830 31170 31270)
| eval surface_eval = coalesce(surface_reelle_bati,surface_terrain) 
| search surface_eval="*"
| eval prixm2_eval = valeur_fonciere/surface_eval
| eval description = "<b>".
coalesce(nature_mutation,"?")." ".
coalesce(type_local,nature_culture,"?").
"<br/>".
coalesce(adresse_numero,"")." ".
coalesce(adresse_nom_voie,"")." ".
coalesce(code_postal,"")." ".
coalesce(nom_commune,"").
"<br/>".
coalesce(tostring(surface_eval,"commas"),"?")." m2 - ".
coalesce(tostring(valeur_fonciere,"commas"),"?")." €".
"<br/>".
coalesce(tostring(prixm2_eval,"commas"),"?")." € / m2".
"<br/>".
coalesce(date_mutation,"?").
"</b>"
| table latitude longitude description
