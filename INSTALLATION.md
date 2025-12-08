# Selvin-Perdomo-PT2

Explicacion de la instalacion de la maquina virtual 
  -# Instal·lació de LAMP stack a Ubuntu 24.04
Per instal·lar una pila LAMP (Linux, Apache, MySQL, PHP) a Ubuntu 24.04, segueix aquests passos detallats. Aquesta guia assumeix que comences amb un sistema net d’Ubuntu 24.04 i tens privilegis `sudo`.

### 1. **Actualitza el sistema**
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. **Instal·la Apache**

```bash
sudo apt install apache2 -y
```

**Activa i inicia el servei:**
```bash
sudo systemctl enable apache2
sudo systemctl start apache2
```

**Verifica l’estat:**
```bash
sudo systemctl status apache2
```

Visita `http://localhost` per veure la pàgina per defecte d’Apache.

### 3. **Instal·la MySQL**

Ubuntu 24.04 ja inclou el paquet `mysql-server` als repositoris oficials (versió 8.0 o superior):

```bash
sudo apt install mysql-server mysql-client -y
```

**Inicia i habilita el servei:**
```bash
sudo systemctl enable mysql
sudo systemctl start mysql
```
**Configura de MySQL:**

#### Accés a la consola de MySQL
```bash
sudo mysql
```

#### Creació de la base de dades
```sql
CREATE DATABASE bbdd;
```

#### Creació de l’usuari local
```sql
CREATE USER 'usuario'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
GRANT ALL PRIVILEGES ON bbdd.* TO 'usuario'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

> **Nota:** Aquest usuari només pot connectar-se des del servidor local (`localhost`), cosa que és suficient si l’aplicació web i la base de dades estan al mateix servidor.

### 4. **Instal·la PHP i extensions comunes**

Ubuntu 24.04 inclou PHP 8.3 als repositoris estàndard:

```bash
sudo apt install php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-zip php-json php-cli -y
```

**Reinicia Apache per carregar PHP:**
```bash
sudo systemctl restart apache2
```

**Verifica la versió de PHP:**
```bash
php -v
```

**Crea un fitxer de prova:**
```bash
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php
```

Visita `http://localhost/info.php` per veure la informació de PHP.

> 🔒 **Mesura de seguretat:** Un cop hagis verificat que funciona, elimina el fitxer:
> ```bash
> sudo rm /var/www/html/info.php
> ```

### Verificació final

La pila LAMP ara hauria d’estar operativa amb:
- **Apache** servint pàgines web.
- **MySQL** preparat per emmagatzemar dades.
- **PHP** processant scripts.

# Configuració de VirtualHost amb apache2

## Introducció

Un servidor web com Apache2 permet allotjar diversos llocs web de forma independent en una mateixa màquina. Aquesta funcionalitat s’assoleix mitjançant **VirtualHosts** (amfitrions virtuals). Cada VirtualHost defineix un lloc web únic amb el seu propi nom de domini, directori d’arrels i configuració específica.

Un cop configurat, podreu accedir al vostre lloc web des d’un navegador utilitzant el nom de domini que hàgiu definit (per exemple, `www.domini.local`).

## 1. Creació de l’estructura de directoris

Per organitzar els vostres llocs web, és recomanable emmagatzemar-los dins del directori per defecte d’Apache: `/var/www/`.

Per al nostre exemple, crearem un directori per al domini `domini.local`:

```bash
sudo mkdir -p /var/www/domini.local
```

> **Nota:** Tot i que podeu emmagatzemar els fitxers en qualsevol ubicació, seguir aquesta convenció facilita la gestió i el manteniment del servidor.

## 2. Definició del VirtualHost

Creeu un fitxer de configuració per al vostre VirtualHost dins del directori `/etc/apache2/sites-available/`:

```bash
sudo nano /etc/apache2/sites-available/domini.local.conf
```

Afegiu-hi la configuració següent (substituïu `domini.local` pel vostre nom de domini):

```apache
<VirtualHost *:80>
    ServerAdmin admin@domini.local
    ServerName www.domini.local
    ServerAlias domini.local
    DocumentRoot /var/www/domini.local
    ErrorLog ${APACHE_LOG_DIR}/domini.local_error.log
    CustomLog ${APACHE_LOG_DIR}/domini.local_access.log combined
</VirtualHost>
```

> **Recomanació:** Utilitzeu fitxers de registre separats per a cada VirtualHost per facilitar la depuració.

## 3. Habilitar el VirtualHost

Apache2 només carrega els VirtualHosts que estan **habilitats**. Per fer-ho, useu la comanda `a2ensite`:

```bash
sudo a2ensite domini.local.conf
```

Aquesta comanda crea un enllaç simbòlic des de `/etc/apache2/sites-available/` cap a `/etc/apache2/sites-enabled/`.

> **Nota:** No cal canviar de directori abans d’executar `a2ensite`; funciona des de qualsevol ubicació.

## 4. Reiniciar Apache2

Després de modificar la configuració, cal reiniciar el servei per aplicar els canvis:

```bash
sudo systemctl restart apache2
```

> **Alternativa:** `sudo service apache2 restart` (funciona, però `systemctl` és l’estàndard modern en sistemes basats en systemd).

## 5. Modificar `/etc/hosts` per resoldre el domini localment

Perquè el vostre sistema resolgui el nom de domini `www.domini.local` cap a la vostra màquina, editeu el fitxer `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

Afegiu la línia següent:

```
127.0.0.1   www.domini.local domini.local
```

Això permet que el navegador trobi el vostre lloc web sense necessitat d’un servidor DNS extern.

## 6. Comprovar el funcionament

Obriu un navegador i accediu a:

```
http://www.domini.local
```

Si el directori `/var/www/domini.local` està buit, Apache pot mostrar un error 403 o una llista de directoris (segons la configuració). Per provar que funciona, creeu un fitxer de prova:

```bash
echo "<h1>Hola, benvingut domini.local</h1>" | sudo tee /var/www/domini.local/index.html
```

Torneu a carregar la pàgina i hauríeu de veure el missatge.

## 7. Solució de problemes: Registres d’Apache2

Si el lloc no funciona com s’espera, consulteu els registres d’Apache:

### Registre d’errors
Conté missatges sobre errors de configuració, permisos, fitxers no trobats, etc.

```bash
sudo tail -f /var/log/apache2/domini.local_error.log
```

### Registre d’accés
Mostra totes les peticions rebudes pel servidor.

```bash
sudo tail -f /var/log/apache2/domini.local_access.log
```

> **Consell:** Useu `tail -f` per veure les entrades en temps real mentre proveu el lloc.

## 8. Assignació de permisos

Apache2 s’executa normalment amb l’usuari `www-data`. Per evitar problemes de permisos, configureu el propietari i els permisos del directori del vostre lloc:

### Canviar el propietari
Permet que el vostre usuari pugui editar fitxers i que Apache els pugui llegir:

```bash
sudo chown -R $USER:www-data /var/www/domini.local
```

### Establir permisos adequats
Assegureu-vos que el propietari i el grup tinguin accés complet, i que altres usuaris només puguin llegir:

```bash
sudo chmod -R 775 /var/www/domini.local
```

> **Explicació:**  
> - `7` (propietari): lectura, escriptura, execució  
> - `7` (grup): lectura, escriptura, execució  
> - `5` (altres): lectura i execució (necessari per accedir a directoris)

## Resum dels passos clau

| Pas | Comanda / Acció |
|-----|------------------|
| Crear directori | `sudo mkdir -p /var/www/domini.local` |
| Configurar VirtualHost | Editar `/etc/apache2/sites-available/domini.local.conf` |
| Habilitar lloc | `sudo a2ensite domini.local.conf` |
| Reiniciar Apache | `sudo systemctl restart apache2` |
| Afegir domini a hosts | `127.0.0.1 www.domini.local` a `/etc/hosts` |
| Verificar permisos | `chown` i `chmod` com s’indica |
| Depurar errors | Consultar `error.log` i `access.log` |



# Guia d’instal·lació i configuració de plataformes cloud (Nextcloud / ownCloud)  
**Dins d’un virtual host preconfigurat (`/var/www/domini.local`)**

Aquesta guia explica com instal·lar **Nextcloud** o **ownCloud** en un entorn on ja tens un **virtual host actiu** apuntant a `/var/www/domini.local` (per exemple, `domini.local`). No cal configurar Apache ni el virtual host, ja que es considera ja operatiu.

## 1. Descàrrega i instal·lació de la plataforma cloud

### 1.1. Enllaços oficials

- **Nextcloud**: [https://www.nextcloud.com](https://www.nextcloud.com)  
  Descàrrega directa:  
  [https://download.nextcloud.com/server/releases/latest.zip](https://download.nextcloud.com/server/releases/latest.zip)

- **ownCloud**: [https://www.owncloud.org](https://www.owncloud.org)  
  Descàrrega directa (versió estable):  
  [https://download.owncloud.com/server/stable/owncloud-complete-20240724.zip](https://download.owncloud.com/server/stable/owncloud-complete-20240724.zip)

> **Nota**: Nextcloud és compatible amb PHP 8.1+, mentre que **ownCloud encara requereix PHP 7.4** en moltes versions estables. Assegura’t de tenir la versió de PHP adequada abans d’instal·lar.


### 1.2. Passos d’instal·lació

1. **Mou’t al directori del virtual host**:
   ```bash
   cd /var/www/domini.local
   ```
2. **Neteja el contingut actual** (si cal):
   > Assegura’t que no hi ha dades importants abans d’executar això.
   ```bash
   sudo rm -rf *
   ```
   
3. **Descarrega el fitxer `.zip`** de la plataforma triada (Nextcloud o ownCloud) al teu sistema.
    ```bash
    wget https://download.nextcloud.com/server/releases/latest.zip
   ```

4. **Descomprimeix l’arxiu directament al directori**:

   - **Heu descarregat l'arxiu a una ruta qualsevol**
   ```bash
   sudo unzip /ruta/al/arxiu.zip
   ```
   > Si l’arxiu crea una carpeta interna (ex: `nextcloud/` o `owncloud/`), assegura’t que el contingut es mogui **al nivell arrel** del virtual host:
   ```bash
   sudo mv nextcloud/* . && sudo rmdir nextcloud
   # o
   sudo mv owncloud/* . && sudo rmdir owncloud
   ```

   - **Podeu fer això directament si ho teniu descomprimit a `Descargas`:**
   ```bash
   cp -R ~/Descargas/nextcloud/. /var/www/domini.local/.
   ```
   Elimineu la carpeta `nextcloud` i l'arxiu `latest.zip`
    ```bash
    sudo rm -rf ~/Descargas/nextcloud && sudo rm -rf ~/Descargas/latest.zip
    ```

   - **Podeu fer això directament si ho teniu descomprimit a `/var/www/domini.local`:**
   ```bash
   cp -R /var/www/domini.local/nextcloud/. /var/www/domini.local/.
   ```
   Elimineu la carpeta `nextcloud` i l'arxiu `latest.zip`
    ```bash
    sudo rm -rf /var/www/domini.local/nextcloud && sudo rm -rf /var/www/domini.local/latest.zip
    ```
6. **Assegura els permisos correctes**:
   ```bash
   sudo chown -R www-data:www-data /var/www/domini.local
   sudo chmod -R 755 /var/www/domini.local
   ```

7. **Accedeix a la interfície web**:
   Obre el navegador i visita:
   ```
   http://domini.local
   ```
   Segueix les instruccions de configuració assistida:
   - Crea un usuari administrador.
   - Configura la base de dades (recomanat: MariaDB/MySQL).
   - Verifica que tots els requisits del sistema es compleixin.

## 2. Recomanacions addicionals

- **Directori de dades**: Durant la instal·lació, es recomana **no emmagatzemar les dades dins del directori web** (ex: `/var/www/domini.local/data`). Millor usa una ruta externa com `/var/ncdata` o `/opt/owncloud-data`.
- **Còpies de seguretat**: Fes *backups* regulars del directori de dades i de la base de dades.
- **Seguretat**: Desactiva l’accés a fitxers sensibles (`.htaccess`, `config.php`) i considera afegir regles de seguretat addicionals a Apache o Nginx.


## Apèndix: Instal·lació de PHP 7.4 a Ubuntu 24.04 (només per a ownCloud)

> **Aquest pas només és necessari si instal·les ownCloud**, ja que moltes versions estables encara no són compatibles amb PHP 8.3 (versió per defecte a Ubuntu 24.04). Nextcloud **no requereix aquest pas**.

### Passos:

1. **Actualitza el sistema**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Instal·la les dependències per afegir repositoris PPA**:
   ```bash
   sudo apt install software-properties-common -y
   ```

3. **Afegeix el repositori de PHP de Ondřej Surý**:
   ```bash
   LC_ALL=C.UTF-8 sudo add-apt-repository ppa:ondrej/php -y
   ```

4. **Actualitza els repositoris**:
   ```bash
   sudo apt update
   ```

5. **Instal·la PHP 7.4 i les extensions requerides**:
   ```bash
   sudo apt install -y php7.4 \
       libapache2-mod-php7.4 \
       php7.4-fpm \
       php7.4-common \
       php7.4-mbstring \
       php7.4-xmlrpc \
       php7.4-soap \
       php7.4-gd \
       php7.4-xml \
       php7.4-intl \
       php7.4-mysql \
       php7.4-cli \
       php7.4-ldap \
       php7.4-zip \
       php7.4-curl
   ```

6. **(Opcional) Selecciona PHP 7.4 com a versió per defecte**:
   ```bash
   sudo update-alternatives --config php
   ```

7. **Activa els mòduls d’Apache necessaris**:
   ```bash
   sudo a2enmod proxy_fcgi setenvif
   sudo a2enconf php7.4-fpm
   ```

8. **Reinicia Apache**:
   ```bash
   sudo systemctl restart apache2
   ```

> **Verificació**: Pots comprovar la versió activa de PHP amb:
> ```bash
> php -v
> ```
