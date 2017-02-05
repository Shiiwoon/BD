
# Base de Donnée
### Postgresql

>Installation
```bash
    #apt-get update 
    #apt-get install postgresql postgresql-contrib     postgresql-doc-9.4 
```

>Modifier le mot de passe de l'utilisateur *système* postgres

        #passwd postgres
>Modifier le mot de passe de l'utilisateur *administrateur de la base* postgres

        #su - postgres
        psql -d template1 -c "ALTER USER postgres WITH PASSWORD 'mdp'"
        
>Créer une base et ce connecté
---
        #su - postgres
        createdb baseDBTest
        psql baseDBTest

>Option de psql

        \h >> aide mémoire commande sql 
        \q >> Quitter la base 
        \? >> aide mémoire commande psql
> Créer un utilisateur 

        #psql -d nomDeLaBase -c "CREATE USER utilisateur WITH PASSWORD 'mdp'"
        #GRANT ALL PRRIVILEGES ON DATABASE nomDeLaBase TO utilisateur;
> Gérer la base a distance (phpMyAdmin/phpPgAdmin)

Il faut activer l'acces dans la base 

        #su - postgress
        psql nomDeLaBase < /usr/share/postgresql/9.4/extension/adminpack--1.0.sql
        
>Activation des connections TCP / IP

Vérifier ces 2 éléments

        #vi /etc/postgresl/9.4/main/postgresl.conf
        listen_addresses='localhost'
        password_encryption=on

        #service postgresql restart
        
>Administration WEB (Serveur apache2 avec php5)

        #apt-get install apache2 php5 libapache2-mod-php5
        #service apache2 restart
        #vi /var/www/html/info.php
        <?php phpinfo() ?>
        
    Tester http://ipServeur/info.php si marche pas vérifer http://ipServeur
    
>Installation de phppgAdmin (Interface d'administration PHP(phpmyadmin < dangereux))


        #apt-get install phppgadmin
        #vi /etc/apache2/conf-available/phppgadmin.conf
        Commenter Require local
        Ajouter Allow From all
        #service apache2 restart
        #vi /etc/phppgadmin/confg.inc.php
        vérifier
        $conf['extra_login_security']=false;
        #service postgresql restart
        #service apache2 restart
        
        Connexion : http://ipServeur/phppgadmin
        
>Installation de TinyTinyRSS ( Agrégateur RSS léger recomander par le prof en veille techno )
    
Module PHP 5.4(ou nouveau) requis  
php5-json php5-curl php5-gd 

https://tt-rss.org/gitlab/fox/tt-rss/wikis/PhpCompatibilityNotes
https://tt-rss.org/gitlab/fox/tt-rss/wikis/InstallationNotes

    #apt-get install git-core
    #git clone https://tt-rss.org/git/tt-rss.git /var/www/html/tt-rss
    
    Connexion : http://ipServeur/tt-rss/install


Mis A Jour de TT-RSS
https://tt-rss.org/gitlab/fox/tt-rss/wikis/UpdatingFeeds

>Réplication "Streaming" PostgreSQL

Besoin de 2 machine Debian avec PSQL d'installé:
**Master IP :**  x.x.x.1
**Slave IP :** x.x.x.2

>**MASTER** 

        #vi /etc/postgresql/9.4/main/postgresl.conf
        listen_addresses='*'
        wal_level=hot_standby
        wal_keep_segment=10(augmenter si la derniere commande donne une table d'état de réplucation empty)
        max_wal_sender=3
        
>Créer un utilisateur spécifique pour la réplication

psql -h localhost -U postgres -W -c "CREATE USER utilrepl WITH REPLICATION PASSWORD 'mdp';"

>Autorisation pour ce connecter

        #vi /etc/postgresl/9.4/main/pg_hba.conf
        host replication utilrepl x.x.x.2/mask  md5 (voir pour le SHA1 ?)
        #service postresql stop

>**SLAVE**

        #vi /etc/postgresql/9.4/main/postgresql.conf
        listen_addresses='*'
        hot_standby=on
        
> Arret de l'esclave et déstruction de tout les fichiers de la base(attention pédro)


