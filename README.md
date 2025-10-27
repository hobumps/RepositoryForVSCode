<?php
session_start();
// DB接続設定
$dsn = 'mysql:dbname=tb270595db;host=localhost';
$user = 'tb-270594';
$password = 'w6fETMAwuw';
$pdo = new PDO($dsn, $user, $password, array(PDO::ATTR_ERRMODE => PDO::ERRMODE_WARNING));
try {
    $pdo = new PDO($dsn, $user, $password, [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    ]);
} catch (PDOException $e) {
    exit('DB接続エラー: ' . $e->getMessage());
}

$errors = [];
$mode   = $_POST['mode'] ?? 'login'; // login or register

// ログイン処理
if ($_SERVER['REQUEST_METHOD'] === 'POST' && $mode === 'login') {
    $username = trim($_POST['username'] ?? '');
    $password = $_POST['password'] ?? '';

    if ($username === '' || $password === '') {
        $errors[] = 'ユーザー名とパスワードを入力してください。';
    } else {
        $stmt = $pdo->prepare('SELECT * FROM users WHERE username = ? LIMIT 1');
        $stmt->execute([$username]);
        $user = $stmt->fetch(PDO::FETCH_ASSOC);

        if ($user && password_verify($password, $user['password'])) {
            $_SESSION['logged_in'] = true;
            $_SESSION['user_id']   = $user['id']; 
            $_SESSION['username']  = $username;
            header('Location: index.php');
            exit;
        } else {
            $errors[] = 'ユーザー名またはパスワードが違います。';
        }
    }
}

    if (!$errors) {
        // 重複チェック
        $stmt = $pdo->prepare('SELECT COUNT(*) FROM users WHERE username = ?');
        $stmt->execute([$username]);
        if ($stmt->fetchColumn() > 0) {
            $errors[] = 'そのユーザー名は既に使われています。';
        } else {
            $hash = password_hash($password, PASSWORD_DEFAULT);
            $stmt = $pdo->prepare('INSERT INTO users (username, password, role) VALUES (?, ?, ?)');
            $stmt->execute([$username, $hash, $role]);
            $_SESSION['logged_in'] = true;
            $_SESSION['user_id']   = $user_id;
            $_SESSION['username']  = $username;
            header('Location: login.php');
            exit;
        }
    }
}

function h($s) { return htmlspecialchars($s, ENT_QUOTES, 'UTF-8'); }
?>
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>タイトル</title>
  <style>
    body { font-family: sans-serif; padding:20px; }
    .tab-btn { margin-right:10px; }
    form { margin-top:20px; }
    .error { color:red; }
    #admin-question { display:none; margin-top:10px; }
  </style>
</head>
<body>
  <h1 id="form-title">ログイン</h1>

  <button class="tab-btn" onclick="switchTab('login')">ログイン</button>
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
    <input type="hidden" id="mode" name="mode" value="<?php echo h($mode); ?>">
    <div>
      <label for="username">ユーザー名:</label>
      <input id="username" name="username" type="text" required>
    </div>
    <div>
      <label for="password">パスワード:</label>
      <input id="password" name="password" type="password" required>
    </div>
    <div>
      <button id="submit-btn" type="submit">ログイン</button>
    </div>
  </form>
</body>
</html>

