<?php
session_start();

// --- DB接続設定 ---
$dsn = 'mysql:host=localhost;dbname=tb270594db;charset=utf8mb4'; // ←あなたが作成したDB名
$user = 'tb-270594';
$password = 'w6fETMAwuw';

try {
    $pdo = new PDO($dsn, $user, $password, [
        PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES   => false,
    ]);
} catch (PDOException $e) {
    exit('DB接続エラー: ' . $e->getMessage());
}

function h($s){ return htmlspecialchars($s, ENT_QUOTES, 'UTF-8'); }

$errors = [];

// --- ログイン処理 ---
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = trim($_POST['username'] ?? '');
    $pw_plain = $_POST['password'] ?? '';

    if ($username === '' || $pw_plain === '') {
        $errors[] = 'ユーザー名とパスワードを入力してください。';
    } else {
        $stmt = $pdo->prepare('SELECT id, username, password_hash FROM users WHERE username = ? LIMIT 1');
        $stmt->execute([$username]);
        $u = $stmt->fetch();

        if ($u && password_verify($pw_plain, $u['password_hash'])) {
            $_SESSION['logged_in'] = true;
            $_SESSION['user_id']   = $u['id'];
            $_SESSION['username']  = $u['username'];
            header('Location: index.php'); // ← ログイン後のページ
            exit;
        } else {
            $errors[] = 'ユーザー名またはパスワードが違います。';
        }
    }
}
?>
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>ログイン</title>
  <style>
    body { font-family: sans-serif; padding: 30px; }
    form { max-width: 350px; margin-top: 20px; }
    label { display: block; margin-top: 10px; }
    input[type="text"], input[type="password"] {
      width: 100%; padding: 8px; box-sizing: border-box;
    }
    button { margin-top: 15px; padding: 8px 12px; }
    .error { color: #b00020; margin-top: 10px; }
  </style>
</head>
<body>
  <h1>ログイン</h1>

  <?php if ($errors): ?>
    <div class="error">
      <ul>
        <?php foreach ($errors as $e): ?>
          <li><?php echo h($e); ?></li>
        <?php endforeach; ?>
      </ul>
    </div>
  <?php endif; ?>

  <form method="post" action="">
    <label for="username">ユーザー名</label>
    <input id="username" name="username" type="text" required>

    <label for="password">パスワード</label>
    <input id="password" name="password" type="password" required>

    <button type="submit">ログイン</button>
  </form>
</body>
</html>
