# Wiki d'un Repositori PHP: Prevenció d'Injecció de Codi

## Descripció del Projecte

Aquest projecte PHP demostra les millors pràctiques per prevenir atacs d'injecció de codi, incloent SQL injection, XSS (Cross-Site Scripting), i altres vulnerabilitats similars.

## Tècniques de Prevenció

### 1. Prevenció d'SQL Injection

```php
// CONNEXIÓ SEGURA AMB PDO
$db = new PDO('mysql:host=localhost;dbname=exemple;charset=utf8', 'usuari', 'contrasenya');
$db->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
$db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

// CONSULTA PREPARADA
$stmt = $db->prepare("SELECT * FROM usuaris WHERE email = :email");
$stmt->execute(['email' => $email]);
$usuari = $stmt->fetch();
```

Alternativa amb MySQLi:

```php
$stmt = $mysqli->prepare("SELECT * FROM usuaris WHERE email = ?");
$stmt->bind_param("s", $email);
$stmt->execute();
```

### 2. Prevenció de XSS (Cross-Site Scripting)

```php
// ESCOMBRA ELS VALORS ABANS DE MOSTRAR-LOS
echo htmlspecialchars($input_usuari, ENT_QUOTES, 'UTF-8');

// PER URLS
echo urlencode($input_usuari);
```

### 3. Validació d'Entrada

```php
// VALIDACIÓ D'EMAIL
if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
    throw new Exception("Format d'email invàlid");
}

// VALIDACIÓ DE NOMBRES ENTERS
$id = filter_var($_GET['id'], FILTER_VALIDATE_INT);
if ($id === false) {
    throw new Exception("ID ha de ser un nombre enter");
}
```

## Exemple Complet

```php
<?php
// config.php - Configuració de la base de dades
define('DB_HOST', 'localhost');
define('DB_NAME', 'aplicacio_segura');
define('DB_USER', 'usuari_segur');
define('DB_PASS', 'contrasenya_complexa');

// database.php - Classe per gestionar la base de dades
class Database {
    protected $connection;

    public function __construct() {
        $dsn = 'mysql:host='.DB_HOST.';dbname='.DB_NAME.';charset=utf8';
        $options = [
            PDO::ATTR_EMULATE_PREPARES => false,
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
        ];

        try {
            $this->connection = new PDO($dsn, DB_USER, DB_PASS, $options);
        } catch (PDOException $e) {
            die("Error de connexió: " . $e->getMessage());
        }
    }

    public function getUsuariSegur($id) {
        $stmt = $this->connection->prepare("SELECT id, nom, email FROM usuaris WHERE id = ?");
        $stmt->execute([$id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }
}

// user.php - Classe per gestionar usuaris
class User {
    public static function mostrarPerfil($id) {
        $db = new Database();
        $usuari = $db->getUsuariSegur($id);

        if ($usuari) {
            // Escapar totes les sortides
            $nom = htmlspecialchars($usuari['nom'], ENT_QUOTES, 'UTF-8');
            $email = htmlspecialchars($usuari['email'], ENT_QUOTES, 'UTF-8');

            echo "<h1>Perfil de {$nom}</h1>";
            echo "<p>Email: {$email}</p>";
        } else {
            echo "<p>Usuari no trobat</p>";
        }
    }
}

// Ús segur
$id_usuari = filter_input(INPUT_GET, 'id', FILTER_VALIDATE_INT);
if ($id_usuari) {
    User::mostrarPerfil($id_usuari);
} else {
    echo "ID d'usuari invàlid";
}
?>
```

## Recursos Addicionals

- [OWASP SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [PHP Data Filtering](https://www.php.net/manual/en/book.filter.php)

Aquest wiki proporciona una base sòlida per desenvolupar aplicacions PHP segures contra injecció de codi.
