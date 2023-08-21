# Задача: создать бекапы клиента на сервере с borg

1. Устанавливаем borg на сервере и клиенте
```
# yum install epel-release -y
# yum install borgbackup -y
```
2. Создаем на сервере пользователя borg, который будет работать на наше благо + директорию .ssh
```
# useradd -m borg 			
# mkdir /var/backup
# chown borg:borg /var/backup/

# su - borg
# mkdir .ssh
# touch .ssh/authorized_keys
# chmod 700 .ssh
# chmod 600 .ssh/authorized_keys
```
3. На клиенте генерим ключ ssh-keygen и подкидываем его серверу  бекапов.

4. Дальше на клиенте выполняем инициализацию репозитория для бекапов, отправляя на сервер удалённую команду
<details>
  
```
[vagrant@client ~]$ borg init --encryption=repokey borg@192.168.11.160:/var/backup/
The authenticity of host '192.168.11.160 (192.168.11.160)' can't be established.
ECDSA key fingerprint is SHA256:qJU14ZGz+hueqe6dQsWAMRmdKyLQBKYnZTTdcc71Rcs.
ECDSA key fingerprint is MD5:76:6e:32:93:3c:51:13:84:3c:6b:e1:13:c4:6d:1d:8c.
Are you sure you want to continue connecting (yes/no)? yes
Remote: Warning: Permanently added '192.168.11.160' (ECDSA) to the list of known hosts.
Enter new passphrase: 
Enter same passphrase again: 
Do you want your passphrase to be displayed for verification? [yN]: y
Your passphrase (between double-quotes): "***"
Make sure the passphrase displayed above is exactly what you wanted.

By default repositories initialized with this version will produce security
errors if written to with an older version (up to and including Borg 1.0.8).

If you want to use these older versions, you can disable the check by running:
borg upgrade --disable-tam ssh://borg@192.168.11.160/var/backup

See https://borgbackup.readthedocs.io/en/stable/changes.html#pre-1-0-9-manifest-spoofing-vulnerability for details about the security implications.

IMPORTANT: you will need both KEY AND PASSPHRASE to access this repo!
If you used a repokey mode, the key is stored in the repo, but you should back it up separately.
Use "borg key export" to export the key, optionally in printable format.
Write down the passphrase. Store both at safe place(s).
```
</details>

5. Запускаем бэкап
```
[vagrant@client ~]$ borg create --stats --list borg@192.168.11.160:/var/backup/::"etc-{now:%Y-%m-%d_%H:%M:%S}" /etc
```
<details>
 <summary>вывод</summary>
  
```
------------------------------------------------------------------------------
Archive name: etc-2023-08-10_10:04:34
Archive fingerprint: 94769ea47ec563885eed66deec150d8927783a2e1b94156415b47fce530aa950
Time (start): Thu, 2023-08-10 10:04:42
Time (end):   Thu, 2023-08-10 10:04:43
Duration: 0.97 seconds
Number of files: 412
Utilization of max. archive size: 0%
------------------------------------------------------------------------------
                       Original size      Compressed size    Deduplicated size
This archive:               17.45 MB              5.86 MB              5.85 MB
All archives:               17.45 MB              5.86 MB              5.85 MB

                       Unique chunks         Total chunks
Chunk index:                     405                  411
------------------------------------------------------------------------------
```
</details>
6. Проверка наличия бэкапа на сервере. Всё на месте

```
[vagrant@client ~]$ borg list borg@192.168.11.160:/var/backup/
Enter passphrase for key ssh://borg@192.168.11.160/var/backup: 
etc-2023-08-10_10:04:34              Thu, 2023-08-10 10:04:42 [94769ea47ec563885eed66deec150d8927783a2e1b94156415b47fce530aa950]
```
  Можно и на сервере проверить
  
```
[borg@server-backup ~]$ borg list /var/backup
Enter passphrase for key /var/backup: 
etc-2023-08-10_10:04:34              Thu, 2023-08-10 10:04:42 [94769ea47ec563885eed66deec150d8927783a2e1b94156415b47fce530aa950]
```

7. 
