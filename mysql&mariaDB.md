# MySQL & MariaDB


>Installation de LAMP

        #apt-get update
        #apt-get install mysql-server mysql-clien
                (attention mdp root)
        vi /etc/mysql/my.cnf
            127.0.0.1 > 0.0.0.0 (POUR LE TP)
        #service mysql restart
        
        #apt-get install apache2
        Vérifier > http://IP
        
        #apt-get install php5 libaapache2-mod-php5 php-mysql+ module UTILE
        vi /var/www/html/info.php
            <? phpinfo(); ?>
        #service apache2 restart
        Verifier > http://IP/info.php que les module soit bien chargés

> Installation de PHPMyAdmin
    #apt-get install phpmyadmin
    
    Vérifier > http://IP/phpmyadmin
    (SI PROBLEME : Crée le lien symbolique de /usr/share/phpmyadmin vers /var/www/html/phpmyadmin)
    
    wget http://wordpress.org/latest.zip
    
    mysql -u root -p
    mysql> CREATE DATABASE wordpress;
    mysql> CREATE USER 'wordpress';
    mysql> SET PASSWORD FOR ''wordpressuser' = PASSWORD('mdp');
    mysql>GRANT ALL PRIVILEGES ON wordpress.* wordpressuser@localhost IDENTIFIED by 'mdp';
    mysql> exit
    
    apt-get install zip
    unzip latest.zip -d /var/www/html
    
    chown -Rf www-data:www-data /var/www/wordpress
    chmod 775 /var/www/wordpress
    
    http://IP/wordpress
    
Commandes et effect

|Commande | Effet |
|------------------------------|---|
| GRANT INSERT ON * TO invite; | Droit de faire des insert PARTOUT pour l'utilisateur 'invite'|
|GRANT INSERT ON table1 TO invite| Droit de faire des insert sur la table 1 pour 'invite'|
|GRAND ALL ON * TO invite| TOUS les droit PARTOUT pour invite|
|REVOKE INSERT ON * FROM invite|Retrait du droit de faire des insert partout pour invite|
|REVOKE INSERT ON table1 FROM invite| Retrait de faire des insert sur la table 1 pour invite|
|REVOKE ALL PRIVILEGES ON * FROM invite| Retrait de tous les droits pour invite|
|repair table nomDeLaBase.nomTable| Répare la table(table crashed should be repaired)|

>Sécurisation du service MySQL

        #mysql_secure_instalation
Permet:
-   de s'assurer d'avoir un mot de passe root MySQL
-   d'empêcher d'accéder en root MySQL depuis le Réseau(connexion via mysql bloquer)
-   d'empêcher les connexions anonymes
-   supprimer la base de test(accesible de tous avec tout les droits)

> Récupération de mot de passe root MySQL



        #/etc/init.d/mysql stop 
     
        # mysqld_safe --skip-grant-tables & 
        
        # mysql -u root 
   
        mysql> use mysql; 
        mysql> update user set password=PASSWORD("nouveaumotdepasse") where user='root'; 
        mysql> flush privileges; 
        mysql> quit 
     
        # /etc/init.d/mysql stop 
     
        # /etc/init.d/mysql start 
        
        # mysql -u root -p 
### #Bah?Elle est ou ma promotion ?=D

>Supervision et tunning MySQL

-   MySQLTuner
         #wget https://raw.github.com/major/MySQLTuner-perl/master/mysqltuner.pl
        ./mysqltuner.pl
-   MyTop
        #apt-get install mytop
        mytop --prompt -d NomBaseDeDonne

>Modification du fichier de configuration

1.  key_buffer:
2.  