<<<<<<< HEAD
#User Account Management
---

=======
# User Account Management
---

Note: Following contents have been taken from [MySQL online reference](https://dev.mysql.com/doc/refman/8.0/en/user-names.html).

- MySQL stores accounts in the **user table** of the **mysql system database**.  
- An account is defined in terms of:  

  i) a user name and   
  ii) the client host or hosts from which the user can connect to the server  
- The account may also have a password.  
- User names, as used by MySQL for authentication purposes, have nothing to do with user names (login names) as used by Windows or Unix.  
- MySQL user names can be up to 32 characters long.  
- Passwords stored in the user table are encrypted using plugin-specific algorithms.  

## MySQL Users

- A user in MySQL is a combination of a username and host string.  
- A host string can be an IP address, hostname, fully qualified domain name, or netmask.  
- This means that even though they share a username, admin@192.168.2.10 is different from admin@’192.168.2.%’, and both users can have different passwords and permissions

### Creating a User

The following query creates an account with user = imran, host = localhost,  and password = alishah

```sql 
CREATE USER imran@localhost identified by 'alishah';
```

After the user account has been created it can be confirmed by issuing following query:

```sql 
SELECT user, host from user;
```
>>>>>>> 1e4af661e7bd843a7e9df68cd2bb99ca9f014999
