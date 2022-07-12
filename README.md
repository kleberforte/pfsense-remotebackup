# 🐧 Script Backup pfSense

```bash
#!/bin/bash

killall wget

caminho=/home/usuario/Backups/pfSense
cliente=NomeDoCliente
endereco=https://urldocliente.dominio.com
usuariopfsense=admin
senhapfsense=minhasenha

[ -d "$caminho/$cliente"/ ] || mkdir "$caminho/$cliente"

wget -qO- --keep-session-cookies --save-cookies cookies.txt \
--no-check-certificate $endereco/diag_backup.php \
| grep "name='__csrf_magic'" | sed 's/.*value="\(.*\)".*/\1/' > csrf.txt

wget -qO- --keep-session-cookies --load-cookies cookies.txt \
--save-cookies cookies.txt --no-check-certificate \
--post-data "login=Login&usernamefld=$usuariopfsense&passwordfld=$senhapfsense&__csrf_magic=$(cat csrf.txt)" \
$endereco/diag_backup.php  | grep "name='__csrf_magic'" \
| sed 's/.*value="\(.*\)".*/\1/' > csrf2.txt

wget --keep-session-cookies --load-cookies cookies.txt --no-check-certificate \
--post-data "download=download&backuprrd=yes&__csrf_magic=$(head -n 1 csrf2.txt)" \
$endereco/diag_backup.php -O "$caminho/$cliente"/pfSense$cliente-`date +%d.%m.%y-%R`.xml

# Removendo temporarios
rm cookies.txt
rm csrf.txt
rm csrf2.txt

# Compactando o arquivo
# --transform='s!.*/!!' pra nao criar o caminho completo do arquivo
tar -czf "$caminho/$cliente"/pfSense$cliente-`date +%d.%m.%y`.tgz --transform='s!.*/!!' "$caminho/$cliente"/pfSense$cliente-`date +%d.%m.%y-*`.xml

# Alterando permissoes
chmod 644 "$caminho/$cliente"/pfSense$cliente-`date +%d.%m.%y`.tgz
chown -R seuusuario:grupousuario "$caminho/$cliente"/pfSense$cliente-`date +%d.%m.%y`.tgz

# Removendo o arquivo XML
rm "$caminho/$cliente"/pfSense$cliente-`date +%d.%m.%y-*`.xml
```

