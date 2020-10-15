show grants for root@localhost;   
GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION;    
GRANT ALL PRIVILEGES ON *.* TO 'hive'@'x.x.x.x' IDENTIFIED BY 'hive' WITH GRANT OPTION;    --- 注意@%不好使，必须把ip写好

mysql> select * from mysql.user;
