📘 Make sure your OpenShift cluster is running and you're logged in before applying the YAML.
## 🚀 Create the deployment mysql

Run the following command to create the mysql application:
```bash
oc new-project dbproj
oc create secret generic mysql-secret \
  --from-literal=MYSQL_ROOT_PASSWORD=myrootpass \
  --from-literal=MYSQL_USER=myuser \
  --from-literal=MYSQL_PASSWORD=mypassword \
  --from-literal=MYSQL_DATABASE=mydatabase
oc new-app mysql
oc set env deploy/mysql --from=secret/mysql-secret
oc set volume deploy/mysql --add --name=mysql-data -t pvc --claim-size=1G --claim-name=mysql-pvc --mount-path=/var/lib/mysql
oc rollout restart deployment/mysql
````
## 🌐 Delete the mysql pod.
```bash
oc delete pod <mysql-pod-xx>
```

## 🌐 Create tables and add data to the mysql database.
```bash
oc exec -it <mysql-pod-name> -- /bin/bash
bash-4.4$ 
sh-4.4$ mysql -u $MYSQL_USER -p
mypassword
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mydatabase         |
| performance_schema |
+--------------------+
3 rows in set (0.01 sec)
mysql> USE mydatabase;
Database changed
mysql> SHOW TABLES;
Empty set (0.01 sec)

mysql> CREATE TABLE employees (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100),
  position VARCHAR(100),
  salary DECIMAL(10, 2)
);

Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO employees (name, position, salary) VALUES
  ('Alice', 'Engineer', 70000.00),
  ('Bob', 'Manager', 85000.00),
  ('Carol', 'Analyst', 60000.00);

Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM employees;
+----+-------+----------+----------+
| id | name  | position | salary   |
+----+-------+----------+----------+
|  1 | Alice | Engineer | 70000.00 |
|  2 | Bob   | Manager  | 85000.00 |
|  3 | Carol | Analyst  | 60000.00 |
+----+-------+----------+----------+
3 rows in set (0.00 sec)

mysql> SHOW TABLES;
+----------------------+
| Tables_in_mydatabase |
+----------------------+
| employees            |
+----------------------+
1 row in set (0.00 sec)

mysql> exit;
```


## 🌐 Delete the pod and verify the data still present in mysql database.
```bash
oc delete pod <mysql-pod-name>
oc exec -it <mysql-pod-name> -- /bin/bash
bash-4.4$ 
sh-4.4$ mysql -u $MYSQL_USER -p
mypassword
mysql> SHOW DATABASES;
mysql> USE mydatabase;
mysql> SHOW TABLES;
mysql> SELECT * FROM employees;

```
