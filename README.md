<?php
// DB接続
$dsn = 'mysql:dbname=tb270594db;host=localhost';
$user = 'tb-270594';
$password = 'w6fETMAwuw';
$pdo = new PDO($dsn, $user, $password, array(PDO::ATTR_ERRMODE => PDO::ERRMODE_WARNING));

// 仮のログインユーザーID（本来はセッションで管理）
$my_id = 1;
$partner_id = 2;

// メッセージ送信処理
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $content = $_POST['message'];
    $stmt = $pdo->prepare("INSERT INTO messages (sender_id, receiver_id, content) VALUES (?, ?, ?)");
    $stmt->execute([$my_id, $partner_id, $content]);
    exit;
}

// メッセージ取得処理（AJAX）
if (isset($_GET['load'])) {
    $stmt = $pdo->prepare("SELECT * FROM messages WHERE (sender_id=? AND receiver_id=?) OR (sender_id=? AND receiver_id=?) ORDER BY sent_at ASC");
    $stmt->execute([$my_id, $partner_id, $partner_id, $my_id]);
    while ($row = $stmt->fetch()) {
        $class = $row['sender_id'] == $my_id ? 'me' : 'other';
        echo "<div class='message $class'>{$row['content']}</div>";
    }
    exit;
}
?>

<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>LINE風チャット</title>
<style>
body { font-family: sans-serif; background: #e5ddd5; padding: 20px; }
#chatBox { height: 400px; overflow-y: scroll; background: white; padding: 10px; border-radius: 10px; margin-bottom: 10px; }
.message { padding: 10px; margin: 5px; border-radius: 10px; max-width: 60%; }
.me { background-color: #dcf8c6; text-align: right; margin-left: auto; }
.other { background-color: #fff; text-align: left; margin-right: auto; }
form { display: flex; }
input[type="text"] { flex: 1; padding: 10px; border-radius: 5px; border: 1px solid #ccc; }
button { padding: 10px; background-color: #00c300; color: white; border: none; border-radius: 5px; }
</style>
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
</head>
<body>
<h2>💬 LINE風チャット</h2>
<div id="chatBox"></div>

<form id="chatForm">
  <input type="text" id="messageInput" name="message" placeholder="メッセージを入力">
  <button type="submit">送信</button>
</form>

<script>
function loadMessages() {
  $.get('?load=1', function(data) {
    $('#chatBox').html(data);
    $('#chatBox').scrollTop($('#chatBox')[0].scrollHeight);
  });
}

$('#chatForm').submit(function(e) {
  e.preventDefault();
  $.post('', { message: $('#messageInput').val() }, function() {
    $('#messageInput').val('');
    loadMessages();
  });
});

setInterval(loadMessages, 3000);
loadMessages();
</script>
</body>
</html>
