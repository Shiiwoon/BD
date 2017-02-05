Fabrice EDGARD 4x4

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


>Supervision et tunning MySQL

-   MySQLTuner
         #wget https://raw.github.com/major/MySQLTuner-perl/master/mysqltuner.pl
        ./mysqltuner.pl
-   MyTop
        #apt-get install mytop
        mytop --prompt -d NomBaseDeDonne

>Modification du fichier de configuration

1.  key_buffer:Regarder dans SHOW GLOBAL STATUS si
key_reads/key_read_requests ratio est inférieur à 0.01 pour éviter l'accès disque
Si non, augmenter le key-buffer

2. thread_concurrency (présent uniquement si machine multicœurs) :
2X le nombre de cœurs
3. Activer le log requêtes lentes dans my.cnf:
log-slow- queries=/home/mysqld-slow.log
Et demander d'indiquer le temps le plus lent
long_query_time = 1
Analyser les résultats:
tail -n 1000 /home/mysqld-slow.log
4. Vérifier qu'il n'y a pas de "connection leakage": there are not too many
open connections in the database.
Utiliser SHOW PROCESSLIST;
5. Analyser les verous avec :
SHOW STATUS LIKE 'Table%';
et SHOW ENGINE INNODB STATUS (si vous utilisez INNODB comme moteur)

>Sauvegardes, complètes et partielles

*On sauvegarde souvent une (ou plusieurs) base en omettant de sauvegarder
également les utilisateurs et leurs privilèges. Ces informations sont dans la table users de
la base mysql*

- A chaud
    -   mysqldump: cli
(SCRIPT USE):MYSQL_PWD=motDePasse
    mysqldump --user=root --all-databases | gzip > save.mysql.sql.gz
    -   mysqlworkbench,phpmyadmin,ect
-   A froid (Arret du serveur)
    -   Archiver /var/lib/mysql + /var/log/mysql.log
    -   Pour les tables MyISAM > .FRM , .MYI et .MYD
    -   Pourt les tables InnoDB > .FRM, .ibdata(data file) ,(table file si innodb_file_per_table)idb, .iblogfile(journalisation intermédiaire)
>Restauration

- A chaud

            mysql -uuser -p
            mysql>use database baseDeDonne
            mysql>source /home/user/mysql/db.sql
            (SCRIPT USE):mysql -uuser--password=mdp </home/user/mysql/db.sql
- A froid(service arréter)
> 
Redéposer les fichiers archivée dans leurs répertoires

>Rsnapshot

        #apt-get install rsnapshot
        /etc/rsnapshot.conf > fichier de conf
        
        vi /etc/cron.d/rsnapshot
        Décommenter les cron pour autoriser les backup
        
        cp /usr/share/doc/rsnapshot/examples/utils/backup_mysql.sh /usr/local/bin/
        
        chown root:root /usr/local/bin/backup_mysql.sh
        chmod o-w /usr/local/bin/backup_mysql.sh
        
        vi /usr/bin/mysqldump --all-databases > mysqldump_all_databases.sql
        OU base par base
        /usr/bin/mysqldump --defaults-file=/etc/mysql/debian.cnf DATABASE > DATABASE.SQL
        vi /etc/rsnapshot.conf
        backup_script /usr/local/bin/backup_mysql.sh TAB localhost/mysqldump
        rsnapshot configtest
        
        cron pour programmer les sauvegarde
        
>Migration de MySQL vers MariaDB

*Faire une sauvegarde avant migration*
 
        #apt-get update && apt-get upgrade
        #apt-get install python-software-properties-common
        
        #apt-key adv --recv-key --keyserver hkp://keyserver.ubuntu.com:80 0xcbcb082a1bb943db (enlever hkp:// et :80 si !IUT) 
        
        vi /etc/apt/sources.list.d/mariadb.list
            deb http://fr.mirror.babylon.network/mariadb/repo/10.1/debian jessie main
        
        #apt-get update
        #apt-get install mariadb-server
        cat /etc/lsb-release
*Des messages de conflit vont s’afficher (normal on fait une migration) et le système va demander d’accepter la suppression de Mysql*
      
           -- Vérification --
        mysql -u root -p
        
>Cluster MariaDB / Galera

Avoir une 2eme machine debian + ssh et installer DIRECT mariaDB

*Production = 3 serveurs minimum*

>MASTER
    
      #service mysql stop
        mysqld --wsrep-new-cluster
        OU
        galera_new_cluster (systemd)
        
>SLAVE

        service mysql stop (pas obligatoire mais peut servir)
        mysqld --wrep_cluster_address=gmcomm://ipMASTER
        
        Pour vérifier
        mysql -uuser -pmdp
        SHOW STATUS LIKE 'wrep_%';
        # mysql -u root -e 'SELECT VARIABLE_VALUE as "cluster size" FROM INFORMATION_SCHEMA.GLOBAL_STATUS WHERE VARIABLE_NAME="wsrep_cluster_size"'
        
>Changement du répertoire de logs MySQL

*Car reduit la réactivité du **cluster** en generant un grand nombre d'IO*

        vi my.cnf
        log-bin=/NouveauCheminRépertoire/mysql-bin
        
        purge binlog
        
        service mysql stop
        
        mv /var/lib/mysql/mysql-bin.* /NouveauCheminRepertoire/mysql-bin
        
        #service mysql start
