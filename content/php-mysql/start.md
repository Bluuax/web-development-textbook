---
title: Start mit PHP und MySQL
order: 10
---
MySQL ist die relationale Datenbank, die bei gemietetem Webspace am öftesten angeboten wird. Hier wird nicht die Funktionsweise einer relationalen Datenbank erklärt (siehe Lehrveranstaltung im 1.Semester), sondern nur die Besonderheiten von MySQL und die für Web-Applikationen wichtigen Aspekte.

MySQL Installation, Wiederholung, Dokumentation
------------------------------------------------
MySQL ist im Paket XAMPP enthalten. (Man könnte die Datenbank auch separat von mysql.com herunterladen und installieren.) In der Standardinstallation ist ein Administrator-Account „root“ ohne Passwort vorhanden. Die MySQL-Shell wird wie folgt gestartet:

<shell caption="auf der Kommandozeile die MySQL-Shell starten">
    > mysql –u username –p datenbankname
    Welcome to the MySQL monitor. Commands end with ; or \g.

    Your MySQL connection id is 2 to server version: 5.0.27-community-nt


    Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

    mysql>
</shell>

Auf der Kommandozeile kann man auch ganze Dateien mit SQL-Befehlen auf einen Schlag einspielen: 

`mysql –u username –p datenbankname < portfolio.sandbox.sql`

Show Tables zeigt alle Tabellen in der aktuellen Datenbank:

<sql caption="Tabellen in der Datenbank anzeigen">
mysql> show tables;

+------------------------------------------+
| Tables_in_portfolio_sandbox              |
+------------------------------------------+
| agegroups                                |
| agegroups_studycourses_departments_users |
| assets                                   |
| awardyears                               |
| departments                              |
| projects                                 |
| projects_roles_users                     |
| projects_tags                            |
| roles                                    |
| studycourses                             |
| tags                                     |
| urls                                     |
| users                                    |
+------------------------------------------+
13 rows in set (0.01 sec)
</sql>

Describe zeigt den Aufbau einer bestimmten Tabelle:
    
<sql caption="Tabelle beschreiben mit describe">
mysql> describe users;
+-----------------+--------------+------+-----+---------+----------------+
| Field           | Type         | Null | Key | Default | Extra          |
+-----------------+--------------+------+-----+---------+----------------+
| id              | int(11)      | NO   | PRI | NULL    | auto_increment |
| firstname       | varchar(255) | YES  |     | NULL    |                |
| surname         | varchar(255) | YES  |     | NULL    |                |
| email           | varchar(255) | YES  | UNI | NULL    |                |
| isfemale        | tinyint(1)   | YES  |     | NULL    |                |
| profile_visible | tinyint(1)   | YES  |     | NULL    |                |
| created_at      | datetime     | YES  |     | NULL    |                |
| updated_at      | datetime     | YES  |     | NULL    |                |
| avatar          | varchar(255) | YES  |     | NULL    |                |
| fullname        | varchar(255) | YES  | MUL | NULL    |                |
+-----------------+--------------+------+-----+---------+----------------+
10 rows in set (0.00 sec)
</sql>

Select und Join funktionieren wie erwartet:

<sql caption="SQL-Befehle absetzten in der MySQL-Shell">
mysql> select id,firstname from users limit 0,5;
+----+-----------+
| id | firstname |
+----+-----------+
|  1 | Brigitte  |
|  2 | Lea       |
|  3 | Stefan    |
|  4 | Karin     |
|  5 | Katharina |
+----+-----------+
5 rows in set (0.00 sec)

mysql> select projects.id,projects.title,urls.url from projects left join urls on projects.id=urls.project_id where projects.id=101;
+-----+---------------------+---------------------------------------------------------+
| id  | title               | url                                                     |
+-----+---------------------+---------------------------------------------------------+
| 101 | Every step you take | http://www.everystepyoutake.org/                        |
| 101 | Every step you take | https://mediacube.at/wiki/index.php/Every_step_you_take |
+-----+---------------------+---------------------------------------------------------+
2 rows in set (0.01 sec)
</sql>

Die Details zu SQL in MySQL (Abweichungen vom SQL Standard, Erweiterungen) kann man der [&rarr;Dokumentation](http://dev.mysql.com/) entnehmen.

![Abbildung 142: Dokumentation von MySQL auf http://dev.mysql.com/](/images/mysql-doku.png)

Ein häufig verwendetes Tool ist der phpMyAdmin, der ein Interface am Web zur Verfügung stellt. 

![Abbildung 143: phpMyAdmin](/images/mysql-phpmyadmin.png)

Über phpMyAdmin kann man viele SQL-Abfragen durch Point+Klick formulieren. Das lehrreiche daran: phpMyAdmin zeigt immer das SQL-Statement mit an — auf diese Art kann man einfach SQL lernen. Besonders praktisch ist das beim Anlegen und Verändern von Tabellen. 

![Abbildung 143: Tabelle anlegen mit phpMyAdmin - Name der Tabelle](/images/phpmyadmin-create-table.png)


MySQL von PHP aus
------------------
Um von PHP auf die Datenbank zuzugreifen gibt es verschiedene Schnittstellen. Hier werden die „PHP Database Objects“ (PDO) vorgestellt.

Verbindungsaufbau

So funktioniert der Verbindung-Aufbau (und -Abbau) zur mysql-Datenbank:

<php caption="Verbindungsaufbau zur Datenbank">
    <?
     include "config.php";
     $dbh = new PDO($DSN, $DB_USER, $DB_PASS);
     $dbh->exec('SET CHARACTER SET utf8');
    ?> 
</php>
    

Das „Database-Handle“ `$dbh` wird nun im Weiteren für Abfragen verwendet. 

### Datenbank-Zugangsdaten und .gitignore

Achtung: zum Erzeugen des Database Handles brauchen wir noch eine zweite Datei mit den eigentlichen Zugangsdaten. Diese Datei heisst im Beispiel config.php heissen:

<php caption="Datei config.php">
    <?php
    $DB_NAME = "portfolio_sandbox"; 
    $DB_USER = "mmtuser"; 
    $DB_PASS = "geheim!";
    $DSN     = "mysql:dbname=$DB_NAME;host=localhost";
    ?>
</php>

Warum zwei Dateien?  Weil dieses zweite Datei niemals, niemals, niemals in git commited werden darf!  Um das zu verhindern, wird die Datei in .gitignore eingetragen. Was das bewirkt zeigt der Vorher / Nachher-Vergleich am besten:

<shell>
    D:\Webprojekte\wp2&gt;git status

    # On branch master
    # Your branch is ahead of 'origin/master' by 2 commits.
    #
    # Untracked files:
    #   (use "git add <file>..." to include in what will be committed)
    #
    #       config.php
    nothing added to commit but untracked files present (use "git add" to track)
</shell>

noch erkennt git die datei config.php als neue, interessante Datei. Nun tragen wir den Dateinamen config.php in die Datei .gitignore im Haupt-Ordner der Working Copy ein:

<shell>
    D:\Webprojekte\wp2&gt;cat .gitignore
    *.bak
    config.php
</shell>

Damit ist git angewiesen, alle Dateien mit der Endung .bak und alle Dateien mit dem Namen config.php (egal in welchem Ordner) zu ignorieren. 

<shell>
    D:\Webprojekte\wp2>git status

    # On branch master
    # Your branch is ahead of 'origin/master' by 2 commits.
    #
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #       modified:   .gitignore
    #
    no changes added to commit (use "git add" and/or "git commit -a")
</shell>

Wie man sieht zeigt git status nun die Datei config.php nicht mehr an. Dafür hat git bemerkt dass die Datei .gitignore geändert wurde. Die sollte man ganz normal commiten.

### Select

Eine Abfrage aus der Datenbank liefert normalerweise eine ganze Tabelle von Daten (mehrere Datensätze). Man braucht also eine Schleife um alle Datensätze abzuarbeiten. Innerhalb der Schleife erhält man den einzelnen Datensatz als Array. 

<php caption="Daten mit SELECT lesen">
    $query =$dbh->query(
         "SELECT * FROM users WHERE profile_visible ORDER BY RAND() LIMIT 1,10"
    );
    $personen = $query->fetchAll(PDO::FETCH_OBJ);
    foreach($personen as $person) {
          echo "$person->firstname $person->email</br>\n";
    }
</php>

Die SQL-Anfrage wird hier als String an die query-methode des Datenbankhandler übergeben.  Der Rückgabewert von query ist ein Query-Handle (ähnlich dem Datenbank-Handle).  Zu diesem Zeitpunkt wurden noch keine Daten von der Datenbank zu PHP übertragen. Das passiert erst in der nächsten Zeile mit der Methode fetchAll. Der Rückgabewert von fetchAll ist in diesem Fall ein Array mit Objekten. Dieses Array wird anschließend mit einer foreach-Schleife abgearbeitet.

### Effizient Arbeiten mit der Datenbank

Ein ganz wichtiges Grundprinzip beim Programmieren mit Datenbanken: Das Filtern und Berechnen der Daten möglichst in der Datenbank erledigen und möglichst wenige Daten zu PHP übermitteln. Folgender Ansatz wäre also ganz schlecht, besonders wenn viele Daten in der Datenbank sind:

<php caption="Ineffizientes Lesen aus der Datenbank">
    // FALSCH !!!
    $query =$dbh->query("SELECT * FROM users");
    $personen = $query->fetchAll(PDO::FETCH_OBJ);
    foreach($personen as $person ) {
        if($person->profile_visible) {
          echo "$person->firstname $person->email</br>\n";
        }
    }
    // FALSCH !!!
</php>

Richtig wäre, den Filter bereits im SELECT einzubauen:

<php caption="Effizientes Lesen aus der Datenbank">
    $query =$dbh->query("SELECT * FROM users WHERE profile_visible");
    $personen = $query->fetchAll(PDO::FETCH_OBJ);
    foreach($personen as $person) {
          echo "$person->firstname $person->email</br>\n";
    }
</php>

Zu diesem Prinzip gehört auch die konsequente Verwendung der richtigen Datentypen in der Datenbank:  zum Beipiel zum Speichern eines Datums DATE oder TIMESTAMP verwenden. Das ermöglicht das Sortieren nach Datum und  Berechnungen wie „falls datum älter als 300 Tage ist“

<sql caption="select mit verwendung von Datums-Angaben">
    select title,publicationdate from projects
    where datediff( curdate( ) , publicationdate ) > 300; 
</sql>

Zeigt Titel und Publikations-Datum aller Werke die vor mehr als 300 Tagen publiziert wurden.

