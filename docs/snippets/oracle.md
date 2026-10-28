# Oracle

## Connect with SQL\*Plus

```text
SQL> connect / as sysdba
SQL> show con_name
```

## Create a user

```text
SQL> connect system/admin@127.0.0.1:1521/XEPDB1
SQL> CREATE USER pepe IDENTIFIED BY pepe;
SQL> grant all privileges to pepe;
```

Minimum permission to let a user log in:

```sql
GRANT CREATE SESSION TO user_name;
```

## Change a password

```sql
ALTER USER usuari IDENTIFIED BY currentPassword REPLACE newPassword;
```


